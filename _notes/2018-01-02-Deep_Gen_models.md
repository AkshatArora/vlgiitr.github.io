---
layout: single
title: "Faster R-CNN: Towards Real-Time Object Detection with Region Proposal Networks"
category: Computer Vision
description: "Compiled by Prakash Pandey"
mathjax: true

---


<!-- # Faster R-CNN: Towards Real-Time Object Detection with Region Proposal Networks -->

Ross Girshick, ArXiv 2015
  

## Summary

This paper proposes a Fast Region-based Convolutional Neural Network method \[ Fast R-CNN \] for object detection tasks. Takes less training and testing time and is more accurate at detection than previous method of R-CNN and SPPnet. Fast R-CNN uses very deep VGG16 network which jointly learns to classify region proposals and simultaneously refine their spatial location using bounding box regressors.

  

+   **Drawbacks of R-CNN & SPPnet**
    

	-   In R-CNN, training is a multi-stage pipeline in which network first generates around 2000 category independent region proposals for the input image. All these region proposals are fed to a CNN which outputs a fixed length [4096] feature vector and then it uses SVM classifier on top of it to classify the object. Finally, bounding box regressors are used for refining the regions around the object of interest.
    
	-   Clearly the above mentioned process is time consuming. The same process is repeated at test time which takes quite a lot of time for processing a single image [47s/image on a GPU].
    
	-   Spatial Pyramid Pooling [SPPnet] was introduced which can share the parameter between different region proposals. SPP also uses Selective Search to generate around 2000 region proposals per image which are then mapped to convolutional feature map.
    
	-   Using Spatial Pyramid Pooling of different bin size on last feature map and concatenating all the outputs, each of the region proposal is converted to a fixed size feature vector which is then fed to SVM classifier.
    
	-   The training of SPPnet is also a multi-stage pipeline and the process of fine-tuning is not very helpful in SPPnet.
    

+   **Fast R-CNN architecture**
    

	-   Here, input is an entire image and a list of Region of Interests [RoIs] in that image. First input is fed to a standard CNN which after several convBlocks, gives a feature map. Now, for each of the region proposal, a RoI pooling layer extracts a fixed length feature vector.
    
	-   This feature vector is fed to a sequence of fully connected layers and the final output from these layers is divided into two parts. One part gives the softmax probabilities per RoI over all the classes and another part gives 4 real numbers which store the information about the bounding boxes. It was shown by the authors that softmax outperforms SVM classifier.
    
	-   Each RoI is pooled to a fixed spatial extent using max pooling. This fix shape is independent of RoI shape which are of different sizes. Like in standard CNN, pooling is applied to each channel of the feature map independently.
    
	-   RoI is basically a special case of SPPnet where we are dealing with only one pyramid level.
    
	-   While training mini-batches are sampled hierarchically where first N images are sampled from the training set and then R/N RoIs are sampled from each image.
    
	-   A multi-task loss is used to jointly train the network for classification as well as bounding box regression over all RoIs in which the first loss is the log loss and the second one is L1 loss. It turns out that Multi-task training performs better than piecewise training.
    
	-   To reduce the computation, we can compress the fully connected layers using truncated SVD. In this, the weight matrix of shape u x v can be approximated by selecting the first t vectors of left and right singular matrix and the first t singular values obtained using SVD [W = UΣtVT]. Then the first layer is connected to weight matrix ΣtVT and  these t values are connected to weight matrix U. This simple compression results into faster detection with a small accuracy loss.
    
	-   We can initialize Fast R-CNN with pre-trained networks by replacing last pooling layer with RoI pooling and output layer by 2 separate layers for softmax layer and bounding box regressors.
    
	-   In deep networks, fine-tuning Conv layers in addition to fully connected layers is also important. All layers starting from conv3_1 up to the last were fine-tuned.
    

  
  

+   **Results**
    

	-   Fast R-CNN achieves the top result on VOC12 with a mAP of 65.7% which is more than R-CNN and SPPnet.
    
	-   Fast R-CNN trains the very deep VGG16 network 9x faster than R-CNN and is 213x faster at test-time. Compared to SPPnet, Fast R-CNN trains VGG16 3x faster and 10x faster at test time.
    

  
  
  

## Strengths

-   It has a higher detection quality than R-CNN & SPPnet.
    
-   Training is single-stage using a multi-task loss.
    
-   Fine tuning can update all layers of the network
    
-   It is much faster than R-CNN & SPPnet both at training as well as testing.
    
-   Memory requirement is much less as it doesn’t require cache features.
    

  
  

## Weaknesses/Notes

-   We saw that Fast R-CNN is much better than R-CNN & SPPnet both in terms of performance and speed. Still, there are few problems.
    
-   Selective search which is used for region proposal tasks is a slow process and is the bottleneck of the overall process especially at test time.
    
-   Region proposals which actually depend on the feature map can directly be generated from the last feature map instead of using a separate selective search algorithm.
    
-   Faster R-CNN solves the above problem by using a Region Proposal Network [RPN] on top of the last feature map to generate regions.
    

  

## Implementation

[https://github.com/rbgirshick/fast-rcnn](https://github.com/rbgirshick/fast-rcnn)

  

## References

[https://arxiv.org/pdf/1504.08083.pdf](https://arxiv.org/pdf/1504.08083.pdf)