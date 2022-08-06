---
title: General Architecture of Convolutional Neural Network
date: 2021-01-31
tags: AI, machine learning, computer vision
index_img: /images/thumbnail/cnn.jpg
---
### Computer vision problems 
To solve the binary classification and multiple-classes recognition problems on image, we can apply the general neural network, where the network takes image pixel values as the input data, uses multiple hidden layers to map the data into probability for the output layer, and update the parameters in each hidden layer during backward propagation. However, if we strech out the scope of the problems, for example, object detection in autonomous driving, where the input image needs to be high resolution, and multiple classes of objects need to be detected and recognized in real-time. In this case, the traditional architecture of neural network may have poor performances on both training and testing phases. 

Imagine the network uses high resolution images as the data inputs, and the size of a high resolution image is $1920 \times 1080$, then the total number of pixels for a single training example will be $1920 \times 1080 \times 3 = 6,220,800$, and the dimension of the parameters in each hidden layer will be $[m, 6,220,800]$ where $m$ is the number of training examples. That is a lot of parameters to train! Therefore, in order to efficiently train a neural network to solve computer vision problems such as face recognition and style transfer, a different architecture of neural network is required. 

### Convolutional neural network
##### Intuition 
As a class of neural network, same as the general neural network, a convolution network(will be stated as "convnet" in below) also has activation layers, forward and backward propagations. However, instead of linearizing the input data with weights and bias, and passing to the activation function of each hidden layer, the activation layers of a convnet uses `filter`, which is also known as `feature detector` to extract the features out of the input image, and project onto the output values. The early layers are usually responsible of detecting some very essential, basic features of the image, such as vertical lines and curved boundary. As going deeper in the network, the activation layers start detecting more complex features, for example, different parts of human faces or the object in foreground. The following image shows the result of using a model called `deconvnet` to visualize the feature activities in intermediate layers of the convnet, and it is not hard to see how the complexity of the extracted features vary across different layers.
![Site Image](/images/cnn/visualize.png)

##### Edge detection 
The most fundamental implementation for the convnet layers is to perform `edge detection` on the input image. Edge detection is a process of extracting a certain features out of image, such as stright lines or round-shaped objects. The kind of features to be extracts depends on how the feature detector is defined. 

Feature detector is a $n \times n$ matrix, where each cell has a parameter used to compute with the pixel value of the input image. The following image shows an example of how to perform edge detection on a $5 \times 5$ input image using a $3 \times 3$ feature detector.
![Site Image](/images/cnn/edge-detection.png)
The number values of the $5 \times 5$ blocks on the left are the pixel of the input image, while the subscript number of the yellow blocks represent the parameters of the feature detector. The computation for edge detection is fairly simple. To get the output values on the right, what we are going to do is take the filter, multiply each filter parameter with the pixel value in the corresponding position, and finally sum them up to a single value. After the first computation, shift the filter on the input image vertically or horizontally by one unit, then repeat the same computation, so on and so forth. Therefore, we get a $4 \times 4$ output image for the activation layer, and these output values represent the extracted features of the input image. Furthermore, to simplify the description for the computation process of edge detection, we can denote the convolve operation as asterisk mark $*$. The formulized description of edge detection will be $[input\ image] * [filter] = [output\ image]$

Note that the input image has RGB color channels, thus the filter will be applied to all the channels. Instead, the filter will also have three layers corresponding to the three color channels of the input. To convolve over the volume, think as both input image and filter as cubes, the output image will simply be sum of the convolution value of all three channels. 

Notice that after the convolutional step, the dimension of the output image will shrink. Therefore usually a `padding` will be added onto the input beforehand extracting the feature in order to keep the output the same size as the input. A padding is an extra border of the image. Normally the value of the padding cells are all $0$ so adding this padding will have no effect when covolving the input image with the filter. Another concept involved in the convolutional step is `stride` which indicates the steps to shift the filter over the input image. For example, a stride of two means the filter will shift with two units instead of one while convolving the image. Overall, the dimension of the output image will be calculated as $\lfloor{\frac{n + 2p - f}{s} + 1}\rfloor \times \lfloor{\frac{n + 2p - f}{s} + 1}\rfloor$ , where $n$ is the original size of input image, $p$ is the padding size, $f$ is the size of the filter, and $s$ is the stride.

##### Pooling layer
Other than the convolutional layer, `pooling layer` is also a fundamental building block of the convolutional step. The purpose of pooling the output image is to reduce the data overfitting. Each feature value of output image reflects the feature of the corresponding pixels of the input. That means, even a slight movement of feature on the input image will result in a totally different feature map for the output. By using the pooling layers, the output image is generalized for similiar input images. Therefore, performing data augmentation such as mirroring or reversing the input data will generate consistent output image. To put it simple, pooling layers reduce the sensitivity of the activation values to the input data, and that achieves to lower the variance for the training result. 

There are mainly two types of pooling layer, `Max` and `average` pooling. The computation is fairly simple. The max pooling select the maximum feature value among the pooling blocks, while the average pooling averages out all the feature values within the pooling blocks. The process of this computation is also called `down-sampling` the features, meaning the signal of the input features are reduced to the lower resolution, and the feature details which are not useful for bettering the training result are discarded. 

Overall, a basic hidden layer of a convnet consists of a convolutional and pooling layers. And most of the modern convnet frameworks are finalized by stacking up those basic hidden layers in which the sizes of the filters and poolings are pre-defined.  

##### Motivation
Now back to the computer vision problems we have discussed in the beginning section. In order to enhance the training performance on high resolution input data, the size of the parameters need to be constrainted. But how organizing the neural network in the way of convet reduce the parameters to be trained? 

Recall that in a general neural network, the weight $w$ and the bias $b$ are the parameter to be trained for each hidden layer. In convnet, since the hidden layers use feature detectors to process the input data, the feature detector has become the parameters to train. Futhermore, a feature detector that is useful in one part of the image can probably be useful in other parts of the image as well, which means, the parameters of the feature detector can be shared across the image patches. Note that image patches here refer to the image blocks after the input image being equally devided by the size of the feature detector. Therefore, the feature detectors in the activation layers minimize the amount of parameters to train for the neural network, which is a boost to the training performance of the neural network.

Another benefit of using convnet layers is reducing the data overfitting of the output images. Each output value depends only on a small set of input pixels. Therefore, the output values become less sensitive to the input data, and that reduces the data variance for the neural network. This feature is called `sparsity of connections`, meaning the connection between input and output is partial.

### Modern convnet models
##### LeNet-5
`LeNet` is one of the most fundamental, also one of the earliest convnet models of all time. The model has been brought up and researched since 1988, and has been constantly improved in the following years and finally renamed to `LeNet-5`. The architecture of LeNet-5 is simple and straightforward, however, inspired the following researches about the convnet models on how to choose the parameters for the neural network. Many modern convnet models uses LeNet-5 as reference or even the fundamental building block for the convnet layers. 

The following is the architecture of LeNet-5 model.
![Site Image](/images/cnn/lenet.png)
The input data in this example has size of $32 \times 32$, and has RGB color channels. LeNet-5 has two hidden layers, each layer has one convolutional layer and one pooling layer. There are 6 filters with size of $5 \times 5$ for the first convolutional layer, followed by a $2 \times 2$ max pool with stide of 2. Thus the size of output values for the first hidden layer is $14 \times 14 \times 6$. For the second layer, the shape of the filters is $5 \times 5 \times 16$, while the following max pool has size of $2 \times 2$ and the stride is 2. Overall, the shape of the final output values after all the hidden layers is $5 \times 5 \times 16$.

Note that in order to apply softmax function to the output values, we need to flatten the output volume into a $n \times 1$ vector first, which is not shown in the graph. The dimension for the flatten layer is $5 \times 5 \times 16 = 400$, and consider this flatten layer as the input layer for the general neual network we have seen previously. The following two `fully connected` layers are just like the normal hidden layers where each layer has hidden units of weights and bias. They are called "fully connected" because all the hidden units are fully connected with each value of input layer. And finally we'll apply softmax function on the last fully connected layer to perform multi-classes classification on the objects.

Let's take a look at how many parameters in total need to be trained for this LeNet-5 model:
- two convolutional layers: $(5 \times 5 \times 3 + 1) \times 6 + (5 \times 5 \times 6 + 1) \times 16 = 456 + 2416 = 2872$
- two max pooling layers: $0$
- two fully connected layers: $(400 \times 120 + 120) + (120 \times 84 + 84) = 48120 + 10164 = 58284$
- softmax layer: $84 \times 10 + 10 = 850$

In total, there are only 62006 parameters to train. Comparing to using the general neural network where millions of paramters need to be trained, this is a huge improvement on the training performance for the neural network.

### Conclusion 
The building blocks for the convnet we have mentioned above are sufficient to perform multi-classes classfication on objects, however, in order to tackle the object detection problem where the position of objects also need to be identified, additional building blocks and algorithm are required. In the next article, we will look into how to implement object localization and YOLO algorithm for object detection. 

Hope you found this article useful, and I'll see you next time!



