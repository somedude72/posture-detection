## Posture Detection

<img width="324" alt="TN" src="https://user-images.githubusercontent.com/101906945/204119817-f6b19fa1-9890-4a67-8aeb-3b37161a269f.png"><img width="324" alt="TN2" src="https://user-images.githubusercontent.com/101906945/204119820-cf92690c-3703-42b0-b596-f7cddb25b570.png">

This page is designated for the Final Project submission for Nvidia and IDTech's program. This project is a machine learning model that can identify good sitting posture and bad sitting posture (Kyphosis and Forward Head) with both images and video. The model is trained from a tiny dataset consisting of only 100 images of bad posture and 100 images of good posture, all of which are taken from a webcam. 

This model works best when you face directly into the camera, and is designed to run on the Jetson Nano. Basic Linux terminal knowledge is recommended. 

## Installation

 - Install Python 3 on your Jetson Nano
 - Clone the jetson-inference project into your home directory
 
```bash
git clone --recursive https://github.com/dusty-nv/jetson-inference
```
 - Build the jetson-inference by running the following
 
```bash
cd jetson-inference
mkdir build
cd build
cmake ../
```
 > [!NOTE]
 > Leave everything as is during the installation process
 
 - Clone this repository into the jetson-inference folder
 ```bash
 git clone https://github.com/SomedudeX/Nvidia-Project
 ```
 
 - Start the docker container by running the following
 
 ```bash
 cd ~/jetson-inference
 ./docker/run.sh
 ```


## Image Recognition


If you would only like to detect stationary images, run the command below. Note that the model is trained to take the images from your webcam, so stock images from the internet is going to work less optimally and will have much higher error rates than images coming out of a webcam. For most optimal results, use video recognition. 

 - Run this command

  ```bash
imagenet.py --model=<RESNET18.ONNX> --input_blob=input_0 --output_blob=output_0 \
--labels=<LABELS.TXT> <INPUT IMAGE> <OUTPUT IMGAE>
  ```

 > [!NOTE]
 > Replace the text in angled brakets with paths to the file (not the directory). \<RESNET18.ONNX\> and \<LABELS.TXT\> should be the files downloaded from this repo. \<INPUT IMAGE\> and \<OUTPUT IMAGE\> should be the path to the image you wish to run and the desired output location and filename. 
 

## Video recognition

If you would like to detect moving objects or directly from your webcam, follow the instructions below. It uses the Jetson Nano to capture and process the image from the webcam, and then uses RTP to send it to your host machine to view. This is a bit more complicated than the image recognition instructions. 

 - Check your host computer's local IP address. 
 - Connect your camera to your Nano and check your video device number. 
   ```bash
   ls -ltrh /dev/video*
   ```
 - Run this command on your Nano to use the model along with RTP to broadcast the video data from your nano to your host computer. 

    ```bash
    imagenet.py --model=<RESNET18.ONNX> --input_blob=input_0 --output_blob=output_0 \
    --labels=<LABELS.TXT> /dev/video<DEVICE NUMBER> rtp://<IP ADDRESS>:1234
    ```

    > [!NOTE]
    > Replace the text in the angled brakets with the paths to the file (not the directory). For \<DEVICE NUMBER\> and \<IP ADDRESS\>, replace with the device number from step 2 and the IP address from step 1. 

Now that the Jetson Nano is broadcasting live video data, we are ready to receive that data from your host computer. There are two ways to do so. One is with GStreamer and the other is with VLC Media player. Using GStreamer is recommended as its latency is much lower than that of the VLC Media player. However if GStreamer does not work, VLC can be a fallback option. The following is taken from [this file](https://github.com/dusty-nv/jetson-inference/blob/master/docs/aux-streaming.md#rtp). 
    

   - Using GStreamer:
     - Install [GStreamer](https://gstreamer.freedesktop.org/documentation/installing/index.html) and run this pipeline (Replace `1234` with the port you are using)
  	
      ```bash
      $ gst-launch-1.0 -v udpsrc port=1234 \
      caps = "application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)H264, payload=(int)96" ! \
      rtph264depay ! decodebin ! videoconvert ! autovideosink
      ```
    	
   - Using VLC Player:
     - Create a SDP file (.sdp) with the following contents (Replace `1234` with the port you are using)
    	
      ```
      c=IN IP4 127.0.0.1
      m=video 1234 RTP/AVP 96
      a=rtpmap:96 H264/90000
      ```
	
     - Open the stream in VLC by double-clicking the SDP file
     - You may want to reduce the `File caching` and `Network caching` settings in VLC as [shown here](https://www.howtogeek.com/howto/windows/fix-for-vlc-skipping-and-lagging-playing-high-def-video-files/)
	
	
## Demo and Resources

 **[Image Recognition Demo](https://youtu.be/Y6P_PTaILX0)**
 
 **[Video Classification Demo](https://youtu.be/sSHaRQRecs8)**
 
 **[Furthur reading](https://github.com/dusty-nv/jetson-inference/)**
 
 
