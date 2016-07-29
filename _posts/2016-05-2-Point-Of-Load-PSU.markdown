---
layout: post
title:  "Point of Load Power Supply for LED Light System"
date:   2016-05-2 20:14:03 -0700
categories: Electronics
---
### [Github Project](https://github.com/mkapuscik/POLPC) ###

![png](/assets/point-of-load/rendering.png)
*Rendering of the power supply*
![jpg](/assets/point-of-load/IMG_7330.jpg)
*The first completed board*

[Dan](https://github.com/dloman) and I have been working on designing a large addressable RGB light installation at [SBHX](http://www.sbhackerspace.com). We want to have 1000 RGB lights distributed over about 100 linear feet of ceiling and be able to power each segment without any dicernable dimming caused by execssive loading. The entire installation will need 65A at 5V and will tolerate only about half a volt deviation before light output is effected. 

The naïve approach to this problem would be to run a cable directly from the power supply to the lights; however, with the currents involved we would need a 2/0 AWG (about 10mm diameter) conductor pair and we would lose about 20% of our power in the cable. Even ignoring the power loss, that solution is prohibitally expensive and running cabling that thick and inflexible would be a nightmare.

## Point of Load Power Supplies ##
Since the advent of low cost DC-DC converters, and with ever lowering operating voltages for modern electronics, a different topology of high current power sources has become necessary: [point of load power supplies](http://electronics.stackexchange.com/questions/231325/what-is-a-point-of-load-converter). The idea of a PoL system is to distribute a number of intermediate power supplies throughtout the system directly adjacent to where the power is needed, and then power these intermediate supplies with a higher voltage, lower current main power supply. This significantly reduces losses in cabling as power dissipated in a cable is proportional to the square of the current traveling through it (P=I^2R) and allows for much thinner (cheaper) cabling. It also has the added benefit of improving the noise figures at the loads because each PoL supply is activley regulating down from a significantly higher voltage and will be less likley to run out of headroom to regulate due to cable voltage drops.

## The Design ##
The main requirements for the power supplies I needed were as follows:


* Low cost (Under $10)
* High efficiency (~90%)
* ~12V input capable, 5V output
* Good thermal performance

There are thousands of DC-DC regulators, but due to the thermal and efficiency requirements, I was able to narrow the options down to just [syncronous regulators](https://en.wikipedia.org/wiki/Buck_converter#Synchronous_rectification) as the diode losses inherent in non-syncronous regulators would likley exceed a watt and significantly contribute to waste heat.

I settled on the [Texas Instruments TPS54520](http://www.ti.com/product/TPS54620) as it fit well into all of the requirements and had the added benefit of having [monotonic](https://en.wikipedia.org/wiki/Monotonic_function) startup characteristics, so the serial receivers in the LED's will power up glitch free. It also had a power good status output that could be linked to another supply's enable pin to ensure the power supplies start in a sequence instead of all simultaneously, which could cause huge current demands.

The design was pretty straightforward for a switching supply, with typical constraints like minimizing loop area of switching nodes, isolating analog traces from noise, and ensuring low thermal resistance to high current nodes. I went pretty heavy-handed when it came to heatsinking the regulator as can be seen here. I was thinking that there is no such thing as too much copper for high current nodes, but as I'll note later, this was actually a weak point of the desing.

![png](/assets/point-of-load/large-copper-area.png)
*Large copper fills on VIN, Inductor out, and GND*

I also wanted to experiment with improving raster based board logos. I have previously had some success with generating logos in the silk layer but since OSHPark has a relativley coarse silkscreen process, I wanted to try out a logo in the copper. Since OSHPark has ENIG plating standard that would mean that the logo would be gold plated too, which ended up looking awesome. 

I generated the logo with a script in altium that chops the raster image up into horizontal traces of a given width, and it ended up looking choppy in the preview. I decided to releive the image by running the raster importer again on the same image, but with a significantly larger trace size, and then copied that larger trace size version of the image onto the soldermask layer.
![png](/assets/point-of-load/sbhx-logo.png)

Here you can see the larger width of the soldermask version behind the smaller copper version of the image effectivley forming a soldermask void around the outside of the image.
![png](/assets/point-of-load/mask-cover.png)
*Soldermask line highlighted to show size difference in trace*

![png](/assets/point-of-load/rendered-mask-relief.png)
*Rendering of image relief*

## Assembly ##
![jpg](/assets/point-of-load/IMG_3683.jpg) 
*Checking designators and values and preping for assembly*

![jpg](/assets/point-of-load/IMG_5699.jpg)
*Board holder for solderpaste stenciling*

![jpg](/assets/point-of-load/IMG_3932.jpg)
*Solderaste applied*

![jpg](/assets/point-of-load/IMG_4493.jpg)
*Board components placed, ready for reflow*

Everything went smoothly when populating the boards and components were pretty easy to place. After reflow, I did notice two of the boards had some issues soldering.

![jpg](/assets/point-of-load/IMG_0136.jpg)
*Switching regulator is starting to tombstone*

![jpg](/assets/point-of-load/IMG_1895.jpg)
*Gap can be seen between the board on the nearest IC face*

After inspecting these solder failures, I have determined that it was due to too much copper and no thermal reliefs at the IC. I think that as the boards were reflowing, the larger copper areas were taking longer to heat up (as they have higher thermal mass), meanwhile on the lower thermal mass sides the solder was already becoming moltent and the surface tension began to pull the IC to the this side.

In the future, I will avoid running large pours next to high density packages like these, or thermally releive their connections, but all things considered, I only had two failures, so I think I got off pretty easy on this mistake.

![png](/assets/point-of-load/IMG_9498.jpg)
*First batch of boards completed and ready for testing*

## Testing ##
![jpg](/assets/point-of-load/IMG_9695.jpg)
*Test setup for temp measurement*

![jpg](/assets/point-of-load/IMG_6195.jpg)
*Quick dummy load I hacked together out of a high current MOSFET, cooler with Aluminum block in water*

![jpg](/assets/point-of-load/IMG_5773.jpg)
*Thermal test setup*

## Thermal Imager Macro Lens Hack ##
As a quick asside, I thought I'd mention the this hack for SBHX's Seek thermal. One of our laser cutters at the hackerspace needed an objective lens replacement as it was not performing as well as it should. So we took the old one out and printed an adapter to affix it to the Seek thermal imager. 

![jpg](/assets/point-of-load/IMG_3126.jpg)
*Laser cutter objective lens adapter*

![jpg](/assets/point-of-load/IMG_3127.jpg)
*Lense adapter to Seek thermal imager*

These CO2 laser cutter lenses are usually a zinc selenide substrate coated with an anti-reflection layer and are manufactured to maximize transmitance around longwave IR (8-15um) which happens to be close to the range the seek operates in (7-14um).

![jpg](/assets/point-of-load/ZnSe Total Transmission.jpg)

This custom adapter for the laser lens is effectivley a IR macro lens and allows me to take much higher resolution pictures of small objects like components on a PCB.

## More Testing ##

To ensure these supplies would perform as desired I tested each one in a 30 minute 130% max load test with only open air heatsinking. The maximum temperatures I saw on the chip were 117C.

![jpg](/assets/point-of-load/img_thermal_1454666055558.jpg)
*Max temp soak test*

The test setup for the supplies had them dissipate ~3.4 watts so the die temp can be calculated as follows TJ = TC + ΨJT * Pdiss 

This yeilds a die temperature of 118C which leaves 32C of "headroom" until the device starts to thermally throttle. As this is a worst-case analysis I'm pretty comfortable saying these devices will perform quite well in service.

All of the boards passed this temperature soak and were soldered into the installation. 

## Design Improvements ##

Beyond improving the manufacturing yeild by fixing the copper pour problem I mentioned above, the main thing I'd like to improve about the design were the connectors or lack-there-of. I chose to use wire to board connections as I could not find low cost high current connectors that I liked however navigating digikey's connector catalog is tantamount to searching for a needle in a haystack so it's entireley possible that I just was looking in the wrong place. Input protection would also be a really nice feature, as these supplies were so well performing that all of the extras I had that didn't make it into the installation have already been used up in my other projects.


