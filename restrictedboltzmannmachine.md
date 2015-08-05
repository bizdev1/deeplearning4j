---
title: 
layout: default
---

# Restricted Boltzmann Machines

Restricted Boltzmann machines are shallow, two-layer neural nets that constitute the building blocks of deep-belief networks. In the paragraphs below, we will attempt to describe in math, diagrams and plain language how RBMs work. 

Invented by Geoff Hinton, [RBMs](../glossary.html#restrictedboltzmannmachine) are useful for [dimensionality reduction](https://en.wikipedia.org/wiki/Dimensionality_reduction), [classification](https://en.wikipedia.org/wiki/Statistical_classification), [collaborative filtering](https://en.wikipedia.org/wiki/Collaborative_filtering), [feature learning](https://en.wikipedia.org/wiki/Feature_learning) and [topic modeling](https://en.wikipedia.org/wiki/Topic_model). Given their relative simplicity, restricted Boltzmann machines are the first neural network we'll tackle.

The first layer of the RBM is called the visible, or input, layer, and the second is the hidden layer. 

![Alt text](../img/two_layer_RBM.png)

Each circle in the graph above represents a neuron-like unit called a *node*, and nodes are simply where calculations take place. The nodes are connected to each other across layers, but no two nodes of the same layer are linked.

That is, there is no intra-layer communication – this is the restriction in a restricted Boltzmann machine. Each node is a locus of computation that process input, and makes [stochastic](../glossary.html#stochasticgradientdescent) decisions about whether to transmit that input or not. (Stochastic means “randomly determined.”)

Each visible node takes a low-level feature from an item in the dataset to be learned. For example, from a dataset of grayscale images, each visible node would receive one pixel-value for each pixel in one image. (MNIST images have 784 pixels, so neural nets processing them have 784 input nodes on the visible layer.)

Let's follow that single pixel value, *x*, through the two-layer net. At node 1 of the hidden layer, x is multiplied by a weight and added to a so-called bias. The result of those two operations is fed into an activation function, which produces the node's activation, or the strength of the signal passing through it, given input x. 

		activation f((weight w * input x) + bias b ) = output a

![Alt text](../img/input_path_RBM.png)

Next, it is useful to see how many inputs combine at one hidden node. Each is multiplied by a separate weight, the products are summed, added to a bias, and the result is passed through an activation function to produce the node's output. 

![Alt text](../img/weighted_input_RBM.png)

So, RBMs’ nodes form a *symmetrical bipartite graph* where data passes through the visible layer on the left to the hidden layer on the right.

In the RBM below, the visible nodes receive several pixel values. Each visible node is connected with each hidden node, and at each hidden node, each input x is multiplied by its respective weight w. That is, a single input x would have three weights here, making 12 weights altogether. The weights between two layers will always form a matrix where the rows are equal to the input nodes, and the columns are equal to the output nodes. 

Each hidden node receives the four inputs multiplied by different weights. The sum of those products is added to a bias (which forces at least some activations to happen), and the result is passed through the activation algorithm producing one output a for each hidden node.

![Alt text](../img/multiple_inputs_RBM.png)

If these two layers were part of a deeper neural network, the outputs of hidden layer no. 1 would be passed as inputs to hidden layer no. 2, and from there through as many hidden layers as you like to a final classifying layer. 

![Alt text](../img/multiple_hidden_layers_RBM.png)

But in this introduction to restricted Boltzmann machines, we'll focus on how they learn to reconstruct unlabeled data by themselves, making several forward and backward passes between the visible layer and hidden layer no. 1 without involving a deeper network.

In the reconstruction phase, the activations of hidden layer no. 1 become the input in a backward pass. They are multiplied by the same weights, one per internode edge, just as x was on the forward pass. The sum of those products is added to a visible-layer bias at each visible node, and the output of those operations is a reconstruction; i.e. an approximation of the original input. This can be represented by the following diagram:

![Alt text](../img/reconstruction_RBM.png)

Because the weights of the RBM are randomly initialized, the difference between the reconstructions and the original input is often large. You can think of reconstruction error as the difference between the values of r and the input values, and that error is then backpropagated against the weights. 

A more thorough explanation of backpropagation is [here](../neuralnet-overview.html#forward). 

As you can see, on its forward pass, an RBM uses inputs to make predictions about node activations, or the probability of output a given weighted x: *p(a|x; w)*. 

But on its backward pass, when activations are fed in and reconstructions, or guesses about the original data, are the output, an RBM is attempting to estimate the probability of inputs x given activations a, which are weighted with the same coefficients as those used on the forward pass. This second phase can be expressed as *p(x|a; w)*. 

Together, those two estimates will lead you to the joint probability distribution of inputs *x* and activations *a*. 

Reconstruction does something different from regression, which estimates a continous value based on many inputs, and unlike classification, which makes guesses about which discrete label to apply to inputs. Reconstruction is making guesses about the probability distribution of the original input; i.e. the values of many varied points at once. 

Let's imagine that both the input data and the reconstructions are normal curves of different shapes, which only partially overlap. 

To measure the distance between its estimated probability distribution and the ground-truth distribution of the input, RBMs use [Kullback Leibler Divergence](https://www.quora.com/What-is-a-good-laymans-explanation-for-the-Kullback-Leibler-Divergence). A thorough explanation of the math can be found on [Wikipedia](https://en.wikipedia.org/wiki/Kullback%E2%80%93Leibler_divergence). 

KL-Divergence measures the non-overlapping, or diverging, areas under the two curves, and attempts to minimize those areas so that the activations of hidden layer one produce a close approximation of the original input. 

![Alt text](../img/KL_divergence_RBM.png)

By iteratively adjusting the weights according to the error they produce, an RBM learns to approximate the data. You could say that the weights slowly come to reflect the structure of the input, which is encoded in the activations of the first hidden layer. The learning process looks like two probability distributions converging, step by step.

![Alt text](../img/KLD_update_RBM.png)

Let's talk about probability distributions for a moment. If you're rolling two dice, the probability distribution for all outcomes looks like this:

![Alt text](https://upload.wikimedia.org/wikipedia/commons/1/12/Dice_Distribution_%28bar%29.svg)

That is, 7s are the most likely, and any formula attempting to predict the outcome of dice rolls needs to take that into account. 

Languages are specific in the probability distribution of letters, because each one uses certain letters more than others. In English, the letters *e*, *t* and *a* are the most common, while in Icelandic, the most common letters are *a*, *r* and *n*. Attempting to reconstruct Icelandic with a weight set based on English would lead to a large divergence. 

In the same way, image datasets have unique probability distributions for their pixel values, depending on the kind of images in the set. Pixels values are distributed differently depending on whether the dataset includes MNIST's handwritten numerals:

![Alt text](../img/mnist_render.png)

or the headshots found in Labeled Faces in the Wild.

![Alt text](../img/LFW_reconstruction.png)

The process of learning reconstructions is, in a sense, learning which groups of pixels tend to co-occur for a given set of images. (Imagine for a second an RBM that was only fed images of elephants and dogs, and which had only two output nodes, one for each animal. The question the RBM is asking itself on the forward pass is: Given these pixels, should my weights send a stronger signal to the elephant node or the dog node? And the question the RBM asks on the backward pass is: Given an elephant, which distribution of pixels should I expect?)

In the two images above, you see reconstructions learned by Deeplearning4j's implemention of an RBM. These reconstructions represent what the RBM's activations "think" the original data looks like. Geoff Hinton refers to this as a sort of machine "dreaming".

One last point. You'll notice that RBMs have two biases. This is one aspect that distinquishes them from other autoencoders. The hidden bias helps the RBM produce the activations on the forward pass (since biases impose a floor so that at least some nodes fire no matter how sparse the data), while the visible layer's biases help the RBM learn the reconstructions on the backward pass. 

Once this RBM learns the structure of the input data as it relates to the activations of the first hidden layer, then the data is passed one layer down the net. Your first hidden layer takes on the role of visible layer. The activations now effectively become your input, and they are multiplied by weights at the nodes of the second hidden layer, to produce another set of activations. 

This process of creating sequential sets of activations by grouping features and then grouping groups of features is the basis of a feature hierarchy, by which neural networks learn more complex and abstract representations of data. 

With each new hidden layer, the weights are adjusted until that layer is able to approximate the input from the previous layer. This is greedy, layerwise and unsupervised pre-training. It requires no labels to improve the weights of the network. 

Because those weights already approximate the features of the data, they are well positioned to learn better when, in a second step you try to classify images with the deep-belief network in a subsequent supervised learning stage.

### Initiating an RBM on Iris

Note how, below, an RBM is simply created as a layer in a NeuralNetConfiguration, a parameter fed into a more general class. Likewise, the RBM object is used to store properties like the transforms applied to the visible and hidden layers, Gaussian and Rectified Linear transforms, respectively. 

<script src="http://gist-it.appspot.com/https://github.com/deeplearning4j/dl4j-0.0.3.3-examples/blob/master/src/main/java/org/deeplearning4j/rbm/RBMIrisExample.java?slice=40:76"></script>

This is an example of an [RBM processing the Iris dataset](../iris-flower-dataset-tutorial.html), which we also cover in a tutorial. 

## Parameters & k

The variable k is the number of times you run [contrastive divergence](../glossary.html#contrastivedivergence). Each time contrastive divergence is run, it's a sample of the Markov chain composing the restricted Boltzmann machine. A typical value is 1.

In the above example, you can see how RBMs can be created as layers with a more general MultiLayerConfiguration. After each dot you'll find an additional parameter that affects the structure and performance of a deep neural net. Most of those parameters are defined on this site. 

**weightInit**, or weightInitialization, represents the starting value of the coefficients that amplify or mute the input signal coming into each node. Proper weight initialization can save you a lot of training time, because training a net is nothing more than adjusting the coefficients to transmit the best signals, which allow the net to classify accurately.

**activationFunction** refers to one of a set of functions that determine the threshhold(s) at each node above which a signal is passed through the node, and below which it is blocked. If a node passes the signal through, it is "activated."

**optimizationAlgo** refers to the manner by which a neural net minimizes error, or finds a locus of least error, as it adjusts its coefficients step by step. LBFGS, an acronym whose letters each refer to the last names of its multiple inventors, is an optimization algorithm that makes us of second-order derivatives to calculate the slope of gradient along which coefficients are adjust.

**regularization** such as **l2** helps fight overfitting in neural nets. Regularization essentially punishes large coefficients, since large coefficients by definition mean the net has learned to pin its results to a few heavily weighted inputs. Such a strong weight can make it difficult to generalize the net's model over new data. 

**visibleUnit/hiddenUnit** refers to the layers of a neural net. The visible unit, or layer, is the layer of nodes where input goes in, and the hiddenUnit is the layer where those inputs are recombined in more complex features. Both units have their own so-called transforms, in this case Gaussian for the visible and Rectified Linear for the hidden, which map the signal coming out of their respective layers onto a new space. 

**lossFunction** is the way you measure error, or the difference between your net's guesses and the correct labels contained in the test set. Here we use RMSE_XENT, or Root-Mean-Squared-Error-Cross-Entropy.

**learningRate**, like **momentum**, affects how much the neural net adjusts the coefficients on each iteration as it corrects for error. These two parameters help determine the size of the steps the net takes down the gradient towards a local optimum. A large learning rate will make the net learn fast, and maybe overshoot the optimum. A small learning rate will slow down the learning, which can be inefficient. 

### Continuous RBMs

A continuous restricted Boltzmann machine is a form of RBM that accepts continuous input (i.e. numbers cut finer than integers) via a different type of contrastive divergence sampling. This allows the CRBM to handle things like image pixels or word-count vectors that are normalized to decimals between zero and one.

It should be noted that every layer of a deep-learning net consists of four elements: the input, the coefficients, the bias and the transformation. 

The input is the numeric data, a vector, fed to it from the previous layer (or as the initial data). The coefficients are the weights given to various features passing through each node layer. The bias ensures that some nodes in a layer will be activated no matter what. The transformation is an additional algorithm that processes the data after it passes through each layer. 

Those additional algorithms and their combinations can vary layer by layer. The most effective continuous restricted Boltzmann machine we've found employs a Gaussian transformation on the visible (or input) layer and a rectified-linear-unit tranformation on the hidden layer. We've found this particularly useful in [facial reconstruction](../facial-reconstruction-tutorial.html). For RBMs handling binary data, simply make both transformations binary ones. 

*A brief aside: Geoff Hinton has noted and we can confirm that Gaussian transformations do not work well on RBMs' hidden layers, which are where the reconstructions happen; i.e. those are the layers that matter. The rectified-linear-unit transformations used instead are capable of representing more features than binary transformations, which we employ on [deep-belief nets](../deepbeliefnetwork.html).*

### Conclusions & Next Steps

You can intrepret RBMs' output numbers as percentages. Every time the number in the reconstruct is *not zero*, that's a good indication the RBM learned the input. We'll have a better example later in the tutorials. 

To explore the mechanisms that make restricted Boltzmann machines tick, click [here](../understandingRBMs.html).

Next, we'll show you how to implement a [deep-belief network](../deepbeliefnetwork.html), which is simply many restricted Boltzmann machines stacked on top of one another.