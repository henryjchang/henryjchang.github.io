---
layout: post
title: "Overview of Learnings from fast.ai’s Practical Deep Learning For Coders Course"
author: "Henry Chang"
categories: journal
tags: [deeplearning]
image: mountains.jpg
---

# Course Overview and Tooling

Of all the deep learning online courses I came across, this [Practical Deep Learning for Coders](https://course.fast.ai/) course by [fast.ai](fast.ai) was the most commonly recommended. I also felt it was the most suitable one for me since I wanted to be able to get hands on with training models early and often, rather than go through yet more material covering theory. I was also able to get introduced to various frameworks and resources I had wanted to start learning how to use, like PyTorch, Kaggle, and Huggingface. After having gone through the course material, I feel much more confident in my ability to quickly get a model up and running and quickly iterate to achieve improvements.

### Tooling
The course is primarily taught using the fastai library, built on top of pytorch. The fastai library contains a layered API to train models with varying levels of flexibility. The layered API contains a high-, mid-, and low-level API. The high-level API is used to quickly demonstrate deep learning for applications like image classification, natural language processing (NLP), and tabular and time series classification and regression. It’s quite opinionated about default values and abstracts away much of the complexity of training a model by providing building blocks with which the user can mix and match. The mid-level API is where we can specify transformations we want to apply to our input data if the high-level API cannot support the user’s use case. The low-level API provides abstractions for the fastai library to build the mid-level API; it’s not expected that the average library user touches the low-level API.

This latest iteration of the course (recorded in 2022) also makes use of various Huggingface features like their library of transformer models (Huggingface Transformers) and site to host webapps (Huggingface Spaces) built with Gradio. It makes heavy use of Kaggle for teaching and encourages students to apply their learnings to Kaggle competitions. Other tools that I wanted to learn about but weren’t used include Weights and Biases and TensorBoard to keep track of experiments.

The course is taught via video lectures and jupyter notebooks to follow along. There is an accompanying book whose chapters are also a series of jupyter notebooks.

### Course Contents
Machine learning can largely be divided into three areas: supervised learning, unsupervised learning and reinforcement learning. The course mostly focuses on supervised learning and dabbles a bit with unsupervised learning.

Given any type of input, a model can be trained to return any type of output: image to text, text to image, text to speech, image to image, etc. Machine learning can solve many types of problems but they broadly fall into the categories of either classification or regression. The course mostly goes over deep learning techniques but also demonstrates classical machine learning techniques for some applications and also shows how to leverage the classical techniques for initial insights during development and to use as baselines. 
The examples used in the course include tabular data prediction (Titanic passenger survival prediction given gender, class, family members onboard, and other features), image classification and multi-class categorization (using MNIST digit classification and animal/pet breed classification), natural language processing (US Patent Phrase to Phrase similarity classification), and recommendation systems (what movie to watch based on movies you and others have rated).

Along the way, the student is exposed to various techniques for efficiently achieving better performance. Techniques include data augmentation, test time augmentation, using a learning rate finder, progressive resizing, weight freezing, label smoothing, weight regulation, and ensembling. I’ll save discussion of these techniques for a future blog post.

# Deep Learning at a High Level
Several components are necessary to train a neural network model. These are: training and validation datasets and dataloaders, an architecture, an optimizer, and a smooth loss function.

### Datasets and Dataloaders
Common datasets can be downloaded from various libraries such as torchvision. Kaggle competitions also come with datasets provided.

A Dataloaders object is given a dataset (or a subset of a dataset) to load, transformations to apply to the data, and batch size. It provides the Learner object with a way to iterate through the data in minibatches. We typically want dataloaders for each of the training, validation, and test sets.

### Model Architecture
A neural network is simply a function approximator. Good function approximation can be achieved even with a single hidden layer if wide enough; this is the universal approximation theorem. However, spreading the same number of parameters across multiple layers makes a model easier to train. A deeper model can be used with fewer parameters, and achieve better performance and faster training.

Between each layer, there must be some activation function that introduces some nonlinearity that makes nonlinear function approximation possible. ReLU is a good activation function and is simply max(0, x). 

The activation function used at the output layer is different from that used for all the other layers. The model is trying to do some classification or regression. In order to do that, we must apply a softmax (for choosing a single class with the highest probability) or sigmoid (for choosing multiple classes with highest probabilities). Softmax applies exp() to all the activations so amplifies the differences between them and has probabilities sum to one, while sigmoid squishes all activations to be between a provided range of values, and each probability is independent of others’ probabilities.

While an activation function at the output layer is necessary for classification, it seems regression tasks do not need one since the output layer outputs a numeric value that should already be suitable.

### Optimizers
Although there are multiple optimizers that can be used for training, the course primarily demonstrates concepts with stochastic gradient descent (SGD). SGD is just applying gradient descent to update the parameters after each batch of data rather than going through all the training data before updating the parameters. This can lead to some stochasticity in the gradients as a batch is not representative of the entire dataset.

Some things not covered in the course are Adam and momentum.

### Loss Function and Metrics
The last major piece necessary for training a model is a loss function. We want loss functions to be smooth and differentiable. The loss is intended for the optimizer to use. In contrast, a human would probably prefer to consume a metric which does not need to be smooth. In some cases, a loss may be used as a metric.

Examples of loss functions are cross-entropy loss, RMSE (L2 loss), and MAE (L1 loss). Cross-entropy loss is typically used for classification problems. RMSE and MAE are typically used for regression problems.

A simple metric for classification could be accuracy. Other common metrics are precision, recall, F1, and area under the precision-recall curve, as well as area under the ROC curve.

For regression, RMSE and MAE are also easily interpretable metrics. However, in some cases (i.e. similarity comparison) the Pearson correlation coefficient could be used.

As an aside, I wanted to briefly mention one of the two Outstanding Main Track Award winners at NeurIPS 2023, [Are Emergent Abilities of Large Language Models a Mirage?](https://arxiv.org/abs/2304.15004) (Schaeffer, *et al.*, 2023). In the area of AI safety and alignment, emergent abilities have been a concern, and especially as part of the discourse with the rise of capable LLMs this past year and their abilities to do well on human benchmark tests like the SAT. However, the authors of the paper demonstrate that “emergent abilities appear due to the researcher's choice of metric rather than due to fundamental changes in model behavior with scale. Specifically, nonlinear or discontinuous metrics produce apparent emergent abilities, whereas linear or continuous metrics produce smooth, continuous predictable changes in model performance.” So it seems smooth metrics may also sometimes be desirable, and it may not be obvious even to experts.

## The fast.ai high-level framework

While we can write our own classes for each of these building blocks, Fast.ai’s high level API uses primarily two classes: DataLoaders and Learner. A DataLoaders object is used for organizing a training and validation dataset. A Learner object takes a DataLoaders object, an architecture, and metrics to display, and is able to choose good defaults for an optimizer and loss function. It automatically applies normalization, layer freezing, and discriminative learning rates for transfer learning.

Here’s an example for all the code needed to fine-tune a vision model on a pets dataset within a couple minutes on a single GPU and achieve pretty good accuracy, borrowed from [Fastai: A layered API for deep learning](https://arxiv.org/abs/2002.04688) (Howard and Gugger, 2020):

```
from fastai.vision.all import *
path = untar_data(URLs.PETS)
dls = ImageDataLoaders.from_name_re( path=path , bs =64 ,
fnames = get_image_files ( path/"images"), pat = r'/([^/]+)_\d+.jpg$',
item_tfms=RandomResizedCrop(450, min_scale = 0.75),
batch_tfms = [ * aug_transforms(size=224, max_warp = 0.), Normalize.from_stats(*imagenet_stats)])
learn = cnn_learner(dls, resnet34, metrics=error_rate)
learn.fit_one_cycle(4)
```

However, we’ll probably want to use the fast.ai mid-level framework or simply drop down to using only pytorch components in our own work.

# Transfer Learning

It’s demonstrated that it’s usually not necessary to train a deep learning model from scratch, depending on the application. Instead, we can easily fine-tune a pre-existing model, drastically reducing the amount of data and compute needed for training. This is called transfer learning and is done by removing the last output layers (the head) of the original model and replacing it with a new head for the desired application. If dealing with a text application, a pretrained language model should be finetuned, and if dealing with a vision application, a pretrained vision model should be finetuned. Ideally the base/foundation model is trained on a similar enough corpus to your dataset. Since most of the early layers of a neural network learn features that are generally applicable, we don’t want to modify those layers too much - we leave their weights frozen. It’s also simpler and faster to finetune a model like this. The only weights that need to be modified would be those of the last layer (that we replaced and whose weights we re-initialized). Of course if there is a larger difference between the dataset used for pre-training and the dataset we care about, the weights belonging to earlier layers would also be modified via discriminative learning rates. A simple starting point would be to set the learning rate of the last layer and linearly decrease the learning rate for each layer moving toward the front in the architecture. 

# Tricks for Achieving Better Training Results
Most of these techniques were discussed in the context of image classification, but are concepts general to other learning tasks as well.

### Data Augmentation
To help a model generalize and have higher accuracy, augmented data can be added to the dataset. For vision tasks, images can be cropped, padded, rotated, flipped, squished, and have changes in brightness. Chaining together multiple transformations can lead to unwanted artifacts, since some pixel values may need to be filled in; it’s better to do all the transformations in one pass then fill in pixel values. It’s good practice to presize to a larger image first.

One more advanced technique, [mixup](https://arxiv.org/abs/1710.09412), is to take a linear combination of images and the same linear combination of their labels. This technique reduces a model’s sensitivity to adversarial examples and can provide much higher accuracy, especially when you don’t have much data and don’t have a pre-trained model to fine-tune on. Note this technique takes more epochs to get good results.

### Test Time Augmentation
Similarly, at test-time, an image can be transformed in multiple ways and a prediction can be made for each transformed image. To come up with a final prediction, predictions can be averaged or combined in a different way to achieve more robust results.

### Learning Rate Finder
Guessing good learning rates can be painstakingly slow. Consider using a [learning rate finder](https://arxiv.org/abs/1506.01186) to get a learning rate vs loss plot to help choose the optimal learning rate. The finder iterates through learning rates and does small runs on batches and plots the loss. One is implemented as part of the fast.ai API, as well as in [PyTorch Lightning](https://pytorch-lightning.readthedocs.io/en/1.4.9/advanced/lr_finder.html).

### Discriminative Learning Rates
In transfer learning, we tend to want to leave the parameters of the early layers the same and change the later layers more. The early layers contain more general information such as detecting edges and shapes whereas the later layers contain information more specific to the dataset. We can use smaller learning rates for the early layers and larger learning rates for the later layers.

### Weight Freezing
In transfer learning, when we fine-tune a model, we “freeze” the parameters of each layer up to the new head. After an epoch or so, we unfreeze the weights and can use discriminative learning rates.
Progessive Resizing
When training a model, we can start with a small image size during initial training and progressively increase the image size as training progresses. This approach can help with faster initial convergence, with better memory efficiency and avoiding overfitting. By beginning with smaller images, basic features can still be learned and larger batch sizes can be used. This technique is introduced [here](https://colab.research.google.com/github/fastai/fastbook/blob/master/07_sizing_and_tta.ipynb).

### Ensembling
There are multiple ways to put together an ensemble of models to achieve better performance. Two concepts are bagging and boosting. 

Bagging (short for bootstrap aggregation) reduces variance and helps to avoid overfitting by training a set of smaller models on data sampled with replacement from the same dataset. We take the mean of the predictions of each model.

Boosting is a technique for taking a series of weak learners and summing their outputs to form a strong learner. Each successive weak learner is trained on the residual of the previous weak learner.

### Label Smoothing
This regularization technique helps the model be less confident in its predictions and be less prone to overfit. It modifies the labels such that instead of a one-hot encoding, we have some small $\frac{\epsilon}{N}$ for each incorrect label and and $1 - \epsilon + \frac{\epsilon}{N}$ for the correct label. So instead of always having a nonzero gradient toward an unreachable value of 1 (after sigmoid or softmax), it can settle at the smoothed target. This helps to keep the weights small leading up to the final activations. This technique was introduced in this [paper](https://arxiv.org/abs/1710.09412).

### Weight Decay
Weight decay is another regularization technique by adding an additional cost to the loss function. It’s simply placing a cost weight on the square of the parameters, keeping the weights and biases small.

### Dropout
Dropout is a regularization technique to help a model generalize and not be too reliant on any single neuron. During training, a random subset of parameters are set to zero, or “dropped out”, so that the model must be able to predict the same label from different subsets of features. This can also be seen as a kind of ensembling technique. During test time, no neurons are dropped out, and the weights are scaled down by the proportion of neurons that had been dropped out during the training phase.  

### Gradient Accumulation
If we’re dealing with large models such that we can’t iterate with a large enough batch size, instead of getting a more expensive GPU, we can leverage gradient accumulation. Instead of updating the model parameters after every batch, we can accumulate the gradients for some specified number of batches before updating the parameters. Notice that we don’t want to zero the gradients until after the parameters have been updated. Gradient accumulation allows us to reduce the amount of memory used without any loss of training stability. See [here](https://www.kaggle.com/code/jhoward/scaling-up-road-to-the-top-part-3 ) for an example.


# Practical Tips

When trying to train the best model, it’s recommended to focus on 1) Creating an effective validation dataset, 2) iterating quickly to find changes that achieve better results on the validation set. 

Good steps to follow after importing the data are:
1. Exploratory data analysis (EDA)
2. Create a validation dataset. Think about whether it makes sense to choose a random reshuffle of the training dataset or if certain factors matter like time or contextual clues that can be confounding factors.
    * NB: If working with text, a tokenizer and numericalizer is needed - pre-trained models have their own unique version of each, so the pre-trained must be chosen first.
3. Pick a batch size that fits the GPU. The GPU can run out of memory if using a larger model and large batch size. If training is CPU bounded, try resizing to smaller, approximate data. And if GPU memory allows, try fine-tuning a better, larger pre-trained model basically for free.
4. Train a model for a few epochs several times and see if the results are stable so we can know if improvements we’re seeing are due to the changes we’ve made.
5. Set up code such that we can quickly try different data processing and training parameters
6. Try out different ideas and see if they improve results. Some ideas are:
    * try a pre-trained model that was trained on data more similar to our dataset,
    * fine-tuning the pretrained model on a larger relevant dataset before fine-tuning on this dataset
    * Data augmentation, regularization, ensembling, and other ideas mentioned above
7. Once the best ideas have been identified, scale up the data to the original size and train for more epochs.
8. Retrain on entire training set and use cross-validation to get more minor improvements, and consider ensembling if there aren’t strong constraints on inference time.

Check out [this notebook](https://www.kaggle.com/code/jhoward/iterate-like-a-grandmaster) for more details.
