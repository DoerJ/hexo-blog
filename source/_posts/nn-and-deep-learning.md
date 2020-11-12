---
title: Boil Down the Neural Network and Deep Learning
date: 2020-11-06
tags: AI, neural network, deep learning
---
### The nature of neural netwrok 
You can think of neural netwrok as a middlelayer that makes machine have "perception" as human to the objects in the real world. The analogy of neural network is taken from the human brain. Human perceive signal inputs(i.e., the light, sound, etc) form the outside world, and the brain takes in those inputs, process in neurons and progagate through synapse, finally form the output which is the judgement to the received signals. The neural network for the machine acts similar. The machine takes in input, which is normally represented as a matrix of entities(e.g: pixels for image), compute inputs with `linear` and `activation` functions for each layer so called `hidden layer`, progagates activation vectors throughout all the hidden layers, and finally reaches to the output layer in which the predicted value of output is formed. 

Before making a neural network perform well on predicting the outputs, the networks needs to be trained. The performance of a neural netwrok is closely related to the `weights` and `bias`, which are the two parameters applied to the linear function for each layer. Each time we pass in a training example which is normally labelled as a pair of input(matrix of entities) and output(the result identified by human), the weight and bias for each layer will be updated according to the cost function(the function evaluates the difference between the correct output and the predicted value). Therefore, the size of training dataset is a huge factor to the performance of a neural network. To have a large training dataset, we need to have sources of data, and of course, the computation power that is strong enough to iteratively process through each data example efficiently. This is also one of the reasons why neural networks really took off only in recent years, even thought the concept has been brought up as early in 90s, because at that time there is no online application or cloud platform at which huge sets of user data can be collected and shared for the purpose of neural network training. 

### Binary classification problem
The binary classification problem refers to the problem where the machine needs to identify an object on a true/false basis. For example, whether it's a image of an animal, or furthermore, whether the input image contain any inappropriate content. The bianry classification has a very wide range of application such as online advertisement and automatic reviews of user uploads of images or videos. It's also the simplest form of problems that can be described and solved by neural network. 

The process of training a neural network for a binary classification problem can be divided into two phases: `forward propagation` and `backward propagation`. Forward propagation is where the input gets passed through each layer, applied with linear and activation functions, and cache the activation vectors of each layer for the later use of backward propagation. The backward propagation is where the partial derivatives of parameters, that is, weight and bias, with respect to the cost function are derived. Finally, we upate parameters for each layer and repeat the same process for a number of iterations.  

### Architecture of a one-layer neural network
![Site Image](/images/one-layer-nn.png)
The image above illustrates the model of a neural network with an output layer only. The input image is usually represented as a column vector of pixels in RGB orders. The linear function applied to the input vector will be $z = w^T x + b$, where w is the weight, and b is bias. The shape of the matrix of weights is (# of training examples, size of training example) while the dimension of bias is (size of training example, 1). Intuitively, we can think of each pixel value in the input vector will have a corresponding weight and bias acting on them, therefore, by inputing more and more tranining examples the weight of bias will be adjusted until reaches to a high accurancy. 

##### activation function 
The core part of the learning algorithm is `activation function`, denoted as $\hat{y} = a = sigmoid(z)$ which takes in the parametrized input vector, and outputs the prediction value $\hat{y}$. Now comes to the question that how do we choose a mathematical model as the activation function so the output "makes sense". For binary classification problems, the predicted value needs to land within [0, 1], which indicates the propability of the input image being a certain object. Therefore, the output layer usually chooses `logistic regression` as the activation function, expressed as $a = A(w^T x + b) = \frac{1}{1 + e^{-z}} = \frac{1}{1 + e^{-(w^T x + b)}}$. If look closely to the equation, we notice that if $z$ tends to be large, $A$ becomes close to 1. Oppositely, if $z$ becomes very small(largely negative), the activation value will get close to 0, which perpectly aligns with the output range we want.  

##### cost function  
After getting the prediction value, we need to measure how accurate our prediction is and adjust the parameters according to the differences. The learning algorithm for measuring the accurancy of prediction is also known as `cost function`, and the formula is: $\mathcal{L}(a^{(i)}, y^{(i)}) =  - y^{(i)}  \log(a^{(i)}) - (1-y^{(i)} )  \log(1-a^{(i)})$, where $a$ is the activation function with respect to $w$ and $b$, and $(i)$ refers to the $ith$ training example. Furthermore, if we have $m$ training examples, we need to sum up the value of cost function for each training example, and take the average by dividing the sum with $m$. The cost function which iterates through $m$ training examples is: $J = -\frac{1}{m}\sum_{i=1}^{m}y^{(i)}\log(a^{(i)})+(1-y^{(i)})\log(1-a^{(i)})$. The following graph shows what the cost function looks like in cartesian coordinate system.
![Site Image](/images/gradient-descent.png)
The horizontal axis of the graph of the function are $w$ and $b$ respectively, and the vertical axis is the function value which is the difference between the prediction and the actual result. Notice that the function is convex where any two points on the graph of the function lies above the graph, meaning that the function has a `global minimum` that indicates the smallest prediction error. Our goal is to find the $w$ and $b$ so the corresponding value of the cost function is the global minimum. In order to update the parameters $w$ and $b$ in a way so the cost function value move towards the global minimum, we need to introduce `gradient descent`. 

##### gradient descent 
Gradient descent is an optimization algorithm which iteratively takes step in the direction of steepest descent of the graph until reaches to a local minimum. The formulas for updating parameters $w$ and $b$ are as follow:
$\frac{\partial J}{\partial w} = \frac{1}{m}X(A-Y)^T$
$\frac{\partial J}{\partial b} = \frac{1}{m} \sum_{i=1}^m (a^{(i)}-y^{(i)})$
$w = w - \alpha \text{ } dw$
$b = b - \alpha \text{ } db$

$\alpha$ is the learning rate which defines how big the step is for each iteration of gradient descent. $\frac{\partial J}{\partial w}$ and $\frac{\partial J}{\partial b}$ are the derivatives, which are also the slopes of the cost function $J$ with respect to $w$ and $b$. The process of taking the partial derivatives of the cost function with respect to the parameters and updating the parameters with the learning rate is called backward propagation. 

##### hyperparameters
At last, we reapeat the same processes above for a number of iterations which tells how many steps to take in gradient descent. Notice that the parameters such as the steps for gradient descent and learning rate are called `hyperparameters`. Hyperparameters are all predefined and have direct impact on the performance of training the neural network. To get the optimized values of these hyperparameters, there is usually an experiment phase where the developer tests out different values of the hyperparameters and adjust accordsing to the graph of the performance of the neural network. 

### Architecture of a deep neural network
A deep neural network consists of at least three layers: the input layer, one hidden layer, and the output layer. Comparing to the logistic regression model which has only the input and output layers, the deep neural network has hidden layers as the middlelayers which are not observable to the outside world. A hidden layer has hidden units which refer to the `nurons` in anology of neural network, and each nuron has the dimension of (# of hidden units in the current layer, # of input features). Same as the learning rate, the number of hidden units in each layer is also a hyperparameter which is predefined by the developer. Normally, the less hidden units each layer has, the deeper the neural network needs to be in order to matain high accurancy on the prediction value.
##### Intuition 
The overall reason of having a multi-layer over a two-layer neual network is that the neural network with multiple hidden layers has better performance. You can think of the each layer of hidden units are computing different task regarding the input data. For example, when the input data is an image of a dog, the first hidden layer can be computing the edges of the image, and the second hidden layer detects the contour of the dog, while the rest of the hidden layers may compute the explicit features of the dog. Intuitively, the early layers of a deep neural network detects the simple features of the input data such as the edges of an image, and composes those simple features together in the later layers to compute more complex features. When comes to the problems that are more complicated than the binary classification such as object recognition, the advantage of deep neural network will become more obvious.

##### activation function 
![Site Image](/images/deep-layer-nn.png)
The figure above shows the activation functions used across a $L$-layer neural network. Notice that layer $0$ to $L - 1$ use `Relu` as the activation function while the activation function for the output layer is `sigmoid`. The reason behind the choices is that to centralize the data better for the next layer, the hidden layers usually choose hyperbolic function such as `tanh` or Relu as the activation functions, thus the output value for the next layer will be close to 0 after taking the average through $m$ tranining examples. However, the activation function for the output layer must be `sigmoid` function since the prediction value to the initial input image needs to be between 0 and 1. 

##### backward propagation 
The backward progagation in a deep neural network is similar to two-layer neural network, except the weights $w$ and bias $b$ need to be updated for each hidden layer. The following are the formulas needed to update the parameters:
$dW^{[l]} = \frac{\partial \mathcal{J} }{\partial W^{[l]}} = \frac{1}{m} dZ^{[l]} A^{[l-1] T}$
$db^{[l]} = \frac{\partial \mathcal{J} }{\partial b^{[l]}} = \frac{1}{m} \sum_{i = 1}^{m} dZ^{[l](i)}$
$dA^{[l-1]} = \frac{\partial \mathcal{L} }{\partial A^{[l-1]}} = W^{[l] T} dZ^{[l]}$

Note that the superscript $l$ stands for the layer $l$. For example, $W^{[l]}$ refers to the weights vector in layer $l$. 

### Conclusion 
Generally, the training process for a neural network can be summarized into the following four steps:
1. Forward propagation 
2. Compute the cost function to check whether the neural netwrok is actually learning
3. Backward propagation 
4. Update parameters $w$ and $b$

This article only gives a brief overview of how a neural network learns based on the given trainining examples, and there are other aspects of the implementation for the learning algorithm such as the initialization of the parameters, and `vectorization` which is a form of parallel computation to get rid of explicit for loops. But overall, the four steps above are the most essential functions for a learning algorithm, and these steps build the backbone of a neural network. 

I hope you found this artical useful, and have some intuitions of deep neural network by now. Stay tuned for more of my thoughts and learnings on the machine learning, and see you next time!












