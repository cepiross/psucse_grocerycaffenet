PennState 2017SP CSE586 Project2: Fine-Tuning CNN <br/> - Jinhang Choi, Jihyun Ryoo
============

This webpage is a comprehensive report of the second project in CSE586 class.

Table of contents :

  * Introduction
  * Background
  * Fine-Tuning CaffeNet
  * Result
  * Summary

Introduction
------------

In this project, we employ Caffe [[1](#Jia14)] as a framework for convolutional neural network (CNN).
Also, as a case study of fine-tuing CNN, we bootstrap CaffeNet [[2](#BAIRCaffeNet)], which is a replication of AlexNet [[3](#Alex12)]. We call the fine-tuned model Grocery-CaffeNet because it exploits CaffeNet's capability to identify some grocery items. Training Grocery-CaffeNet is based on [ImageNet](http://image-net.org/) Dataset.

Background
------------

### Caffe

Caffe is a framework for deep learning which is developed by Berkeley AI Research (BAIR).

Since Caffe supports both CPU and GPU (NVIDIA/AMD/INTEL) based on C++/CUDA/OpenCL, and provides wrapper APIs for python and matlab, users can easily run Caffe on their machines by adjusting confiuration.

It has numerous implementation of general layers used in CNNs; like convolution, response normalization, maximum/average pooling, rectified linear activation (ReLU), sigmoid activation, etc.
Using the layers, our own model can be trained from scratch.
In the meantime, Caffe provides several CNN references such as AlexNet, GoogleNet, and CaffeNet. Therefore, practitioners can benefit from the pre-trained networks to generate byproduct without wasting huge amount of time for duplication.
In this project, we choose CaffeNet.

### AlexNet and CaffeNet

![Layers-of-AlexNet](result/AlexNet.PNG)

Alexnet [[3](#Alex12)] is convolutional neural network which wins the ILSVRC-2012 competition.
The upper figure shows the architecture of CNN.
It consists of five convolutional layers, two local response normalization layers, three max pooling layers and three fully connected layers.

In the paper, AlexNet used dataset from ILSVRC-2010 and ILSVRC-2012.
They could reduce the top-1 and top-5 error rates substantially compared to sparse coding or SIFT + FVs.

CaffeNet is similar to the AlexNet with some differences.
One is that CaffeNet does not use relighting data-augmentation for training.
The other change is the order of layers.
In AlexNet, the pooling layers come after the normalization layers.
But in CaffeNet, the pooling layers come before the normalization layers.

### ImageNet and ILSVRC

[ImageNet](http://image-net.org/) is an image dataset that generally used for image training and inference.
It provides more than 15M labeled high resolution images with about 22K categories including different kinds of animals, vehicels, objects, etc.

ILSVRC (ImageNet Large Scale Visual Recognition Competition) is an annual competition of image classification and detection tasks using ImageNet dataset.
ILSVRC classification (CLS-LOC) dataset contains more than 1.2M images in 1K categories, and ILSVRC detection (DET) dataset contains more than 450K images in 200 categories.
In this project, we used subset of ILSVRC DET data.

Fine-Tuning CaffeNet
------------
For target data, we decided to use ILSVRC-2014 DET Dataset [[4](#ilsvrc14)] since it does not overlap ILSVRC-2012 Dataset used in CaffeNet. To be specific, we choose 21 of 200 DET classes, which might represent some items in a grocery store. The following table describes a breakdown of images in our dataset. Due to annotation issue, all 15,380 images come from DET training dataset, where training/validation dataset are randomly separated in 9:1 manner. Since there are [file size limits](https://help.github.com/articles/what-is-my-disk-quota/) in Github, we do not provide the entire images. Instead, you can recreate them by refering to a full list of \[[train](data/train.txt) | [val](data/val.txt)\] dataset. For the sake of simplicity, we adapted centered 255x255 downscaling to each image. In training, Caffe randomly selects 227x227 RGB input from 255x255 image data to consider manifold variations in object distribution for CaffeNet. There is no data augmentation except annotation arrangement.

|         | Apple   | Artichoke | Bagel   | Banana  | Bell pepper | Burrito | Cucumber  | Fig     | Guacamole |
|:-------:|:-------:|:---------:|:-------:|:-------:|:-----------:|:-------:|:---------:|:-------:|:--------:|
| Train   | 1081    | 809       | 586     | 733     | 633         | 560     | 492       | 475     | 651       |
| Val     | 140     | 95        | 87      | 88      | 68          | 84      | 77        | 53      | 90        |
| Total   | 1221    | 904       | 673     | 821     | 701         | 644     | 569       | 528     | 741       |

|         | Hamburger | Lemon   | Milk can  | Mushroom | Pineapple | Pizza   | Pomegranate | Popsicle | Pretzel |
|:-------:|:---------:|:-------:|:-------:|:--------:|:---------:|:-------:|:-----------:|:--------:|:------:|
| Train   | 555       | 662     | 501       | 671      | 530       | 730     | 822         | 592      | 571     |
| Val     | 72        | 86      | 73        | 75       | 64        | 104     | 79          | 87       | 66      |
| Total   | 627       | 748     | 574       | 746      | 594       | 834     | 901         | 679      | 637     |

|         | Strawberry | Water bottle | Wine bottle | Summary |
|:-------:|:----------:|:------------:|:-----------:|:-------:|
| Train   | 576        | 686          | 733         | 13649   |
| Val     | 83         | 87           | 73          | 1731    |
| Total   | 659        | 773          | 806         | 15380   |

As shown in the following CNN scheme, we directly borrowed layers of CaffeNet except softmax, which is composed of 21 classes. Even though reusing weights and biases in CaffeNet, however, we still need to refine the trained parameters for extracting features in some way. As a result, we did not eliminate learning rate of every layers, but fortify training the last fully connected layer [10x](model/mdl_grocery_caffenet/train_val.prototxt#L364-L371) than other layers. Instead, the initial learning rate decreases to [0.001](model/mdl_grocery_caffenet/solver.prototxt#L4) so that Grocery-CaffeNet can rely on the original CaffeNet's parameters. 

![train-grocery-caffenet](result/train.png)

Since there are 13,649 images in our training dataset, an epoch is roughly 110 iterations in terms of batch size 125. We repeated testing validation dataset for every 110 iterations to understand a trend of top-1 classification accuracy while training continues. Testing itself takes 28 iterations with batch size 62.

<div style="text-align: center;padding: 20px 4px 20px 4px;border-bottom: 1px solid #999;border-top: 1px solid #999;">
  <img src="result/train_loss.png" width="48%" style="max-width: 98%;">
  <img src="result/test_accuracy.png" width="48%" style="max-width: 98%;border-left: 1px solid black;">
  <div style="font-weight: 400;font-size: 14px;color: #575651;text-align: justify;">
    While training Grocery-CaffeNet, we decrease learning rate by one-tenth for every 10K iterations, i.e. rougly 90 GPU epochs. There is no significant difference in both train loss (<b>left</b>) and test accuracy (<b>right</b>) after 30K iterations. Therefore, 30K iterations would be sufficient enough to fine-tune this CNN model.
  </div>  
</div>

A trend of classification accuracy in testing validation dataset indicates that Grocery-CaffeNet introduces fast convergence even with just an epoch due to derivation of parameters from CaffeNet. In the meantime, we can ensure potential of over-fitting issue by monitoring a trend of trian loss penalty. From our experiments, 45,000 iterations are probed to test the extent of available classification in the current setting. Our model may identify 21 grocery item classes in 76.2% accuracy<sup id='rfn1'>[1](#fn1)</sup>.

| name                   | caffemodel                                                                                               | license      | sha1                                     |
|:----------------------:|:--------------------------------------------------------------------------------------------------------:|:------------:|:----------------------------------------:|
| Grocery-CaffeNet model | [caffenet\_train\_iter\_45000.caffemodel](https://drive.google.com/open?id=0B0lt6MbaK2RCZWd0ZklTMmVGbjg) | unrestricted | e43cb843634aae054a2a5bbb813967e0c63b5048 |

Result
------------
Let's analyze the top-1 classification accuracy in more detail. To understand what happend to test validation dataset, we deploy Grocery-CaffeNet on [classification script](script/classify.py) per image. For your information, please refer to the following [log](result/classification.log).

![deploy-grocery-caffenet](result/deploy.png)

In deployment, we compare ground truth to the inference result with a highest confidence score so that can figure out which features Grocery-CaffeNet really learns. The only difference in our CNN scheme is data layer to feed an image at a time (batch size 1).

<div style="text-align: center;padding: 20px 4px 20px 4px;border-bottom: 1px solid #999;border-top: 1px solid #999;">
  <img src="result/result_top1.png" style="max-width: 98%;">
  <div style="font-weight: 400;font-size: 14px;color: #575651;text-align: justify;">
    Since a few of validation images are redundantly tested due to batch size, actual top-1 classification accuracy is slightly lower than the validation trends in training Grocery-CaffeNet. Additionally, this test is based on center-cropped 227x227 images. Since ILSVRC-2014 DET dataset does not locate its detection objects at the center of an image, classification accuracy can be deteriorated. For instance, object occlusion in an image keeps Grocery-CaffeNet from identifying its class preciesly. Nevertheless, mean average precision is near to 70%.
  </div>
</div>

Interestingly, Grocery-CaffeNet does not identify some classes which share similar features with other classes such as shape and color<sup id='rfn2'>[2](#fn2)</sup>. If we fine-tune more deeper network such as GoogLeNet [[5](#Szeg14)] or ResNet [[6](#Kaiming15)], it would recognize the detailed characteristics with respect to each object class.

Summary
------------
We presented fine-tuning CaffeNet model, where it is possible to transfer a capability of the pre-trained model's feature extraction to another model, Grocery-CaffeNet. As described in this case study, dataset collection is very important to take advantage of training CNNs, which can start to converge within just a few GPU epochs. Due to a lot of hidden parameters, however, a CNN model is easily affected by similarities and differences of input data. Even though it can be dealt with more deeper CNNs, there is still a need to consider appropriate matching between dataset with well-defined annotations and a CNN model's capability. That is, unless our goal is to build the most accurate CNN model, we should focus on exploiting well-known CNN model's capability with our own data generation.

------------
<a name='fn1'> </a>
[1.](#rfn1) The top-1 accuracy of the original CaffeNet is 57.4% against ILSVRC-2012 dataset.

<a name='fn2'> </a>
[2.](#rfn2) For instance, red apple versus pomegranate, bagel versus pretzel, and water bottle versus wine bottle have visually similar features. Here are some examples which you can make sure:

<div style="text-align: center;padding: 20px 4px 20px 4px;border-bottom: 1px solid #999;border-top: 1px solid #999;">
  <img src="data/ILSVRC2014_train_00031188.JPEG" style="max-width: 98%;">
  <img src="data/ILSVRC2014_train_00034821.JPEG" style="max-width: 98%;">
  <img src="data/ILSVRC2014_train_00052736.JPEG" style="max-width: 98%;">
  <img src="data/n07695742_7997.JPEG" style="max-width: 98%;">
  <img src="data/ILSVRC2014_train_00005172.JPEG" style="max-width: 98%;">
  <img src="data/ILSVRC2014_train_00005152.JPEG" style="max-width: 98%;">
  <div style="font-weight: 400;font-size: 14px;color: #575651;text-align: justify;">
    The order of ground truth among six images ranging from top-left to bottom-right is apple, pomegranate, bagel, pretzel, water bottle, and wine bottle.
  </div>
</div>

You can find their corresponding inference results here - \[[1](result/classification.log#L454-L458), [2](result/classification.log#L8080-L8084), [3](result/classification.log#L1912-L1916), [4](result/classification.log#L9316-L9320), [5](result/classification.log#L9886-L9890), [6](result/classification.log#L10408-L10412)\]

References
------------
<a name='Jia14'> </a>
[1] [Yangqing Jia, Evan Shelhamer, Jeff Donahue, Sergey Karayev, Jonathan Long, Ross Girshick, Sergio Guadarrama, and Grevor Darrell, "Caffe: Convolutional Architecture for Fast Feature Embedding", arXiv preprint arXiv:1408.5093, 2014.](http://caffe.berkeleyvision.org/ "Caffe Homepage")

<a name='BAIRCaffeNet'> </a>
[2] [Berkeley AI Research (BAIR) CaffeNet Model](https://github.com/BVLC/caffe/tree/master/models/bvlc_reference_caffenet)

<a name='Alex12'> </a>
[3] [Alex Krizhevsky, Ilya Sutskever, and Geoffrey E. Hinton, "ImageNet Classification with Deep Convolutional Neural Networks", NIPS, 2012.](https://papers.nips.cc/paper/4824-imagenet-classification-with-deep-convolutional-neural-networks "AlexNet")

<a name='ilsvrc14'> </a>
[4] [Imagenet Large Scale Visual Recognition Challenge 2014 (ILSVRC-2014) Dataset](http://image-net.org/challenges/LSVRC/2014/index#data)

<a name='Szeg14'> </a>
[5] [Christian Szegedy, Wei Liu, Yangqing Jia, Pierre Sermanet, Scott Reed, Dragomir Anguelov, Dumitru Erhan, Vincent Vanhoucke, and Andrew Rabinovich, "Going Deeper with Convolutions", arXiv preprint arXiv:1409.4842, 2014.](https://github.com/BVLC/caffe/tree/master/models/bvlc_googlenet)

<a name='Kaiming15'> </a>
[6] [Kaiming He, Xiangyu Zhang, Shaoqing Ren, and Jian Sun, "Deep Residual Learning for Image Recognition", arXiv preprint arXiv:1512.03385, 2015.](https://github.com/KaimingHe/deep-residual-networks)
