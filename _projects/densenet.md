---
layout: page
title: DenseNet API
description: A simple Artificial Neural Network API for Supervised Learning using Python.
img: /assets/img/densenet.png
importance: 4
---

Github : [https://github.com/staveesh/DenseNet-API](https://github.com/staveesh/DenseNet-API)

Collaborators : Taveesh Sharma, Kushagr Arora, Vishal Agrawal

The API currently supports:

##### 1. Activation functions:

  * ReLU
  * Sigmoid
  * Softmax
  * Linear

##### 2. Loss functions:

  * L1 loss
  * L2 loss
  * Cross Entropy
  * SVM loss

##### 3. Optimisers:

  * SGD 
  * SGD with momentum

### Using the API:

```python
import numpy as np
from DenseNet import DenseNet
from Graph_API import Optimiser

opti = Optimiser(learning_rate=0.09, momentum_eta=0.5)

X_train = # Inputs
Y_train = # One-hot encoded Labels
net = DenseNet(X_train.shape,opti,'cross entropy') # 'l1' for L1 loss, 'l2' for L2 loss, 'svm' for svm

```

To add a hidden/output layer, simply call addlayer function with activation function and number of neurons.

 ```python
net.addlayer(activation='sigmoid',units=4)
net.addlayer(activation='relu', units=3)
```

For output layer, add units same as the number of classes

```python
net.addlayer(activation='softmax',units=y_train.shape[1])
```

The train function runs SGD for 1 iteration only. Call it for multiple iterations for training.

```python 
iterations = 10000
for i in range(iterations):
  error = net.train(X_train,Y_train)
```

To make predictions use :

```python
X_test = #test data
predictions = net.predict(X_test)
```