# <Project Title>  
*<Short subtitle or wry aside that hints at the problem being solved>*

---

## Overview

When I started this project, I had no plans for the Jensen to become a plug-in hybrid. I was still, if you can believe it, holding onto some semblance of Colin Chapman’s “Simplify and add lightness” philosophy, and a charger really didn’t add to any of the sexy-ness of a powerful performance hybrid. To this end, when I first “charged” the Jensen, it was by setting the regen parameter (manually, in the inverter EEPROM serial interface) to be roughly -20 Nm and high-idling the engine with a credit card between the the throttle linkage and the backstop. After doing this for about 30mins, I decided it was time to work a charger into my plans.

---

## Version 1
After some research, I found the Gen 2 Chevy Volt charger was fairly compact and cheap, watercooled, and had been sufficiently reverse-engineered by the OpenInverter community. After calling between a number of dismantlers off Car-Part.com, I finally locked down my very own for about $120.

Funny thing- the Gen 1 Chevy Volt Power Electronics suite was generally air-cooled (save, the charger) and controlled via 500kbps CAN. Then, when it came time to release the Gen 2 Chevy Volt, the team at Chevy opted for watercooled, PWM controlled hardware. Maybe it was for cost, maybe to help fight against noise- I’m not entirely sure, I just know it was a PITA. The trouble is that the control signals for the charger were given in 100Hz 12V PWM, and the feedback signals, such as charger temperature, output voltage, output current, were also outputted in 12V PWM. In order to get 12V PWM, I wired up an Arduino to an Amazon’d FET board and checked the signal output with my oscilloscope. 

Then, I went and bought the necessary HV and LV connectors to output power and communicate. The HV DC out I pinned from the charger-side sealed and shielded HVA280 to a very unsealed and  unshielded Anderson connector. The HV HV AC input I pinned directly to a chopped 120V extension cord. I then stacked my buddies used transmission oil cooler on top, and hooked up a 12V (Amazon’d) water pump to circulate coolant. With all the proper connections and control hardware in order, all that was left to do was a bedroom floor bench test.

Now that it has been thoroughly tested to the highest standards, it’s time to hook it up directly to the car. There’s only one “AUX” HV output out of my contactor box (see: didn’t plan for a charger), so when charging, the DCDC had to be disconnected. From there, it was a short matter to hook up a surge protector to an extension cord to power a DC power supply, which provided 12V to the charger PWM supply / water pump and a USB 5V to power the Arduino.

And with this new setup, I was able to charge! My charging speed was limited by the outlet I was attached to, which put out roughly 1.4kw as long as nobody was grinding, welding, using the air compressor, or microwaving dinner. 


I continued like this for the next year or so until one day the setup stopped working. It could have been a blown breaker, or a loose wire in the Arduino broken free, but I honestly didn’t care. I'd decided I had enough of this mess of a charging solution. It took forever to set up, looked like a bomb, and was unreliable. No longer would I have to spend 20mins to set up a charger that I constantly had to nurture.

---

## Version 2

Lucid’s Space Concept had clearly not made it to 1970’s West Bromwich, England- and so the Jensen-Healey was burdened with extraordinarily volume between the unibody and the fender. This volume has been used by another member of the Jensen-Healey community to store a dry sump oil reservoir. I planned to use this space to:

A. House a charge port

B. Stow away the Chevy Volt charging unit

My initial concept was to make a NACS port housed in a trapezoidal flap area, with an adjacent “Fuel” physical gauge that showed SOC. I still sometimes wish I went this route, I wanted to include more mentions of the Jensen’s trapezoid motif and the physical gauge is still a really cool idea … but I know that it would have been a nightmare to implement. First, I would need to find and package a swing-arm mechanism, then I would need to include adjustability in the design so that the panel gaps all line up- all while packaging the Chevy Volt charger as well. One of my problems is scope creep, and focusing on shock-and-awe more than reliability and functionality, so I decided to take the boring high road on this one.

I figured that just cutting the bare minimum off the fender- just enough for a plug to fit, plus a little area for a gentle lead-in might be an acceptable, clean solution. I designed a small ‘carrier’ to hold a Telsa charge puck, that had features for easy and rigid mounting to the body. In this respect, I also took the easy way out- opting to rivet the carrier to the fender wall.

Got the project done just in time to bring the Jensen in to Lucid’s annual Cars and Caffeine. Apparently, the NACS port was too subtle to be noticed by most of anyone, so I was egged on to bring out a mobile charging plug to really underline the feature. The car got a lot of good exposure and it was awesome to finally show all of my peers the car I had been talking about for the past year.

All was well and good, but there was of course the underlying issue that this instance of the charger was non-functional. Once home again, and after a long debugging session, I found that something got lost in the translation of my original Arduino code from ChargerV1 to ChargerV2, in which the PWM function didn’t output properly. I quickly fixed this and was a step closer to convenient, easy, fast charging.

Finally! A working charger, all packaged neatly in my fender. To celebrate, I decided to spend the afternoon supervising its maiden charge from 2% to 100%. Running at a crisp charging speed of 1.2kw, it was a beautiful hour of charging. Now, readers paying attention may note that my battery pack is not 1.2kwh. The Arduino somehow burnt out within an hour of charging and an oscilloscope confirmed that the PWM pin was no longer making square waves. Part of me must have expected this, as I had not taken a single step towards packing up the fender until I saw the charger work flawlessly. 

---

## Version 2.1

Done with the Arduino’s shenanigans, I bit the bullet and tried the most reliable version of this I could- running the PWM off of a spare 5V driving ignition output from my MaxxECU (why is it that the MaxxECU is always the answer?!) This, of course, worked perfectly the first time and had no issues charging the battery for the 4 hours it took to get to 100%. This is not to say that the charger is bottlenecked at ~1.2kw, I was just meaning to play it safe. These batteries are rated to take far more than the rated 7.2kw of my charger.

My control schema within the Arduino’s code was heavily limited by my programming ability- offering a constant PWM duty to the Chevy Volt charger as long an external ground switch was applied. Within the MaxxECU, the story was very different. I mapped the control signal PWM duty to a 4D map of cell voltage and temperature, and a third “flag” parameter. 

Charge speed would dwindle as the full charge voltage is achieved. Currently the charge current is held constant throughout charging- I have yet to chat with a cell expert to see if this is wise. I also have it here that excessively high temperatures will result in a derate in PWM duty. Finally, once the cell voltage exceeds 4.060V, a flag is triggered within the ECU and charging control PWM is dropped to 0% duty. This flag is latched and the only way it can become unlatched is if the cell voltage rises back to 4.060V (not likely to happen without the charger’s influence). In addition, this flag triggers the left turn signal light on the Mini Cooper Tach- to indicate to me that the charge session has completed. It’s not a perfect indicator, but it was green.

## References & Supporting Material




