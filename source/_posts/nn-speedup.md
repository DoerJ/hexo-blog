---
title: Approaches of Optimizing Neural Network
date: 2020-12-05
tags: AI, machine learning, optimization
index_img: /images/thumbnail/nn-speedup.png 
---
### Optimization of neural network 
Building up and training a neural network model could be time-consuming. To make the model robust on predicting based on the given input features, the training data must be large enough to minimize the variance so the model has acceptable performance on fitting the general testing data. When the training data gets large, the models usually takes long time to run through one single iteration and update the parameters. That makes developers hard to inspect the progress of the learning process and adjust the hyperparameters accordingly. To speed up the model learning and optimize the model perfomance, some additional techniques need to be applied to processing input data and gradient descent. 

### Process the input data
In some degress, the slow learning proces of the model is due to the large amount of the input data. Imagine when performing the object detection tasks, which requires much more input dataset than the usual binary classification problem, and the input pictures are normally high resolution that contains fair amount of input features. Feeding the model with a whole set of input pictures in one iteration is sometimes too much in terms of the learning time. 

##### Normalization 
`Normalization` scales the input data values into a certain range, normally $[-1, 1]$, while still maintaining the one-to-one mapping between X and Y. Consider one of the training examples has two input features $[x_1, x_2]$, where $x_1$ ranges from 100 to 100,000 while $x_2$ ranges from 1 to 10. Recall that the linear combination of parmeters and input features is $z = w^T x + b$, thus the parameters $w$ and $b$ will both exist on the same scale as $x$ which is the input features. Since $w_1$ and $w_2$ are on the different scales, the cost function will most likely be elongated. 
![Site Image](/images/nn-optimization/elongated-contour.png)
The graph above shows the elongated contour of a cost function with different scaled parameters, and the graph on the right is the contour of the cost function using normalization. Without using normalization, to make the learning path converge towards the minimum point, the parameters $w$ and $b$ have to take different proportion of steps, and more importantly, the learning rate of the gradient descent may have to be small enough so the learning path won't diverge on the cost function. The smaller the learning rate is, the more iterations need to take for the training process. After using normalization, the contour of the cost function becomes more symmetric and the parameters $w$, $b$ are now taking equal proportional steps towards the convergence. Therefore, adding a layer of normalization to the input data is essential for speeding up the learning rate. 

The standard procedure of normalization usually involves `subtracting mean` and `normalizing variance`. The following are the equations used in normalization:
$\hat{x} = \frac{1}{m}\sum_{i=1}^m x^{(i)}$,
$x = x - \hat{x}$,
$\varphi^2 = \frac{1}{m}\sum_{i=1}^m (x^{(i)})^2$,
$x = \frac{x}{\varphi^2}$  

##### Mini-batch
`Mini-batch` is a technique of adding another layer of processing the input data to fasten the learning iteration. The core idea of mini-batch is to partition the input dataset into equal-sized mini-batches, and instead of updating parameters(weight $w$ and bias $b$) after iterating over all training examples, the model with mini-batches is updated on the batch basis, that is, one mini-batch is considered as one iteration of updating. For example, the size of training examples is 100,000 and the batch size is 5,000, then there will be $100,000/5,000 = 20$ batches in total. Thus the model parameters will be updated every 5,000 training examples and that makes the iteration for the gradient descent way much faster than before. 

There are two phases involved when apply mini-batch to the input data. The first step is to `shuffle` the input set (X, Y) to make sure every batch of input data has random data distribution and has no correlations with the sibling training examples. Note that the shuffle must be synchronous between X and Y, meaning that X and Y after the shuffle will still have the same one-to-one mapping as before. The second phase is `partition`, where the entire dataset will be divided into batches of equal sizes. The final look of input data in mini-batches will look like below:
![Site Image](/images/nn-optimization/mini-batch.png)

However, there is downside of using mini-batches for the gradient descent. Since the parameters will be updated after seeing only a subset of the training examples, each step the gradient descent is taking may not be converging d  irectly towards the local minimum point. Thus the downhill learning path may oscillate, and that also results in slowing down the learning process for the model to minimize the overall prediction cost. The following is the contour of the cost function graph from looking above, and shows what the learning path will look like during the gradient descent with using mini-batches. 
![Site Image](/images/thumbnail/nn-speedup.png)

To reduce this oscilation effect, we have to use some other approaches to smooth out the learnig path throughout the gradient descent. 

### Optimize gradient descent
##### Gradient descent with momentum 
As we can see earlier, the learning path of the gradient descent appears to be oscillate with using mini-batches. `Momentum` is one way to reduce the oscillation effect and make the learning path smoother. Before getting into the gradient descent with momentum, there is one more concept that needs to be introduced called `exponentially weighted average`. 

Exponentially weighted average is a nice way to describe the trend of the data. What it does is to iteratively place wieghts on the most recent data. For example, consider the daily prices of a stock in the market is shown as $[x_0, x_1, x_2, ..., x_n]$. The exponentially weighted average of the daily price values can be expressed as:
$v_0 = x_0$,
$v_1 = \beta{v_0} + (1 - \beta){x_1}$,
$v_2 = \beta{v_1} + (1 - \beta){x_2}$,
$...$,
$v_n = \beta{v_{n-1}} + (1 - \beta){x_n}$
Note that $x_n$ is the actual data value on day n, $v_n$ is the exponentially weighted average value of day n, and $\beta$ is the weight. Consider the weight $\beta$ to be 0, then $v_n = x_n$ and the trending graph will be the data points of the daily prices, which may look oscillate and jiggle. If the weight is 1, then $v_n$ will totlly depend on the previous average values and the data value of the current day will have no contribution at all, and the trending graph will becomes a smooth curve. Therefore, finding a proper $\beta$ value between $[0, 1]$ can reduce the oscillation of the graph, and the same rule can also be applied to the cost function. 

Momentum takes the idea of exponentially weighted average, takes the previous gradients into account during the parameter updates to smooth out the learning path. The following is the implementation of the gradient descent with momentum:
$v_{dW^{[l]}} = \beta v_{dW^{[l]}} + (1 - \beta) dW^{[l]}$,
$W^{[l]} = W^{[l]} - \alpha v_{dW^{[l]}}$,
$v_{db^{[l]}} = \beta v_{db^{[l]}} + (1 - \beta) db^{[l]}$,
$b^{[l]} = b^{[l]} - \alpha v_{db^{[l]}}$
Recall that $dw$ and $db$ are the gradients on the parameters $w$ and $b$, $l$ is the number of layer of neural network, and $\beta$ is the weight used for computing the average value. 

##### RMSprop
`RMSprop` stands for "Root Mean Square prop", which is also a technique used for speeding up the gradient descent learning. Since the gradient descent learning is based on both parameters $w$ and $b$, the learning speed for $w$ and $b$ may be different in order to converge to the minimum point. Consider the vertical direction to be $w$ axis, and horizontal direction is $b$ axis in cost function. The oscillations in vertical direction may be faster than in horizontal direction, or in the opposite way. In that case, we want the learning speed to be faster in one direction, and slower in another direction. 

The core implementation for RMSprop is to divide the gradient by the running average of the recent gradients. The followings are the equation to udpate the parameters:
$S_{dw} = \beta{S_{dw}} + (1 - \beta)dw^2$,
$S_{db} = \beta{S_{db}} + (1 - \beta)db^2$,
$w = w - \alpha{\frac{dw}{\sqrt{S_{dw}}}}$,
$b = b - \alpha{\frac{db}{\sqrt{S_{db}}}}$
The algorithm above determines the learning speed based on the gradient. For example, notice that if the partial derivative $dw$ is large, which means the gradient in $w$ axis is large, then $\frac{dw}{\sqrt{S_{dw}}}$ in updating $w$ will become small, and the learning speed for $w$ will become small as well. Oppositely, if the gradient of $w$ turns to be small, $w$ will be updated with a large step. By using RMSprop, the model can have a resonable big learning rate without having the learning path diverged. 

##### Adam optimization 
`Adam optimization` is a combination of RMSprop and the gradient descent with momentum, and takes advantages of the both to smooth out and speed up the convergence to the local minimum point of the cost function. Adam optimization is also an adaptive learning algorithm, which means the algorithm computes separated learning rate for the different parameters. Therefore, the model can self-adjust and optimize the learning path in gradient descent, making sure the model is taking the "fastest" route downhill the cost function. 

Adam optimization involves two steps. The first step is to compute the exponentially weighted average of the previous gradients, and stores the value in a cache $v^{corrected}$. The next step is to compute the running average of the square of the previous gradients, and stores in $s^{corrected}$. If look closely, the two steps above are the implementations of momentun and RMSprop respectively. By taking the example of updating the parameter $w$, the detailed implementation of Adam optimization is as follow:
$v_{dW^{[l]}} = \beta_1 v_{dW^{[l]}} + (1 - \beta_1) \frac{\partial \mathcal{J} }{ \partial W^{[l]} }$,
$v^{corrected}_{dW^{[l]}} = \frac{v_{dW^{[l]}}}{1 - (\beta_1)^t}$,
$s_{dW^{[l]}} = \beta_2 s_{dW^{[l]}} + (1 - \beta_2) (\frac{\partial \mathcal{J} }{\partial W^{[l]} })^2$,
$s^{corrected}_{dW^{[l]}} = \frac{s_{dW^{[l]}}}{1 - (\beta_2)^t}$,
$W^{[l]} = W^{[l]} - \alpha \frac{v^{corrected}_{dW^{[l]}}}{\sqrt{s^{corrected}_{dW^{[l]}}} + \varepsilon}$
$\beta_1$ and $\beta_2$ are the two hyperparameters used for computing the running averages of the previous gradient, and $\varepsilon$ is a very small number that avoids the denominator being 0. Similarly, the parameter $b$ will be applied with the same steps above. 

### Some other notes 
The gradient descent with momentum is an efficient approach to reduce the oscillations of the learning path, however, the algorithm may has difficulty working well with mini-batches. Since momentum computes the exponentially weighted average of the recent gradients, when the learning iteratives through to the next mini-batch of the input data, the running average of the gradient will be reset and lose the information on the previous mini-batches. Therefore, the learning path will still look a bit "jiggle" whenever transit to the next mini-batch. On the other hand, Adam optimization performs much better on working with mini-batches than using momentum, and also converges much faster. 

I hope you find this article useful. Stay tuned for more of my thoughts on machine learning!


