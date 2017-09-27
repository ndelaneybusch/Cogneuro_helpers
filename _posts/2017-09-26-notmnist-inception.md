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
  
Like many simple visual classification tasks, this is a good opportunity to break out our handy [inception block (see here for my implementation)](/Cogneuro_helpers/deep/Inception/). We'd could likely get some pretty great performance if we trained a sequence of a inception blocks, but to start with I wanted to see what I could accomplish keeping the training time to about a half-hour on my GTX 1070, so we'll start with one.  
  
To reduce the width of the problem, I started with a [convolution block](/Cogneuro_helpers/deep/Inception/) over the 28x28x1 input directly into a MaxPooling with a 2x2 stride, resulting in a 14x14 width. Two more convolution blocks and another MaxPooling reduced it to a 7x7 width. Then, things were passed through the inception block. My intuition was that a lot of the relevant low-level features were pretty simple and relatively large, but that the primary challenge of the classification task would be to combine the higher-order features in sophisticated ways. So moved things towards depth very quickly, despite the relatively modest initial size (many image classification problems take pictures with 3-4 orders of magnitudes more pixels, and multiple color channels).  
  
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

Training was batched to 512 samples at a time, againt mostly to help reduce overfitting. A single pass through the data was taking about ~25 seconds, so I justed scaled it up to 60 runs to get a training time under a half-hour. Loss was still dropping on the training data (which isn't surprising given that it had a million and a half parameters), but gains were pretty modest by that point:  

[![tensorboard_notmnist](/Cogneuro_helpers/img/tensorboard_notmnist.png)](/Cogneuro_helpers/img/tensorboard_notmnist.png)  
  
The fitting process looked nice and stable, which isn't too surprising given the batch size and quantity of available training data for this particular (relatively simple) model.  

So how'd we do? For reference, an optimized multinomial logistic regression gets us to about ~90% accuracy.  

I ran my preferred classification report, and here's the output:

First is the confusion matrix:
<pre>
[[972   2   0  12   0   5   0   2   2   5]
 [  0 977   5   8   3   0   4   0   1   2]
 [  1   1 975   2   8   0  11   0   1   1]
 [  0   3   1 985   1   1   3   0   1   5]
 [  0   0   9   1 975   7   1   0   4   3]
 [  0   1   1   3   7 977   1   2   5   3]
 [  1   3   1   4   4   1 983   0   0   3]
 [  4   3   1   2   2   1   1 970   9   7]
 [  2   6   2   6   3   2   3   3 960  13]
 [  1   3   0   3   2   0   0   0  14 977]]
 </pre>
 
 Then the precision, recall, and f1 values:
 <pre>
              precision    recall  f1-score   support

          0       0.99      0.97      0.98      1000
          1       0.98      0.98      0.98      1000
          2       0.98      0.97      0.98      1000
          3       0.96      0.98      0.97      1000
          4       0.97      0.97      0.97      1000
          5       0.98      0.98      0.98      1000
          6       0.98      0.98      0.98      1000
          7       0.99      0.97      0.98      1000
          8       0.96      0.96      0.96      1000
          9       0.96      0.98      0.97      1000

avg / total       0.98      0.98      0.98     10000
</pre>

Then the summary metrics, sensitivity, and specificity (note: recall and sensitivity are the same thing).
<pre>
accuracy: 0.9751
cohen's kappa: 0.972333333333
sensitivity: [ 0.972  0.977  0.975  0.985  0.975  0.977  0.983  0.97   0.96   0.977]
specificity: [ 0.999   0.9976  0.9978  0.9954  0.9967  0.9981  0.9973  0.9992  0.9959
  0.9953]
</pre>

This single inception-block model brings us all the way down to ~2.5% misclassification. It looks like the primary problem is confusing "i" and "j" with one another (see bottom-right of confusion matrix), and some conservative labeling with "a" and "h" cases (sensitivity is lower than the others, but specificity is higher). The former would be pretty easy to address by building a very small secondary model specifically to address cases where this one thinks that both "i" and "j" are reasonably probable (to focus training demands on the discriminating features of the case), the latter might be addressable by either weighting "a" and "h" cases a bit and/or synthesizing new training cases with more variable features (to try to encourage more robust generalization for those letters).  

In all, it's a pretty good performance for some educated guesses, half an hour of training, and only 1.5 million parameters. 
