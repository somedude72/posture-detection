<img width="324" alt="TN" src="https://user-images.githubusercontent.com/101906945/204119817-f6b19fa1-9890-4a67-8aeb-3b37161a269f.png"><img width="324" alt="TN2" src="https://user-images.githubusercontent.com/101906945/204119820-cf92690c-3703-42b0-b596-f7cddb25b570.png">


# Posture Detection
This is the Final Project for Nvidia and IDTech's program. It is a model that can identify good sitting posture and bad sitting posture (Kyphosis and Forward Head) with both images and video. This model works best when you face directly into the camera, and is designated to run on the Jetson Nano. Basic Linux terminal knowledge is recommended. 

# Prerequisites

 - A working Jetson Nano
 - A working computer that can SSH into the Jetson Nano via Visual Studio Code or Terminal

# Installation

 - Install Python3 on your Nano
 - Download/Clone the jetson-inference project from GitHub by running the following
```bash
git clone --recursive https://github.com/dusty-nv/jetson-inference
```
 - Build the jetson-inference by making a build directory and running the following in the build directory
```bash
cd jetson-inference
mkdir build
cd build
cmake ../
```
 - Clone this repository onto your Jetson Nano by running the following command
 ```
 git clone https://github.com/SomedudeX/Nvidia-Project
 ```


# Image Recognition


If you would only like to detect stationary images, run the command below. Note that the model is trained to take the images from your webcam, so stock images from the internet is going to work less optimally and will have much higher error rates than images coming out of a webcam. For most optimal results, use video recognition. 

 - Run the following command

  ```bash
imagenet.py --model=<PATH TO DOWNLOADED RESNET18.ONNX> --input_blob=input_0 --output_blob=output_0 \
--labels=<PATH TO DOWNLOADED LABELS.TXT> <PATH TO INPUT IMAGE> <PATH TO OUTPUT LOCATION>
  ```

 > **Note**: When copying, replace the text in angled brakets with their respective paths. For \<PATH TO DOWNLOADED RESNET18.ONNX\> and \<PATH TO DOWNLOADED LABELS.TXT\>, replace with the path to the files from this repo. For \<PATH TO INPUT IMAGE\> and \<PATH TO OUTPUT LOCATION\>, replace with the path to the image you wish to run and the desired output location and filename. 
 

# Video recognition

If you would like to detect moving objects or directly from your webcam, follow the instructions below. It uses the Jetson Nano to capture and process the image from the webcam, and then uses RTP to send it to your host machine to view. This is a bit more complicated than the image recognition instructions. 

 - First, check and notate your computer's (not the nano's) local IP address. We will need this later. 
 - Then, connect your camera to your nano and check your video device number by running the following
   ```bash
   ls -ltrh /dev/video*
   ```
 - Run the following command on your nano to use the model along with RTP to broadcast the video data from your nano to your host computer. Replace <DEVICE NUMBER> with your video device number, and <LOCAL IP ADDRESS> with your computer's local IP address. 

    ```bash
    imagenet.py --model=<PATH TO RESNET18.ONNX> --input_blob=input_0 --output_blob=output_0 \
    --labels=<PATH TO LABELS.TXT> /dev/video<DEVICE NUMBER> rtp://<LOCAL IP ADDRESS>:1234
    ```

    > **Note**: When copying, replace the text in angled brakets with their respective paths. For \<PATH TO DOWNLOADED RESNET18.ONNX\> and \<PATH TO DOWNLOADED LABELS.TXT\>, replace with the path to the files from this repo. For \<DEVICE NUMBER\> and \<LOCAL IP ADDRESS\>, replace with the device number of your camera from step 2 and your computer's local IP address from step 1. 

 - Now that the Jetson Nano is broadcasting live video data, we are ready to receive that data from your host computer. There are two ways to do so. One is with GStreamer and the other is with VLC Media player. Using GStreamer is recommended as its latency is much lower than that of the VLC Media player. However if GStreamer does not work, VLC can be a fallback option. The following is taken from [this file](https://github.com/dusty-nv/jetson-inference/blob/master/docs/aux-streaming.md). 
    

    * Using GStreamer:
  	  * [Install GStreamer](https://gstreamer.freedesktop.org/documentation/installing/index.html) and run this pipeline (replace `port=1234` with the port you are using)
  	
    	```bash
    	$ gst-launch-1.0 -v udpsrc port=1234 \
    	caps = "application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)H264, payload=(int)96" ! \
    	rtph264depay ! decodebin ! videoconvert ! autovideosink
    	```
    	
    * Using VLC Player:
    	* Create a SDP file (.sdp) with the following contents (replace `1234` with the port you are using)
    	
    	  ```
          c=IN IP4 127.0.0.1
          m=video 1234 RTP/AVP 96
          a=rtpmap:96 H264/90000
    	  ```
	
    	* Open the stream in VLC by double-clicking the SDP file
    	* You may want to reduce the `File caching` and `Network caching` settings in VLC as [shown here](https://www.howtogeek.com/howto/windows/fix-for-vlc-skipping-and-lagging-playing-high-def-video-files/)
	
	
# Demo

#### [Image Recognition Demo](https://youtu.be/Y6P_PTaILX0)
