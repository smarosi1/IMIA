# Imitative Membership Inference Attack

This repository contains an implementation of the [Imitative Membership Inference Attack](https://arxiv.org/abs/2509.06796). This attack aims to imitate the behavior of a machine learning model and determine if specific pieces of data were used to train that model. The attack is written in an IPython notebook, meaning users can run each part of the attack separately, go at their own pace, and modify chunks of code individually. 

# Attack Structure

## Training Target Model

First, the model to be attacked is trained on a random sample of the training dataset. For this project, I decided to use the CIFAR-10 dataset, as this dataset was used in the original IMIA paper. These dataset contains 32x32 pixel images from 10 classes of vehicles and animals. 

![CIFAR-10](/images/cifar10-3.0.2.png)

## Subset Creation

Once the target model is trained, N subsets are created using the original dataset. Each query from the dataset is placed in N/2 subsets. This means half of the subsets will contain any given query, and the other half will not contain that query, which is crucial for the inference phase of the attack (described later).


## Imitative Training

After the N subsets have been created, each one is used to train one of N imitative "shadow" models. First, the models are trained using a normal loss function to bring their accuracies to a sufficient level. Once each model has been "warmed up", they are then trained using imitative loss. Rather than measuring how accurate the model is, imitative loss compares how far the outputs of the shadow model are from the outputs of the target model. This means that the shadow models are trained to directly mimic the target model's behavior for any given query.

![Imitative Training](/images/imitative_training.png)

## Inference

Lastly, I iterate through all the queries in the original dataset and try to determine if they were present in the target model's training dataset. Since every query in the original dataset is present in half of the subsets, and since each subset was used to train one imitative model, half of the imitative models were trained with that query and half were trained without. I then compare three quantities: How confident the target model's outputs are, how confident the "in" models' outputs are on average, and how confident the "out" models' outputs are on average. If the target model's confidence is more like the the "in" models, then the prediction is that the query was in the dataset used to train the model. Otherwise, the prediction is that the query was not used to train the model.