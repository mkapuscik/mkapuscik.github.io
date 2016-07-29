---
layout: post
title:  "Electron Microscope Digital Capture Retrofit: Part 2"
date:   2016-07-21 20:14:03 -0700
categories: Electron Microscope
---
## The Setup ##

*See [part 1](/electron/microscope/2016/07/18/Electron-Microscope-Capture-Interface/) for more information on the setup.*

The electron microscope has been pumping down to a high vacuum for the last couple of days and I have finally had some extra time to get everything setup and to test the capture process for a real image. I added a ~60 μm metal mesh grid to the sample holder and configured it to take a 200x magnification image at 8kV accelerating voltage.

![jpg](/assets/IMG_3112.jpg)
*Looking at the mesh through the CRT*

The image quality is pretty poor here, as all of the calibration coefficients I configured last month were wiped out after the machine was shut down for our trip to Maker Faire. I suspect there is a battery-backed RAM somewhere that needs a new cell, but I'll just continue with a rough calibration for now.

While the image was on screen, I set the scan rate to be 60fps and then initiated a single shot capture on the oscilloscope. After the 20 minute save process of a 300k row CSV, I had some data to play with. I wrote the following [IPython notebook](http://jupyter.org/) to process the raw traces and assemble the image:

## The Image Processing and Assembly ##

***In [2]:***

{% highlight python %}
%matplotlib inline
import math
import numpy as np
import matplotlib.pyplot as plt
plt.rcParams['figure.figsize'] = (20.0, 20.0)
{% endhighlight %}

***In [3]:***

{% highlight python %}
#import data from CSV from scope, skip few rows to avoid headers
data = np.genfromtxt(open("/Users/michaelkapuscik/git/mkapuscik.github.io/assets/NewFile6.csv","rb"), delimiter=',', skiprows=10, unpack=True)
#check to see if the data imported
print data[0:2][0:10]
{% endhighlight %}

    [[ 0.16  0.16  0.16 ...,  0.16  0.16  0.16]
     [ 4.    4.    4.   ...,  4.    4.    4.  ]]


***In [10]:***

{% highlight python %}
#plot the raw data to ensure it matches what we see on the oscilloscope
plt.plot(np.linspace(0,len(data[0]), len(data[1])), data[0], linewidth=0.1, alpha=1)
plt.plot(np.linspace(0,len(data[0]), len(data[1])), data[1], linewidth=0.1, alpha=0.3)
plt.plot(np.linspace(0,len(data[0]), len(data[1])), data[2], linewidth=0.1, alpha=0.3)
{% endhighlight %}




    [<matplotlib.lines.Line2D at 0x157b36850>]



 
![png](/notebooks/microscope_files/microscope_2_1.png) 
*Plot of the three signals from the oscilloscope*

***In [11]:***

{% highlight python %}
#clean up digital signals by thresholding
filtered_line_sync = data[1].copy()
#need to add in threshold detection to support arbitrary voltage offset from scope
filtered_line_sync[np.where(data[1] < 2)] = 0
filtered_line_sync[np.where(data[1] >= 2)] = 1

filtered_frame_sync = data[0].copy()
filtered_frame_sync[np.where(data[0] < 2)] = 0
filtered_frame_sync[np.where(data[0] >= 2)] = 1

#get frame boundries by differentiating digital framesync data and finding the first non-zero value
frame_edges = np.where(np.diff(filtered_frame_sync) != 0)
frame_start = frame_edges[0][1]
frame_stop = frame_edges[0][2]

#plot single frame of data to check assumptions
plt.plot(filtered_frame_sync[frame_start:frame_stop])
plt.plot(np.diff(filtered_line_sync[frame_start:frame_stop]), linewidth=0.2)
plt.plot(data[2][frame_start:frame_stop]/6, linewidth=0.2)

{% endhighlight %}




    [<matplotlib.lines.Line2D at 0x12b970bd0>]



 
![png](/notebooks/microscope_files/microscope_3_1.png) 


***In [13]:***

{% highlight python %}
#count number of line signal transitions by pl
line_start = np.where(np.diff(filtered_line_sync[frame_start:frame_stop]) > 0)[0]
line_stop = np.where(np.diff(filtered_line_sync[frame_start:frame_stop]) < 0)[0]
print "Line positive edges: %i, Line negative edges: %i" % (len(line_start), len(line_stop))

#get just single frame worth of data so indexing is easier
frame_lines = data[1][frame_start:frame_stop]
frame_intensities = data[2][frame_start:frame_stop]
{% endhighlight %}

    Line Positive edges: 237, Line negative edges: 237


***In [16]:***

{% highlight python %}
#plot all of the line of data to verify assumptions and look for incorrect decoding
for lineNum in range(len(line_start)):
    plt.plot(frame_intensities[line_start[lineNum]:line_stop[lineNum]], color="blue", linewidth=0.1)
    plt.plot(frame_lines[line_start[lineNum]:line_stop[lineNum]], color="red", linewidth=0.1)

{% endhighlight %}

 
![png](/notebooks/microscope_files/microscope_5_0.png) 

This graph shows all of the [scan lines](https://en.wikipedia.org/wiki/Scan_line) from a single frame plotted on top of eachother. Since the image is of a regular metal grid where the rows and columns are parallel to the image axes, the graph should show a repetitive pattern that has 6 troughs and 5 peaks. This is because the picture of the grid has 6 low intensity "holes" (the voids in the grid) and 5 high intensity "peaks" (the metal column areas) when divided up horizontally. Another cool thing about this approach is that it verifies the image is not inverted as the number peaks in the graph (bright areas) should match the number of vertical metal columns in view.


***In [19]:***

{% highlight python %}
#wrap the np.interp function to be a little more user friendly
def interpolate_line_data(line_beginning, line_end, data, frame_num, interpolation_size = 256):
    x = np.linspace(line_beginning[frame_num], line_end[frame_num], line_end[frame_num] - line_beginning[frame_num])
    xvals = np.linspace(line_beginning[frame_num], line_end[frame_num], interpolation_size)
    interpolated = np.interp(xvals, x, frame_intensities[line_beginning[frame_num]:line_end[frame_num]])
    
    return interpolated
{% endhighlight %}

***In [20]:***

{% highlight python %}
#create image array to hold decoded data, must be 256 lines vertically
image_width = 5000
image = np.ndarray(shape=(256,image_width))

#loop through lines and fill image with interpolated data on each line
#skip interpolating on vertical axis for now
for line in range(len(line_start)):
    image[line] = interpolate_line_data(line_start, line_stop, frame_intensities, line, interpolation_size=image_width)
    
transposed_image = np.transpose(image)
plt.imshow(image, interpolation='sinc', aspect='auto')
    
#plt.imshow(np.transpose(image))
plt.set_cmap("gray")
{% endhighlight %}

 
![png](/notebooks/microscope_files/microscope_7_0.png)
*I got an image!*

### The Good ###
The images are much better quality than what are displayed on the monitor as they are significantly higher resolution and don't suffer from artifacts due to ghosting or blanking. At 60fps, it's possible to generate more than 5000px of horizontal resolution by running the oscilloscope at a high sample rate, and this scales linearlly higher as the framerate goes down (30Hz should yeild > 10k px). Additionally, it is now possible to capture images at lower probe currents and oversample and average the data to reduce the detector noise.

### The Bad ###

The image generated is a 5000px x 256px image stretched to be 1:1 aspect ratio by [sinc interpolating](https://en.wikipedia.org/wiki/Whittaker%E2%80%93Shannon_interpolation_formula) the vertical axis. Unfortunatley, until a better method is found for sampling, the images are going to be restricted to 256 lines of real vertical resolution. This is because as the beam is traveling left to right on a single line, the fundamental limit on the horizontal resolution is how fast the analog voltages can be sampled from the photomultiplier (or more likley, the bandwidth of the amplifiers that buffer the  <abbr title="Photomultiplier Tube">PMT</abbr>), but the vertical deflection happens in discrete lines and cannot supply more information about the areas between these lines.

There are also now effictivley two brightness and constrast adjustments that can rob the image of resolution. The first is the microscope's brightness and contrast settings.  The microscpe controls the <abbr title="Photomultiplier Tube">PMT</abbr> bias point to move the signal up and down vertically, and the contrast adjustment controls the gain of the amplifier chain. The second is the vertical offset and vertical gain on the oscilloscope. To maximize the fidelity of the image, both of these settings need to be adjusted to maximize the signal on the oscilloscope.

## The Next Steps ##

I am pretty happy with how the images are turning out so far, though I plan on testing more to try and squeeze out any extra resolution. Direct network capture is the next priority, as a flash drive based capture takes forever, and hopefully LXI will alleviate that. After that, I have an Artix-7 FPGA board on the way that I plan on using to build a more permanant interface which should also be capable of higher framerate captures and maybe even HDMI or ethernet streaming in real time.
