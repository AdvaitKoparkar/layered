[![Code Climate][1]][2]

[1]: https://codeclimate.com/github/danijar/layered/badges/gpa.svg
[2]: https://codeclimate.com/github/danijar/layered

Layered Neural Network
======================

This project is aims to be a clean reference implementation of feed forward
neural networks in Python 3 under the MIT license. It's part of my efforts to
understand the concepts of deep learning.

You can use this repository when doing your own implementation of neural
networks which I highly recommend if you are interested in understanding them.
It makes sure you correctly understand all the details. For example, I had a
small misunderstanding of the backpropagation formula. My network still trained
but I found the mistake by numerical gradient checking.

Instructions
------------

If you have Matplotlib for Python 3 installed on your machine, you can just run
this command. To tweak the network and problem definition, just edit the same
file.

```bash
python3 main.py
```

Problem Definition
------------------

Learning problems are defined in YAML. Here is an example for classifying
handwritten digits.

```yaml
dataset: Mnist
cost: Squared
layers:
  - size: 784
    activation: Linear
  - size: 500
    activation: Relu
  - size: 300
    activation: Relu
  - size: 10
    activation: Sigmoid
training_rounds: 5
batch_size: 10
learning_rate: 0.25
momentum: 0.3
weight_scale: 0.01
weight_decay: 1e-3
evaluate_every: 5000
```

Quick Start
-----------

### Network Definition

In this guide you will learn how to create and train models manually. A network
is defined by it's layers. The parameters for a layer are the amount of neurons
and the activation function. The first layer has a linear activation since the
input layer shouldn't transform the data.

```python
from layered.network import Network
from layered.activation import Linear, Relu, Sigmoid

network = Network([
    Layer(num_inputs, Linear),
    Layer(700, Relu),
    Layer(500, Relu),
    Layer(300, Relu),
    Layer(num_outputs, Sigmoid)
])
```
### Activation Functions

| Function | Definition | Derivative | Graph |
| -------- | :--------: | :--------: | ----- |
| Linear | x | 1 | ![Linear activation](image/linear.png) |
| Sigmoid | 1 / (1 + exp(-x)) | y * (1 - y) | ![Sigmoid activation](image/sigmoid.png) |
| Relu | max(0, x) | 1 if x > 0 else 0 | ![Relu activation](image/relu.png) |

### Weight Initialization

Then we create the weights matrices for the network. We'll hand the this object
to algorithms like backpropagation, gradient decent and weight decay.

If the initial weights of a neural network would be zero, no activation would
be passed to the deeper layers. So we set them to random values of a normal
distribution.

```python
from layered.network import Matrices

weights = Matrices(network.shapes)
weights.flat = np.random.normal(0, weight_scale, len(weights.flat))
```

### Optimization Methods

We want to learn good weights using backpropagation and gradient decent.
Therefore, the classes for this can be imported from the `gradient` and
`optimization` modules. We also need to decide for a cost function here.

```python
from layered.cost import Squared
from layered.gradient import Backprop
from layered.optimization import GradientDecent

backprop = ParallelBackprop(network, cost=Squared())
decent = GradientDecent()
```

### Gradient Algorithms

- Backprop
- BatchBackprop
- ParallelBackprop
- NumericalGradient
- CheckedBackprop

### Cost Functions

| Function | Definition | Derivative | Graph |
| -------- | :--------: | :--------: | ----- |
| Squared | (pred - target) ^ 2 / 2 | | ![Squared cost](image/squared.png) |
| CrossEntropy | -((target * log(pred)) + (1 - target) * log(1 - pred)) | | ![Cross Entropy cost](image/cross-entropy.png) |

### Dataset and Training

Download the four files of the MNIST dataset from Yan LeCun's website.

```python
from layered.dataset import Mnist

dataset = Mnist('dataset/mnist')
for example dataset.training:
    gradient = backprop(network, cost)
    weights = decent(weights, gradient, learning_rate=0.1)
```

### Evaluation

Then evaluate the learned weights on the testing examples.

```python
import numpy as np
from layered.utility import averaged

error = averaged(examples, lambda x:
    float(np.argmax(x.target) !=
    np.argmax(network.feed(weights, x.data))))
print('Testing error', round(100 * error), '%')
```

Contribution
------------

Feel free to file pull requests. If you have questions, you can ask me.
