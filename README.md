# Hermes App

Wildfires have been ever-increasing, devouring our planet earth, rendering our planet worse day by day. With early detection and mitigation, it is possible to reduce the damage caused by wildfires.

![Wildfires](resources/wildfires.png)

To better enable our front-line workers, here is Hermes, an AI-powered Computer Vision application that helps in early detection of Wildfires using reconnaissance drones.

## Citations

* [AlexeyAB/darknet](https://github.com/AlexeyAB/darknet)
* [damiafuentes/DJITelloPy](https://github.com/damiafuentes/DJITelloPy)
* [moses-palmer/pynput](https://github.com/moses-palmer/pynput)

## Index

1. [Introduction](#Introduction)
2. [Deepstream Setup](#Deepstream-Setup)
    1. [Install System Dependencies](#Install-System-Dependencies)
    2. [Install Deepstream](#Install-Deepstream)
3. [Ryze Tello Setup](#Ryze-Tello-Setup)
    1. [Install pip packages](#Installing-pip-packages)
    2. [Redis](#Redis)
    3. [Connect Tello](#Connecting-the-Tello)
4. [Running the Application](#Running-the-Application)
    1. [Clone the repository](#Cloning-the-repository)
    2. [Run with different input sources](#Run-with-different-input-sources)
    3. [Run with the drone](#Run-with-the-drone)

## Introduction

Hermes Application consists of two parts. An Intelligent Video Analytics Pipeline powered by Deepstream and NVIDIA Jetson Xavier NX and a reconnaissance drone, for which I have used a Ryze Tello.

![Tello and Jetson NX](resources/tellonx.jpg)

This project is a proof-of-concept, trying to show that surveillance and mapping of wildfires can be done with a drone and an onboard Jetson platform.

## Deepstream Setup

This post assumes you have a fully functional Jetson device. If not, you can refer the documentation [here](https://docs.nvidia.com/jetson/jetpack/install-jetpack/index.html).

### 1. Install System Dependencies

```sh
sudo apt install \
libssl1.0.0 \
libgstreamer1.0-0 \
gstreamer1.0-tools \
gstreamer1.0-plugins-good \
gstreamer1.0-plugins-bad \
gstreamer1.0-plugins-ugly \
gstreamer1.0-libav \
libgstrtspserver-1.0-0 \
libjansson4=2.11-1
```

### 2. Install Deepstream

#### DeepStream 5.1

Download the DeepStream 5.1 Jetson Debian package `deepstream-5.1_5.1.0-1_arm64.deb`, to the Jetson device from [here](https://developer.nvidia.com/deepstream-getting-started). Then enter the command:

```sh
sudo apt install deepstream-5.1_5.1.0-1_arm64.deb
```

#### DeepStream 6.0

### Install Dependencies

Enter the following commands to install the prerequisite packages:

```
$ sudo apt install \
libssl1.0.0 \
libgstreamer1.0-0 \
gstreamer1.0-tools \
gstreamer1.0-plugins-good \
gstreamer1.0-plugins-bad \
gstreamer1.0-plugins-ugly \
gstreamer1.0-libav \
libgstrtspserver-1.0-0 \
libjansson4=2.11-1
```

### Install librdkafka (to enable Kafka protocol adaptor for message broker)

Clone the librdkafka repository from GitHub:

```
$ git clone https://github.com/edenhill/librdkafka.git
```

### Configure and build the library:

```
$ cd librdkafka
$ git reset --hard 7101c2310341ab3f4675fc565f64f0967e135a6a
./configure
$ make
$ sudo make install
```

### Copy the generated libraries to the deepstream directory:

```
$ sudo mkdir -p /opt/nvidia/deepstream/deepstream-6.0/lib
$ sudo cp /usr/local/lib/librdkafka* /opt/nvidia/deepstream/deepstream-6.0/lib
```

### Install latest NVIDIA BSP packages

Open the apt source configuration file in a text editor, for example:

```
$ sudo vi /etc/apt/sources.list.d/nvidia-l4t-apt-source.list
```

Change the repository name and download URL in the deb commands shown below:

```
deb https://repo.download.nvidia.com/jetson/common r32.6 main
deb https://repo.download.nvidia.com/jetson/<platform> r32.6 main
```

<platform> identifies the platform’s processor:
t186 for Jetson TX2 series
t194 for Jetson AGX Xavier series or Jetson Xavier NX
t210 for Jetson Nano or Jetson TX1

For example, if your platform is Jetson Xavier NX:

```
deb https://repo.download.nvidia.com/jetson/common r32.6 main
deb https://repo.download.nvidia.com/jetson/t194 r32.6 main
```
    
Save and close the source configuration file.

Enter the commands:

```
$ sudo apt update
```

    Install latest NVIDIA V4L2 Gstreamer Plugin using the following command:

```
    $ sudo apt install --reinstall nvidia-l4t-gstreamer
```
    
If apt prompts you to choose a configuration file, reply Y for yes (to use the NVIDIA updated version of the file).

### Install latest L4T MM and L4T Core packages using following commands:

```
    $ sudo apt install --reinstall nvidia-l4t-multimedia
$ sudo apt install --reinstall nvidia-l4t-core
```
## Note

You must update the NVIDIA V4L2 GStreamer plugin after flashing Jetson OS from SDK Manager.

## Install the DeepStream SDK

Method 1: Using SDK Manager

Select DeepStreamSDK from the Additional SDKs section along with JP 4.6 software components for installation.

Method 2: Using the DeepStream tar package: https://developer.nvidia.com/deepstream_sdk_v6.0.0_jetsontbz2

Download the DeepStream 6.0 Jetson tar package deepstream_sdk_v6.0.0_jetson.tbz2 to the Jetson device.

Enter the following commands to extract and install the DeepStream SDK:

```
$  sudo tar -xvf deepstream_sdk_v6.0.0_jetson.tbz2 -C /
$ cd /opt/nvidia/deepstream/deepstream-6.0
$ sudo ./install.sh
$ sudo ldconfig
```
    
Method 3: Using the DeepStream Debian package: https://developer.nvidia.com/deepstream-6.0_6.0.0-1_arm64deb

Download the DeepStream 6.0 Jetson Debian package deepstream-6.0_6.0.0-1_arm64.deb to the Jetson device. Enter the following command:

```
$ sudo apt-get install ./deepstream-6.0_6.0.0-1_arm64.deb
```

## Ryze Tello Setup

### 1. Installing pip packages

First, we need to install python dependencies. Make sure you have a working build of python3.7/3.8

```sh
sudo apt install python3-dev python3-pip
```

The dependencies needed are the following:

```sh
djitellopy==1.5
evdev==1.3.0
imutils==0.5.3
numpy==1.19.4
opencv-python==4.4.0.46
pycairo==1.20.0
pygame==2.0.1
PyGObject==3.38.0
pynput==1.7.2
python-xlib==0.29
redis==3.5.3
six==1.15.0
```

You can either install them with pip command or use the requirements.txt file. Whatever sails your boat :)

```sh
# For individial packages
pip3 install <packagename>

# For requirements.txt
pip3 install -r requirements.txt
```

### 2. Redis

Redis is used for it's queueing mechanism, which will used to create an rtsp stream of the tello camera stream

Install the Redis Server

```sh
sudo apt install redis-server
```

### 3. Connect Tello

First, connect the Jetson Device to the WiFi network of Tello.

![Ryze Tello WiFi](resources/dronewifi.png)

Next, run the following code to verify connectivity

```python
# Importing the Tello Drone Library
from djitellopy import Tello
pkg = Tello()
pkg.connect()
```

On successful connection, your output will look something like this

```json
Send command: command
Response: b'ok'
```

If you get the following output, you may want to check your connection with the drone

```json
Send command: command
Timeout exceed on command command
Command command was unsuccessful. Message: False
```

## Running the Application

### 1. Clone the repository

This is a straightforward step, however, if you are new to git or git-lfs, I recommend glancing threw the steps.

First, install git and git-lfs

```sh
sudo apt install git git-lfs
```

Next, clone the repository

```sh
# Using HTTPS
git clone https://github.com/aj-ames/Hermes-Deepstream.git
# Using SSH
git clone git@github.com:aj-ames/Hermes-Deepstream.git
```

Finally, enable lfs and pull the yolo weights

```sh
git lfs install
git lfs pull
```

### 2. Run with different input sources

The computer vision part of the solution can be run on one or many input sources of multiple types, all powered using NVIDIA Deepstream.

First, build the application by running the following command:

```sh
make clean && make -j$(nproc)
```

This will generate the binary called `hermes-app`. This is a one-time step and you need to do this only when you make source-code changes.

Next, create a file called `inputsources.txt` and paste the path of videos or rtsp url.

```sh
file:///home/astr1x/Videos/Wildfire1.mp4
rtsp://admin:admin%40123@192.168.1.1:554/stream
```

Now, run the application by running the following command:

```sh
./hermes-app
```

### 3. Run with the drone

We utilize the livestream of the camera for real-time detection of wildfires.

Since the tello streams over UDP and the Deepstream Hermes app accepts RTSP as input, we need an intermediate UDP->RTSP converter. Also, we need to control the tello's movement.

Run the following command to start the tello control script:

```sh
python3 tello-control.py
```

This script will start the tello stream on the following URL:

```sh
rtsp://127.0.0.1:6969/hermes
```

To control the drone with your keyboard, first press the `Left Shift` key.
The following is a list of keys and what they do -

* `Left Shift` -> Toggle Keyboard controls
* `Right Shft` -> Take off drone
* `Space` -> Land drone
* `Up arrow` -> Increase Altitude
* `Down arrow` -> Decrease Altitude
* `Left arrow` -> Pan left
* `Right arrow` -> Pan right
* `w` -> Move forward
* `a` -> Move left
* `s` -> Move down
* `d` -> Move right

Finally, add the url in `inputsources.txt` and start `./hermes-app`.
