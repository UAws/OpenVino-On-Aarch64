# OpenVino-on-Aarch64

### There is a project to build OpenVino on Ubuntu 20.04 LTS in Aarch64 Architecture.

## Infrastructure Flow Diagram

![OpenVino-on-Aarch64-Page-2.drawio (2)](https://minio.llycloud.com/image/uPic/image-20220114pUqDFC.png)

1. Commit the Infrastructure Code to Gitalb Instance in order to build OpenVino on Aarch64.
   The Dockerfile for build OenVino On Aarch64 is modified and captured from [OpenVino Docker-CI official Repository](https://github.com/openvinotoolkit/docker_ci/tree/master/dockerfiles/ubuntu20/build_custom)

2. Build OpenVino on Aarch64 Image on Cloud VM, the Oracle Cloud has been used. The VM is leveraged by Oracle Cloud Always Free Tier.
   The VM have 4 A1 CPU (Arm), 24 GB memory, 200 GB Disk.
   [https://www.oracle.com/au/cloud/free/

   ](https://www.oracle.com/au/cloud/free/)The Runner configures with docker executor and use DIND (docker in docker typology) to build image.
   [https://docs.gitlab.com/runner/executors/docker.html ](https://docs.gitlab.com/runner/executors/docker.html)

3. Push the OpenVino on Aarch64 image to the self-hosted [gitlab container registry](https://docs.gitlab.com/ee/user/packages/container_registry/).

4. Pull the image from gitlab container registry.
   As the project in state of internal, only authorized user could pull this imnage

5. Run the image as a container and perform the Openvino object detection operations

## Getting Start

### Installation

#### STEP 1 : Install OS

##### Download and Install Ubuntu 20.04.2.0 LTS `ARM64`

[HERE](https://cdimage.ubuntu.com/releases/20.04/release/ubuntu-20.04.3-live-server-arm64.iso) to donwload Ubuntu 20.04.2.0 LTS ISO image ` ARM64`

##### Install Ubuntu 20.04

[HERE](https://phoenixnap.com/kb/install-ubuntu-20-04) is a guide for Ubuntu installation.

#### STEP 2 : Install Docker


```sh
sudo apt update
sudo apt-get update
sudo apt-get install docker.io
```

Inorder to run as non-root user, you have to run commands below.

```sh
sudo usermod -aG docker $USER
```

If you still have some problem about permission. Try with

```sh
sudo chmod 777 /var/run/docker.sock
```

#### STEP 3 : Pull Docker image

You need to make sure you have the access to the self-hosted gitlab container registry. Then Login to the registry.

https://gitlab.dev.vmv.re/AkideLiu/openvino-on-aarch64/container_registry

```shell
sudo docker login gitlab-registry.dev.vmv.re
```

Pull the latest version image

```sh
sudo docker pull gitlab-registry.dev.vmv.re/akideliu/openvino-on-aarch64:latest
```

#### STEP 4 : Run Docker image

```sh
sudo docker run -it --privileged -v /dev/video0:/dev/video0  -v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY=$DISPLAY --device-cgroup-rule='c 189:* rmw' -v /dev/bus/usb:/dev/bus/usb  -d --net=host gitlab-registry.dev.vmv.re/akideliu/openvino-on-aarch64
```

- `-it` : For interactive processes (like a shell), you must use `-i -t` together in order to allocate a tty for the container process. `-i -t` is often written `-it` as you’ll see in later examples
- `--privileged` :  Give extended privileges to this container, in order to access host devices, camera and NCS2
- `-v /dev/video0:/dev/video0` : mapping the hosting video0 camera device to container
- `-v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY=$DISPLAY` : x11 forwarding application docker support. This setting will allow to display graphic user interface to view the real time object detection.
- `-device-cgroup-rule='c 189:* rmw'` : NCS2 rules
- `-v /dev/bus/usb:/dev/bus/usb` mapping the hosting usb device (NCS2) to container
- `-d` run on daemon
- `--net=host` support x11 for docker

reference docs:

https://docs.docker.com/engine/reference/run/

https://hub.docker.com/r/openvino/ubuntu20_dev

https://gist.github.com/sorny/969fe55d85c9b0035b0109a31cbcb088

### STEP 5 : Run Pre-built Sample application 

Run `sudo docker ps` to retrieve the container ID, in following example, the container ID is `8060f370a6cb`

```sh
(openvino) ubuntu@pi-home-01:~$ sudo docker ps
CONTAINER ID   IMAGE                                                     COMMAND        CREATED       STATUS       PORTS                                                                                          NAMES
8060f370a6cb   gitlab-registry.dev.vmv.re/akideliu/openvino-on-aarch64   "/bin/bash"    6 hours ago   Up 6 hours                                                                                                    crazy_satoshi
```

Step into the OpenVino container

```sh
docker exec -it CONTAINER_ID /bin/bash
```

Run the pre-build Open Model Zoo demo

```sh
cd ~/omz_demos_build/aarch64/Release/

./security_barrier_camera_demo -d MYRIAD -d_va MYRIAD -d_lpr MYRIAD -nc 1 -m /opt/intel/openvino/demos/security_barrier_camera_demo/cpp/intel/vehicle-license-plate-detection-barrier-0106/FP16/vehicle-license-plate-detection-barrier-0106.xml -m_lpr /opt/intel/openvino/demos/security_barrier_camera_demo/cpp/intel/license-plate-recognition-barrier-0001/FP16/license-plate-recognition-barrier-0001.xml -m_va /opt/intel/openvino/demos/security_barrier_camera_demo/cpp/intel/vehicle-attributes-recognition-barrier-0039/FP16/vehicle-attributes-recognition-barrier-0039.xml
```

https://github.com/openvinotoolkit/open_model_zoo/tree/master/demos/security_barrier_camera_demo/cpp



![image-20220114014730915](https://minio.llycloud.com/image/uPic/image-2022011477ir89.png)

## Acknowledgement

1. Oracle Cloud Always Free Tier
2. Gitlab Self Host Chart Edition
3. OpenVino Docker CI
