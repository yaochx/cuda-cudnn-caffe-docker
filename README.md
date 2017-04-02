Login
====
normal username
---
username: ulsee  
password: ulsee!

sudoer
---
username: root  
password: **********

# Server
In order to reduce Operation Engineer's work, we choose docker to manage dev&ops.  
On server, CentOS7 and Nvidia 375 diver are installed, by nvidia-docker tool we can  
passthrough GPU device beyond the docker container, this will bring performance and  
ease of usage.

## Server Software

Item | Detail
------- | -------
OS | centos7
CUDA | nvidia-375

## Server Hardware
Item |Detail
------- | -------
CPU | Intel(R) Xeon(R) CPU E5-2683 v3 @ 2.00GHz X 2
GPU | Nvidia Titan X Pascal X 8 
Memory | Samsung DDR4 128GB
SSD | Intel PCIex4 1TB(mounted on /)
HardDisk | 7TB raid5(mounted on /raid5)

## [nvidia-docker](https://github.com/NVIDIA/nvidia-docker/wiki/Why%20NVIDIA%20Docker?)
If you have no experience on Docker, please google some materials about it, also here  
is a [reference](http://www.docker.org.cn/book/) of Docker tutorial.  

Do not worry about the difference between *docker* and *nvidia-docker*, good news is that  
*nvidia-docker* tried to stay as close as possible to the Docker philosophy. So you can use  
*nvidia-docker* as an alternative Docker CLI.

# Docker Deep Learning Images
For the custom requirements, we need to define our own Docker image by Dockerfile. We can  
find a lot of Caffe/MXNet/TensorFLow images online, eg [dockerhub](https://hub.docker.com/explore/) or [github](https://github.com/)

It's an very clear procedure to make a Dockerfile, you can try to explore and find your featured  
Dockerfile lines to add into yours.

## Basic Image
We refer to the [nvidia/cuda](https://hub.docker.com/r/nvidia/cuda/) and [caffe dockerfile](https://hub.docker.com/r/bvlc/caffe/), create some basic images located at
 * /raid5/SourceCode/ulsDeepLearningDocker


## Your Own Image
You can start from the Local images, eg. add this line into Dockerfile's 1st line, then fill your  
own commands, which will reduce a lot of building time, especially, there're lots of downloading tasks.  
Thus, you can skip waiting for downloading.
```
From ulsee/cuda8.0-cudnnv5.1-caffe-centos7
```

## Docker Run Image
### 
To passthrough GPU device in Docker Container, you can run
```
nvidia-docker run --rm ulsee/cuda8.0-cudnn5.1-caffe-tensorflow nvidia-smi 
```
Note: If you want to use network in the container, please add ```--net=host``` in the middle. I.E.
```
nvidia-docker run --rm ulsee/cuda8.0-cudnn5.1-caffe-tensorflow nvidia-smi 
```
Note: ```--rm``` can destroy the backup images generated in the ```run``` period.
### Specify a name for a Container
```
nvidia-docker run --rm -ti --net=host --name ulsee_test ulsee/cuda8.0-cudnn5.1-caffe-tensorflow
```

If you have ```apt-get``` something new, please remember to ```docker commit``` your container to the images. Or all the operation in the image will be cleared.

```
docker commit f14dc ulsee/cuda8.0-cudnn5.1-caffe-tensorflow
```
```f14dc``` is the identical container ID(can be represented by the specific first several letters), ```ulsee/cuda8.0-cudnn5.1-caffe-tensorflow``` is the docker image to be updated, which can be obtained by :

```
docker ps -l
```

For more, you can *nvidia-docker/docker run --help*
Some [tips](http://blog.csdn.net/O1_1O/article/details/52710733) to run docker images in background and foreground again freely

## Mount Local Volume

You can add a data volume to a container using the ```-v``` flag with the ```docker create``` and ```docker run``` command. You can use the ```-v``` multiple times to mount multiple data volumes. 
```
nvidia-docker run --rm -ti -v /Users/<path>:/<container path> ulsee/cuda8.0-cudnn5.1-caffe-tensorflow 
```

Note: They must be AP(Absolute Path). You can also use the VOLUME instruction in a Dockerfile to add one or more new volumes to any container created from that image. For more, please refer to [tips](https://docs.docker.com/engine/tutorials/dockervolumes/#mount-a-host-directory-as-a-data-volume).

## Run Matlab in Docker Container

You can run ```matlab``` by ```mount``` the matlab of the host.

```
nvidia-docker --rm -ti -v /usr/local/MATLAB:/usr/local/MATLAB ulsee/cuda8.0-cudnn5.1-caffe-tensorflow
```
Then, 

```
cd /usr/local/MATLAB/R2016a/bin
```
Then, execute the matlab.

```
./matlab
```
## Policy
### GPU Policy

We totally have 8 Nvidia Titan X Pascal GPUs on our server, to achive the greatest value of our GPUs and let everyone to use the resouce fairly, there are some polices we should observe:

 *  Before select the GPUs to start training process, you should type ```nvidia-smi``` to query GPU devices state, then __choose the idle one__. We should __avoid sharing__ the same GPU for computing with others.
 *  In most cases, you should use __one__ GPU card for training if sufficient.  At most you can use __two__ cards for training, if it still not satisfy your requirement, please conntact Chris to get the authorisation for more cards.
 *  If you have made sure which gpus will be used, you should pass the indices of the GPUs to the environment variable ```NV_GPU``` to __isolate__ with others:
 ```NV_GPU=5 nvidia-docker run -ti nvidia/cuda nvidia-smi```.
 Thus you can only see GPU 5 in your container now.
 
### Name convertion of Docker Image
 
 Your own custom docker images should be named like:
 
 ```${USERNAME}/${ImageName}```,
 eg. ```shikun/abc```
 
 The public basic docker images shared by us should be named like:
 
 ```ulsee/${ImageName}```,
eg. ```ulsee/faceapi:v1.0```

### Memory limit

We should use `-m` flag to specify the maximum amount of memory the container can use, the maximum memory we allow to use is __30GB__, if it doesn't satify your requriement, please conntact Chris for more memory resouces. The command looks like:
```docker run -m 30g ...```
 
 
## Create an account for you
If you have read all the content before this chapter, you can conntact me(Shikun) to create an account for you.Then you will have your own home dir on server, please place your train/test data at this dir, __DO NOT__ put them inside the container or other place. 

Enjoy!!!
 
 

