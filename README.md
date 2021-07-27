# MNIST with the minimum number of labels

The objective of this exercise was to develop an algorithm that could classify the MNIST dataset by using the minimum number of labels from the training dataset.
## Baseline

In order to define a baseline, I trained a CNN with a subset of the training dataset. I started with the entire dataset and I iteratively reduced the number of samples until it became impossible to obtain an accuracy above 90%. 
The samples were selected randomically and multiple training sessions were performed for each dimension of the dataset. It was interesting to see that the particular CNN I used was quite effective and it consistently managed to obtain more than 90% validation accuracy until I decreased the number of samples to less than 150. 
It should be mentioned though that, when the number of samples was smaller than 500, in order to reach an accuracy > 90% on the validation dataset, I had to train the model for more than 200 epochs.
The objective of this task was therefore to find a subset of the original training dataset with less than 150 examples that could be used to bring the model’s accuracy above 90%.

## Unsupervised Learning + Supervised learning

Initially I used kmeans to divide the training dataset into clusters with visually similar features. The idea was that by providing one example from each cluster, the model would have seen a distribution diverse enough to understand how to differentiate all the different digits. 

I selected the elements closest to the centroids of each cluster and labeled them in order to use them for the training of the CNN.

At the beginning of this experiment, I selected the parameter k of kmeans equal to the number of digits (10). However 10 clusters do not allow the algorithm to identify the different features well enough. I therefore increased the number of clusters to 32 and tried to train a CNN with the data points closest to the centroids of each cluster.
Unfortunately this small amount of samples was not enough to reach 90% of accuracy.

I tried therefore to add to the set of labelled data also the elements in each cluster that had the larger euclidean distance from the center of the cluster. This was inspired by the idea that the model needed to see some rare, out of the distribution examples in order to learn how to generalize. Unfortunately though this didn’t help to reach the objective.

One successful approach was to increase the number of kmeans clusters to 120. I used the labels of the 120 samples closest to the centroid of each cluster to train the CNN and managed to obtain > 90% accuracy.
The result was obtained by training the CNN for 250 epochs.  

Another approach that worked was to label each sample of the training dataset with the label of the center of the cluster it belonged to. This allowed the model to reach the 90% validation accuracy in just 2 epochs. It showed to be a bit fragile though, as the validation loss quickly increased after a few epochs, as soon as the model started to learn the “dirty” labels obtained with this method. In this case the number of examples used to train were all the examples of the dataset, but the number of labels required were only 120. 

## Active learning - Monte Carlo Dropout Sampling

This method is the one that I liked the most and it’s the one that returned the best results.

I used Monte Carlo Dropout Sampling to select the samples for which the model was least certain as to what the correct output should have been and I labelled them. The intuition was that those samples were the most interesting, informative, instructive for the network and the ones that could help it learn faster.

I trained the CNN with an initial number of labelled data (70 samples) selected randomically. By training the model with these first labelled samples for 210 epochs, I obtained an accuracy of 78%. 
Afterwards I iteratively used the resulting model to find the 10 hardest examples across the unlabelled samples of the training dataset. 
In order to find the hardest examples, I predicted the class of each unlabelled sample with dropout enabled 10 times. The 10 samples that showed on average the highest entropy were selected, labelled and used to continue the training of the model.
By doing so, I was able to reach 92% of validation accuracy with 100 samples.

The intuition is that the model found the response to its most complicated doubts every time that the hard examples were added to the list of the labelled samples.

## Conclusions
I liked doing this exercise. Playing around with MNIST is always fun as it gives the opportunity to quickly experiment ideas and better understand some behaviours that we don’t always have time to appreciate. 
For example I was particularly impressed by the behaviour of the CNN I used. Given the unusually small dimension of the training dataset, I had enough patience to wait for 250 epochs. I observed that even though the model overfitted in the initial phase of the training, after 70/80 epochs, when the training accuracy was already equal to 100%, and the training loss was still slowly decreasing, the validation accuracy started to increase again and it kept increasing until 90%. 
The accuracy function and the loss function are related but are not the same thing and even if the training accuracy is equal to 100%, the weights can still find their way out of their “overfitted location”. This was the first time I observed this behaviour and I was quite happy. Techniques like early stopping assume that overfitting is irreversible but it is not. In simple cases like MNIST, a high degree of dropout and a lot of epochs can allow the backpropagation to recover from an unfortunate local minima.
 



