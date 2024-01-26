##SVM and SVR ##
###SVM###
A Support Vector Machine (SVM) is a supervised machine learning algorithm that can be employed for both classification and regression purposes. Support vector machines can be applied to both classification and regression. SVMs are based on the idea of finding a hyperplane that best divides a dataset into two classes, as shown in the image below: ![](http://dni-institute.in/blogs/wp-content/uploads/2015/09/SVM-Planes.png)
####Support Vectors####
Support vectors are the data points nearest to the hyperplane, the points of a data set that, if removed, would alter the position of the dividing hyperplane.
####How SVM finds the right hyperplane####
The distance between the hyperplane and the nearest data point from either set is known as the margin. The goal is to choose a hyperplane with the greatest possible margin between the hyperplane and any point within the training set, giving a greater chance of new data being classified correctly.
In order to classify a dataset like the one above it's necessary to move away from a 2d view of the data to a 3d view. Explaining this is easiest with another simplified example. Imagine that our two sets of colored balls above are sitting on a sheet and this sheet is lifted suddenly, launching the balls into the air. While the balls are up in the air, you use the sheet to separate them. This 'lifting' of the balls represents the mapping of data into a higher dimension. This is known as kernelling. 

![](https://i.stack.imgur.com/1gvce.png)

Because we are now in three dimensions, our hyperplane can no longer be a line. It must now be a plane as shown in the example above. The idea is that the data will continue to be mapped into higher and higher dimensions until a hyperplane can be formed to segregate it.

###SVR###
Support vector machines can be applied to both classification and regression. When it is applied to a regression problem it is just termed as support vector regression. When you have a linearly separable set of points of two different classes, the objective of a SVM is to find where that particular hyperplane in which separates these two classes with minimum error while also making sure that the perpendicular distance between the two closes points from either of these two classes is maximized. That's how the hyperplane is determined. You get something similar to following picture.

![](https://staesthetic.files.wordpress.com/2014/02/svm.png?w=1060)

SVR uses the same idea but it finds a hyperplane that has points on the either side of it but I want to ensure that the distance between these points and the line should not farther than epsilon. It is equivalent to hand-draw a line somewhere in the middle of the set of points making sure the line is as close to them as possible.

![](https://qph.ec.quoracdn.net/main-qimg-1b899fcfd6dbee47d238b9b661aa2f05)

Therefore SVM optimizes:

![](https://qph.ec.quoracdn.net/main-qimg-847bed06a8f44b045641507c306a7404)

while SVR optimizes:

![](https://qph.ec.quoracdn.net/main-qimg-408b6f8a3af1a9440c2e691305fbce68)


