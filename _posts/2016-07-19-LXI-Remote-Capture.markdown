---
layout: post
title:  "LXI Remote Capture"
date:   2016-07-19 20:14:03 -0700
categories: Tools
---
During the documentation of the SEM repair and capture interface build I had to dump a bunch of images and waveform data to a USB drive and then copy that data onto my computer. That got old pretty quickly so I wanted to find a better way to get data off of the scope. Fortunately the Rigol scope I have is [LXI](https://en.wikipedia.org/wiki/LAN_eXtensions_for_Instrumentation) equipped so it should be possible to do a screengrab remotely and save it on my machine all in one step. This would make writing documentation significantly easier as I wouldn't have to move flash drives back and forth.

After about 20 minutes of consulting the [programming manual](http://www.batronix.com/pdf/Rigol/ProgrammingGuide/MSO1000Z_DS1000Z_ProgrammingGuide_EN.pdf) and working in IPython I ended up with the following notebook. I'm using [PyVISA](https://pyvisa.readthedocs.io/en/stable/) to handle the SCPI/LXI so the script ends up being pretty short.



## LXI remote screen capture ##
This is a short script that does a screen grab of a remote oscilloscopes current screen image and saves is locally.



```python
import visa
import datetime
```


```python
# Connect to scope
ip = "10.18.14.171"
INSTRUMENT = "TCPIP::%s::INSTR" % (ip)
rm = visa.ResourceManager()
scope = rm.open_resource(INSTRUMENT, timeout=20)
```


```python
# Verify we are talking
scope.query("*IDN?")
```




    u'RIGOL TECHNOLOGIES,DS1104Z,DS1ZA171509445,00.04.03\n'




```python
# Query for the current screen data in BMP24 format, use uchar encoding
returned_data = scope.query_binary_values(":DISPLAY:DATA?", datatype='B')
#record time of aquisition so we don't get multiple names for one file
aquisition_date = datetime.datetime.now().strftime("%Y-%m-%d-%H:%M")
```


```python
filename = "scope-capture-" + str(aquisition_date)
with open(filename + ".bmp", 'wb') as f:
    f.write(bytearray(returned_data))
```

![](/assets/scope-capture-2016-07-19-21-43.bmp)
*It works!* 

## Next Steps ##

Image capture off of the scope is something I do so often that I will probably make this a command line utillity with some error checking and additional options like auto clipboard population of the file name to make writing the markdown even easier.

