---
title: A key is made, but what is the lock?
author: Shuai Huang
date: '2021-09-22'
slug: key-and-lock-paradox
categories:
  - Example
tags:
  - deep learning
  - machine learning
  - paradox
---

During a class lecture about deep learning, a student asked me what that key on the slides was made for. The “key” he mentioned was actually an architecture of a deep net model that was built to detect surgical site infection based on wound images. It looked like a key, while in a certain sense it indeed was a key since it unlocked the mystery about a given wound image - was there infection or not? But what was still puzzling was that, now there was a key, but what did the lock look like?

This is a paradox in data science practices. Speaking of deep learning, on the one hand it becomes increasingly complicated, with all the rapidly advancing modeling techniques, the massive scale of the models, and the need for big data and enormous computational power. There is no way one can write out the entire mathematical form of the deep net one has designed. On the other hand, with tools like Tensorflow, it becomes possible to take on a mentality not too different from when one is playing lego, i.e., as Raul Rojas in his seminar book <Neural Networks: a Systematic Introduction> had pointed out, to explain the logic of designing neural networks, "... reason in this sense is nothing but reckoning, that is adding and subtracting ...” (which itself is a quote from <Leviathan>). Softwares such as TensorFlow build on this mentality by allowing users to use graphic user interface (GUI) to compose the architecture of their deep networks using basic building blocks. These basic building blocks are repeatedly used in different kinds of composition, e.g., in parallel, concatenation, or in a sequence. The basic forms are also called architectural primitives or foundational building blocks. A deep net is a spatial manifestation made of these basic forms. TensorFlows made this key-making process remarkably simple.

Why does your deep net model look like what it looks like? It is a question that hovers over everyone who has ever created a deep net model. And it is hard to explain. And the answer often becomes a bit esoteric. 

After all, for deep net models, if a model looks deep, it is a deep model (with both the literal and actual meanings of “deep”). The universal approximation theorem has shown that a neural net model with one hidden layer could characterize all smooth functions. While there is no guarantee that in practice adding more layers will always be better, the theoretical results did imply that is the right direction. 

Well, it is fair to say that this paradox has been there in the very early days of machine learning. Only in recent years, with the advance of deep learning techniques, this paradox caught more attention and became more self-evident. One main difference between machine learning models and non-machine-learning models is that, for machine learning models, the model validation process has been made automatic and data-driven, i.e., given a few candidate models, we no longer need to validate the models by their scientific implication but only check how well they fit the data and how likely they don’t overfit the data. A deep model, just like any other machine learning model, doesn't demand interpretability or validity to be useful. In practice, your task is empirical: put together the architecture of a deep model, and if it obtains superior performance on data, it is a superior model. Now, if a complex problem in the real world is a sophisticated lock, deep learning's real appeal is that we only need to spend effort in guessing at the key (i.e., the architecture of the deep neural network), but not necessarily in understanding the lock. And what makes it more convenient is that we can try every key we made (i.e., by fitting it with data) until the lock is opened. Is this a rational practice? There have been discussions around this topic and readers may be interested to look into this, i.e., see, Hutson, M., “Has artificial intelligence become alchemy?”, Science, 2018. At any rate, not all models are made with a full understanding of the underlying logic. One is tempted to think about what Prof. George Box once said, “all models are wrong, some are useful.”  
