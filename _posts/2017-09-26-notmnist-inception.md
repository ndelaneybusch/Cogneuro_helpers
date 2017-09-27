---
layout: post
title: notMNIST Inception
subtitle: Classifying letters with an inception model
---

The [notMNIST](https://yaroslavvb.blogspot.co.uk/2011/09/notmnist-dataset.html) data set is great one for playing around with relatively simple neural network models. It's a collection of letters (a through j) from a huge array of different fonts, all formatted to the same 28x28 picture size. From our perspective, this is a 10-label classification problem with hundreds of thousands of samples to train from and some really funky visual features to try to extract and generalize across:
  
[![source_notmnist](/Cogneuro_helpers/img/source_notmnist.png)](/Cogneuro_helpers/img/source_notmnist.png)  
<sub><a href='https://yaroslavvb.blogspot.co.uk/2011/09/notmnist-dataset.html'>source</a>  
  
It's a wonderfully dirty data set with a healthy dose of mislabeled and/or bizarre trials.  
  
[![sample_notmnist](/Cogneuro_helpers/img/sample_notmnist.png)](/Cogneuro_helpers/img/sample_notmnist.png)  
  
This random sample of 100 letters appears to include an alien, a pear, a jalepeno, a star, and a strip of film?  

Fortunately, the test set is clean, so we'll just need to build a model that is robust to these weird training samples.  
  
Like many simple visual classification tasks, this is a good opportunity to break out our handy [inception block (see here for my implementation)](/Cogneuro_helpers/deep/Inception/). We'd could likely get some pretty great performance if we trained a sequence of a inception blocks, but to start with I wanted to see what I could accomplish keeping the training time to about ~20 minutes on my GTX 1070, so we'll start with one.  
  
To reduce the width of the problem, I started with a [convolution block](/Cogneuro_helpers/deep/Inception/) over the 28x28x1 input directly into a MaxPooling with a 2x2 stride, resulting in a 14x14 width. Two more convolution blocks and another MaxPooling reduced it to a 7x7 width. Then, things were passed through the inception block. My intuition was that a lot of the relevant low-level features were pretty simple, but that the primary challenge of the classification task would be to combine the higher-order features in sophisticated ways. So moved things towards depth very quickly, despite the relatively modest initial size (many image classification problems take pictures with 3-4 orders of magnitudes more pictures and multiple color channels).  
  
Following the inception block, I flattened the deep output to a single dimension and fed it into a modest 64-neuron fully connected layer, which then fed into a 10-neuron softmax classifier (which returns class probabilities).  

That brings us to this:  
{% highlight python %}
shape=(28,28,1)
main_input = Input(shape, dtype='float32', name='main_input')
main_model = conv_block(main_input, 32, 3, 3, strides=(1,1))
main_model = MaxPooling2D((3,3), strides=(2,2), padding='same')(main_model)
main_model = conv_block(main_model, 32, 3, 3)
main_model = conv_block(main_model, 64, 3, 3)
main_model = MaxPooling2D((3,3), strides=(2,2), padding='same')(main_model)
main_model = inceptionv2_block(main_model, 64, 96)
main_model = Flatten()(main_model)
main_model = Dense(64, activation='relu')(main_model)
main_output = Dense(10, activation='softmax', name='main_output')(main_model)
model = Model(inputs=main_input, outputs=main_output)
{% endhighlight %}

Considering the bizarre training cases that we want to avoid being overly influenced by, we're definitely going to want to some extra insurance against overfitting. Every convolution layer is implemented with my convolution block, which already has l2 regularization over the weights. But this situation demands a few dropout layers to help generalization. Dropout layers take some random proportion of the activation (here, 20%) that would be passed to the next layer during each training run, and sets it to zero. This ensures that no neuron can depend on any particular preceding weights, which decreases overfitting to specific trials (i.e. sets of neurons that "remember" specific training samples) and increases redundancy (i.e. the most "important" features in each level tend to have information reflected in many neurons).  

So we finish things up by adding two dropout layers, compiling the model with a loss function and a training algorithm, slap on some tensorboard callbacks so we can follow along during training, and hit "go". The full model is as follows (assuming the conv_block and inception block are already defined, as described [here](/Cogneuro_helpers/deep/Inception/)):

{% highlight python %}
from keras.models import Sequential
from time import time
from keras.layers import Dense, Dropout, Activation, Merge, Conv2D, MaxPooling2D, Flatten, BatchNormalization, Input, Concatenate
from keras.regularizers import l2, l1
from keras.models import Model
from keras.callbacks import TensorBoard

shape=(28,28,1)
main_input = Input(shape, dtype='float32', name='main_input')
main_model = conv_block(main_input, 32, 3, 3, strides=(1,1))
main_model = MaxPooling2D((3,3), strides=(2,2), padding='same')(main_model)
main_model = conv_block(main_model, 32, 3, 3)
main_model = Dropout(0.2)(main_model)
main_model = conv_block(main_model, 64, 3, 3)
main_model = MaxPooling2D((3,3), strides=(2,2), padding='same')(main_model)
main_model = inceptionv2_block(main_model, 64, 96)
main_model = Flatten()(main_model)
main_model = Dropout(0.2)(main_model)
main_model = Dense(64, activation='relu')(main_model)
main_output = Dense(10, activation='softmax', name='main_output')(main_model)
model = Model(inputs=main_input, outputs=main_output)
model.compile(optimizer='rmsprop', loss='categorical_crossentropy')
tensorboard = TensorBoard(log_dir="logs/{}".format(time()))
model.fit(train_dataset, train_labels, batch_size = 512, epochs = 60, verbose = 1, callbacks=[tensorboard])
{% endhighlight %}

