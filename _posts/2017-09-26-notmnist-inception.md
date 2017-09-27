---
layout: post
title: notMNIST Inception
subtitle: Classifying letters with an inception model
---

The [notMNIST](https://yaroslavvb.blogspot.co.uk/2011/09/notmnist-dataset.html) data set is great one for playing around with relatively simple neural network models. It's a collection of letters (a through j) from a huge array of different fonts, all formatted to the same 28x28 picture size. From our perspective, this is a 10-label classification problem with hundreds of thousands of samples to train from and some really funky visual features to try to extract and generalize across:

[![source_notmnist](/Cogneuro_helpers/img/source_notmnist.png)](/Cogneuro_helpers/img/source_notmnist.png)
<sub><a href='https://yaroslavvb.blogspot.co.uk/2011/09/notmnist-dataset.html'>source</a>

It's a wonderfully dirty data set with a healthy dose of mislabeled and/or bizarre trials.

[![sample_notmnist](/Cogneuro_helpers/img/sample_notmnist.png)](/Cogneuro_helpers/img/sample_notmnist.png)

This random sample of 100 letters appears to include an alien, a pear, a jalepeno, a star, and a strip of film?

Fortunately, the test set is clean, so we'll just need to build a model that is robust to these weird training samples.
