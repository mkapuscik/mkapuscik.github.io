---
layout: post
title:  "Electron Microscope Digital Capture Retrofit"
date:   2016-07-18 20:14:03 -0700
categories: Electron Microscope
---
## The Plan ##
Now that the microscope is operational, I would really like to be able to record high quality digital images and maybe even videos. Having spent some time looking over the schematics, I am pretty confident that retrofitting a digital capture interface will be possible. In order to accomplish this, the aquisition interface board will need some sort of frame and line sync information and analog sensor values from the detector so the image can be properley reconstructed. My first thought is to follow the left CRT video signal path backwards until I hit the video mixing responsible for overlaying text.

## Working Backwards from the CRT ##

![](/assets/IMG_2913.jpg)
*Left CRT image with text overlayed*

The left monitor shows the image from the secondary electron detector superimposed with magnification and acceleration voltage text. In order for this to happen there must be some sort of digital to analog converter in the signal path to generate the text characters and then either analog or digital video mixing to combine them. This circuit would be the most logical place to look for video sync signals as well as unaltered detector intensities.

Unfortunatley, when following the signal path, I failed to find any sort of digital video generation. It turns out that all of the video mixing happens at the monitor. Time to find a new apporach.

![](/assets/IMG_3116.jpg)
*2 ports provide analog intensity streams for text and secondary electron detector seperatley*

## Plan B ##

The next logical place to look would be the scan generation system that controls the deflection coils on the electon beam. This is ultimately the source of the horizontal, vertical and frame blanking signals, so starting here and working back up the signal path seems like the next course of action.

Following the signals up from the deflection coils lead me to a group of signals that connected to the video backplane and several of the video processing cards. 

![](/assets/V_SYNC_Data.png)
*60Hz repeating, 237 pulse active high signal*

One of these signals looked like it was a line blanking signal as it had a period of 60Hz, which was the video frame rate at the time I was taking the capture, and contained 237 pulse (this was a little odd as the microscope reports it has 256 scan lines). To double check my theroy I changed the capture rate, and the signal reflected the change. 

![](/assets/IMG_3100.jpg)
*R43 is a pullup for the vertical blanking signal*

![](/assets/IMG_3102.jpg)
*Breakout wires soldered onto scan generation card (I only added the light gray and red wires, the rest were bodge wires from JEOL or the previous owners)*

I removed the second scan generation board from the rack and soldered some wires onto the bottom of R43 to tap into the vertical blanking signal.

![](/assets/DS1Z_QuickPrint3.png)
*Result: 60Hz frame blanking signal on CH1, filtered PMT signal on CH3*

And to double check this was correct I changed the scan rate to 30Hz on the microscope and verified the frame blanking signal frequency matched.

![](/assets/IMG_3103.jpg)
*Scan speed set to SR (30Hz)*

![](/assets/scope-capture-2016-07-19-21-55.bmp)
*30Hz pulses at output*

To double check that I had all of the signals I wanted and they behaved as expected, I decided to inject a squarewave from a function generator into the photomultiplier input on the microscope. I chose a frequency that was an even multiple of the scan rate to generate a mostly stationary stiped image, and then captured a single frame of data with the oscilloscope to verify the values I got were not unreasonable.

![](/assets/IMG_3106.jpg)
*Display of image injected from function genrator*

![](/assets/scope-capture-2016-07-20-01-57.bmp)
*Output of nodes responsible for video production*

Everything looks good!

## Next Steps##

The next steps are to build out a python script to process the raw traces and compile them into an aligned image. It's almost 3 A.M. so I am going to call it quits for tonight; the microscope needs to continue pumping down anyways. I'll come back tomorrow to work on the image processing script and with any luck will successfully generate a digital image from the capture interface.



