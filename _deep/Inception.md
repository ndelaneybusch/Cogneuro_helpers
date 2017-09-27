---
title: Convolution shorthands and inception blocks
---
# Description
The convolution code block provides a shorthand for adding "convolution -> batchnormalize -> ReLU" layer sequences to a Keras model. The inception block provides a shorthand for adding [inception blocks](https://arxiv.org/pdf/1409.4842.pdf) to Keras models. These are coded for channel-end input formats, but would be pretty easy to modify for channel-first formats.  
  
The inception block implements convolution blocks over different scales (1x1, 3x3, 5x5, and pooled), and then concatenates them depthwise. They are a simple and computationally cheap but highly effective way of improving image classification.  
  
[![inceptionblock_diagram](/Cogneuro_helpers/img/inceptionblock_diagram.png)](/Cogneuro_helpers/img/inceptionblock_diagram.png)
<sub><a href='https://medium.com/initialized-capital/we-need-to-go-deeper-a-practical-guide-to-tensorflow-and-inception-50e66281804f'>diagram source</a>

Each convolution in the inception is implemented with the convolution -> batchnormalize -> ReLU sequence (see end of page for the expanded tensorboard model showing all the layers).

# Dependencies
{% highlight python %}
from keras.models import Model
from keras.regularizers import l2
from keras.layers import Conv2D, BatchNormalization, Activation, MaxPooling2D, Input, Concatenate
{% endhighlight %}

# Functions
## Convolution block:
{% highlight python %}
def conv_block(x, nb_filter, num_row, num_col, strides=(1,1), padding='same', bias=False):
    x = Conv2D(nb_filter, kernel_size=(num_row, num_col), strides=strides, padding=padding, kernel_regularizer=l2(0.00004), use_bias=bias)(x)
    x = BatchNormalization(axis=-1)(x)
    x = Activation('relu')(x)
    return x
{% endhighlight %}

The convolution block is required for the inception blocks:

## Inception block:
{% highlight python %}
def inception_block(x, filter_1by1, filter_primary):
    convmin_br = conv_block(x, filter_1by1, 1, 1)

    conv33_br = conv_block(x, filter_1by1, 1, 1)
    conv33_br = conv_block(conv33_br, filter_primary, 3, 3)

    conv55_br = conv_block(x, filter_1by1, 1, 1)
    conv55_br = conv_block(conv55_br, filter_primary, 5, 5)

    directpool_br = MaxPooling2D((3,3), strides=(1,1), padding='same')(x)
    directpool_br = conv_block(directpool_br, filter_primary, 1, 1)

    x = Concatenate()([convmin_br, conv33_br, conv55_br, directpool_br])
    return x
{% endhighlight %}

## Modern Inception block:
From [Rethinking the Inception Architecture for Computer Vision](https://arxiv.org/pdf/1512.00567.pdf)
{% highlight python %}
def inceptionv2_block(x, filter_1by1, filter_primary):
    convmin_br = conv_block(x, filter_1by1, 1, 1)

    conv33_br = conv_block(x, filter_1by1, 1, 1)
    conv33_br = conv_block(conv33_br, filter_primary, 3, 3)

    conv33stack_br = conv_block(x, filter_1by1, 1, 1)
    conv33stack_br = conv_block(conv33stack_br, filter_primary, 3, 3)
    conv33stack_br = conv_block(conv33stack_br, filter_primary, 3, 3)

    directpool_br = MaxPooling2D((3,3), strides=(1,1), padding='same')(x)
    directpool_br = conv_block(directpool_br, filter_primary, 1, 1)

    x = Concatenate()([convmin_br, conv33_br, conv33stack_br, directpool_br])
    return x
{% endhighlight %}

# Function Call
## Convolution block
Put the previous model step (or Input) as the first variable. The next three describe the depth of the convolution filter and the size of the convolution kernal. Options for stride, padding, and bias.
{% highlight python %}
x = conv_block(x, 32, 3, 3, strides=(2,2))
{% endhighlight %}
The 2D convolution is followed by a batch normalization (to center and scale the weights and improve model training) and the application of a rectified linear unit activation (which supports nonlinearities while maintaining extremely nice derivatives for the training).

## Inception blocks:
The convolutions in an inception block are all pre-specified, so just specify the previous layer and the depths (the "1by1" filter sets the depths of all of the initial 1x1 convolution layers, while the "primary" filter sets the target depths that will be concatenated).
{% highlight python %}
x = inception_block(x, 64, 96)
x = inceptionv2_block(x, 64, 96)
{% endhighlight %}

Here is a tensorboard schematic of the full modern inception block, in this case following a max pooling layer (often a good idea):
[![inceptionblock_tensorboard](/Cogneuro_helpers/img/inceptionblock_tensorboard.png)](/Cogneuro_helpers/img/inceptionblock_tensorboard.png)
