# Mask RCNN for First Robotics images

20181028

## 1. Notebooks for First Robotics

Project is forked from
https://github.com/matterport/Mask_RCNN

Idea is similar to
https://github.com/huuuuusy/Mask-RCNN-Shiny

Best prototype to build recognition pipeline for new class based on pretrained model
https://github.com/matterport/Mask_RCNN/tree/master/samples/balloon

Mask_RCNN technology is described in
https://engineering.matterport.com/splash-of-color-instance-segmentation-with-mask-r-cnn-and-tensorflow-7c761e238b46

- Folder *balloon* is copied as
https://github.com/matterport/Mask_RCNN/tree/master/samples/first_robotic
- Notebooks and balloon.py are adjusted to deal with robotics realm
- ballon.py renamed to robotic.py
- *train** and *val** folders with images: should be placed in Mask_RCNN\datasets\frobotics
- optionally: Mask_RCNN\images_fr folder could be added with a few examples of first robotic images
  It could be used in samples/demo.ipynb (folder ref should be adjusted)

Masks and annotations for training and validaton sets could be prepared in:
http://www.robots.ox.ac.uk/~vgg/software/via/
file:///Users/adenis201/Documents/LabWeek/MaskRCNN/via-2.0.2/via.html
No class selection is necessary inside because we are doing binary classification robot/background for now.

Demonstration of training datasets and validation of training results (from last_model) could be done via notebooks *inspect_robotic_data.ipynb* and *inspect_robotic_model.ipynb*.

Training itself was done from terminal on GPU machine
based on MS COCO model.


## 2. Backend setup

Notebooks are fully functional even on CPU only machines. For example **T3** server on AWS with 16Gb and 20Gb  SSD

Training done on **P3** on AWS 8 GPU with 50Gb SSD. It took around an hour per 3,000 iterations.
Pretrained MS COCO model was used.

**P3** server is on my AWS account, currently stopped, but not terminated

**T3** instance is running on e2etests account - ‘labweebk’. Server stopped, could be restarted at any time

### 2.1 CPU machine backend

- Image: ubuntu/images/hvm-ssd/ubuntu-xenial-16.04-amd64-server-20180912 (ami-059eeca93cf09eebd),
- T3 instance, 
- 16Gb RAM, 20 Gb SSD
- Security group:
port 22 for 'my IP' only,
8888 for all

Code verified to work with
https://hub.docker.com/r/waleedka/modern-deep-learning/

- Install docker on VM

On VM in folder /home/ubuntu

    $git clone https://github.com/matterport/Mask_RCNN.git

Copy train and val datasets into Mask_RCNN/datasets/first_robotic

Optionally:
- copy trained balloon model into Mask_RCNN
- copy datasets unzipped for ballon into Mask_RCNN/datasets/balloon, see details in https://github.com/matterport/Mask_RCNN/tree/master/samples/balloon/README.md

```
docker run -it -p 8888:8888 -p 6006:6006 -v ~/:/host waleedka/modern-deep-learning
```

Inside container

    $cd /host

validate that Mask_RCNN folder is visible.

```
$pip3 install -r requirements.txt
$python3 setup.py install
$jupyter notebook --allow-root
```


Or just run

    docker run -d -p 8888:8888 -v ~/:/host waleedka/modern-deep-learning jupyter notebook --allow-root /host

On Mac port forwarding should be used for localhost access

ssh -i "aws_NVirginia_labweek18.pem" -L 8157:127.0.0.1:8888 ubuntu@ec2-34-207-203-187.compute-1.amazonaws.com

localhost access
http://localhost:8157/notebooks/Mask_RCNN/samples/frobotic/inspect_robotic_data.ipynb

External access
http://labweek.gumby.io:8888/notebooks/Mask_RCNN/samples/frobotic/inspect_robotic_data.ipynb


## 2.2. Training on GPU

- Image Deep Learning AMI (Ubuntu) Version 15.0 (ami-06ddeee5b31a0308f)
- P3 instance on AWS, 8 GPU, 40-50Gb SSD, with

in folder  /home/ubuntu/:

    $git clone https://github.com/matterport/Mask_RCNN.git`


Copy *train* and *val* datasets into Mask_RCNN/datasets/first_robotic

Optionally
- copy trained *balloon* model into Mask_RCNN
- copy datasets unzipped for balloon into Mask_RCNN/datasets/balloon

```
$source activate python3
$jupyter notebook --allow-root
```

Copy token from terminal;
connect to notebook on localhost:8157 using this token

Switch to tensorflow_p36 inside notebook


From terminal

    $source activate tensorflow_p36

Install pycocotool into tensorflow_p36

    $pip install git+https://github.com/waleedka/cocoapi.git#egg=pycocotools&subdirectory=PythonAPI

Or just follow command advise how to install pycocotool in response to attempt to run training

### Run training

     $cd Mask_RCNN\samples\first_robotic`

Around 7GB on SSD is necessary for trained models in 30000 iterations (240Mb per model)

    $python3 robotic.py train --dataset=/host/Mask_RCNN/datasets/fr --weights=coco

To continue after interruption

    $python3 robotic.py train --dataset=/host/Mask_RCNN/datasets/fr --weights=last

Alt:

    python3 robotic.py train --dataset=/host/Mask_RCNN/datasets/fr --weights=imagenet


### Optional. GPU validations

The simple way using tensorflow is:

from tensorflow.python.client 

```
import device_lib
print(device_lib.list_local_devices())
```

from Keras:

```
import backend as K
K.tensorflow_backend._get_available_gpus()
```


Version Validatons

```
python -c "import tensorflow as tf; print(tf.__version__)"
```

Or

python3

```
import tensorflow as tf
print(tf.__version__)
```


