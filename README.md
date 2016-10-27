# TensorZoom-TF

This is a Tensorflow project of super-resolution that implemented the paper [Photo-Realistic Single Image Super-Resolution Using a Generative Adversarial Network](https://arxiv.org/abs/1609.04802). The TensorZoom network can scale up an image by 4 times its width and height (16 times the area) with significantly better quality comparing with bilinear scaling. The training process is based on adversarial training that train a discriminative network and a generative network alternatively in order to achieve a better visual result.



##Android Demo
This project is linked to an Android demo. The demo can be found in here: 

<a href='https://play.google.com/store/apps/details?id=uk.tensorzoom&utm_source=global_co&utm_medium=prtnr&utm_content=Mar2515&utm_campaign=PartBadge&pcampaignid=MKT-Other-global-all-co-prtnr-py-PartBadge-Mar2515-1'><img alt='Get it on Google Play' src='https://play.google.com/intl/en_gb/badges/images/generic/en_badge_web_generic.png' width='323'/></a>

This demo has already included the pre-trained GraphDef files:
- [tz6-s-stitch.pb](https://github.com/machrisaa/tensorzoom/blob/master/results/tz6-s-stitch/tz6-s-stitch-gen.pb) for small image and level 3 zooming
- [tz6-s-stitch-sblur-lowtv-gen.pb](https://github.com/machrisaa/tensorzoom/blob/master/results/tz6-s-stitch-sblur-lowtv/tz6-s-stitch-sblur-lowtv-gen.pb) for large image takne from camera

The demo also support to use custom GraphDef file. Open setting -> select "Use custom Tensorflow GraphDef for Tensorzoom" -> Select GraphDef file to choose a .pb file on the device.

> Hint: Depending on your Android version, you may need to click "..." icon on the menu and select "Show SD card" in order to find the files.

See the "Export GraphDef" section below to see how to create a .pb file for the APP and the detail of all the pre-trained Graph in this project.



##Modifications
- This implementation is mainly focus on mobile usage. Therefore only 6 batch normalistion layers are used.
- In general, mobile device can only afford to calculate a image with at most 120x120 size. So a large image is sliced into smaller tiles to render. Then the tiles are stitched together to form a large image. In order to improve the quality of the stitched result, we added function to slice the input image into 16 smaller images and concat them together to form a batch for training. This method can greatly improve the quality when multiple scaled images are stitched together.
- A additional deblur training is added. The input image is blurred using ```tf.nn.depthwise_conv2d```. The quality is better for the image taken from mobile camera but worse than normal for small images or thumbnails.



##Samples
Here are some sample result generated by this algorithm.(click to download the full size to see the result)
<table>
  <tr>
    <td><img src="https://github.com/machrisaa/tensorzoom/blob/master/analysis/cat_h.jpg?raw=true"/></td>
    <td><img src="https://github.com/machrisaa/tensorzoom/blob/master/analysis/cat_h_tz6-s-stitch-gen.jpg?raw=true"/></td>
  </tr>
</table>
<table>
  <tr>
    <td><img src="https://github.com/machrisaa/tensorzoom/blob/master/analysis/london2.jpg?raw=true"/></td>
    <td><img src="https://github.com/machrisaa/tensorzoom/blob/master/analysis/london2_tz6-s-stitch-sblur-notv-gen.jpg?raw=true"/></td>
  </tr>
</table>



##How to use
### Generate Scale-Up Image
To test rendering an image is simple and do not require extra files and project.
> An example is provided in ```net_analysis.py```.

### Train the Network
To train the network need to download additional files and change the settings in ```trainer.py```:
```
# https://github.com/machrisaa/tensorflow-vgg
VGG_NPY_PATH = '../tensoflow_vgg/vgg19.npy'

# http://msvocds.blob.core.windows.net/coco2014/train2014.zip
COCO2014_PATH = '../../datasets/coco2014/train2014'
```
The training need to use a the result of the ```conv2_2``` layer from a pre-trained VGG19 network - [Tensorflow-VGG](https://github.com/machrisaa/tensorflow-vgg). The dataset we are using is the Microsoft Coco2014 data set - [2014 Training images [80K/13GB] data set](http://mscoco.org/dataset/#download).
Download and update the path in ```trainer.py``` before start running this file.
The training will save per 100 iterations in 2 formats:
- NPY file (dis and gen): easlier to be used by other python applications
- standard Tensorflow save file: easiler to resume training, e.g. globle steps
Both the NPY files and the Tensorflow save can be used to resume the training. Tensorboard logging will also be added per 100 iterations.

### Export GraphDef (for mobile)
Exporting a network into a GraphDef file require to select a NPY file and call the ```save_graph``` method:
```
net = TensorZoomNet(npy_path='./results/tz6-s-stitch-sblur-lowtv/tz6-s-stitch-sblur-lowtv-gen.npy', trainable=False)
net.save_graph(logdir='./', name='export.pb')
```

> Sample can be found in [here](https://github.com/machrisaa/tensorzoom/blob/master/tensorzoom_net.py#L299)

This repository provide several pre-trained NPY results that can be used to export the GraphDef:
- [tz6-s-nostitch-gen.npy](https://github.com/machrisaa/tensorzoom/blob/master/results/tz6-s-nostitch/tz6-s-nostitch-gen.npy): The plain vanilla training without stitch and deblur method. Bad for mobile sliced images but good for rendering a whole image on desktop.
- [tz6-s-stitch-gen.npy](https://github.com/machrisaa/tensorzoom/blob/master/results/tz6-s-stitch/tz6-s-stitch-gen.npy): A training that used the stitch method, no deblur. Bad for multi mega pixels photo taken by mobile device.
- [tz6-s-stitch-sblur-gen.npy](https://github.com/machrisaa/tensorzoom/blob/master/results/tz6-s-stitch-sblur/tz6-s-stitch-sblur-gen.npy): Trained from the stitch result with additional deblur training method. Better quality for large image takne from mobile camera.
- [tz6-s-stitch-sblur-lowtv-gen.npy](https://github.com/machrisaa/tensorzoom/blob/master/results/tz6-s-stitch-sblur-lowtv/tz6-s-stitch-sblur-lowtv-gen.npy): Another version with deblur version. Try to remove total variant cost to train the network.

> Due to the file size limit, the NPY files for the Discriminative Network is not included.

