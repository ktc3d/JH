# <Project Title>  
*<Short subtitle or wry aside that hints at the problem being solved>*

---

## Overview

Briefly describe **what this sub-project is** and **why it exists**.  
This should answer, in 2–4 sentences:

- What problem are we solving?
- Why the factory solution was a compromise
- Why this change improves the car in the real world

Think: engineering context, not a build diary.

---

## Motor Selection

I chose the motor out of Hyundai Ioniq Plug-In Hybrid. I chose it because, while researching, I found that it was unique in that it had a throw-out bearing and input spline built into the motor housing. Having recently swapped my transmission over to a T5, I was familiar with these elements and the hybrid project seemed more achievable. Additionally, the motor casing casting was self-enclosed and had a bolt pattern (not some weird splined interface to a larger housing), which meant adding it onto my engine was just a matter of making an adapter plate — something, again, I had done before during my T5 transmission swap project.

With the P2 layout, there were a good many constraints — primarily axial length and packaging issues. For instance, my friends were egging me on to add an extra clutch between the transmission and the electric motor. This would have solved a couple problems I'm dealing with now, but would have increased the length, cost, and mechanical complexity of the drivetrain a fair bit. I chose to take on a bit of tech debt in this regard and solve the inevitable motor-rotor/input-shaft inertia issue with software later (see speed-matching shifter section). I designed the adapter plates to have a pretty solid locational tolerance of the center axis, but I fear that in my haste of installing the system (in order to move out of my college house in time), this alignment may have suffered. I hard-committed to the P2 layout when I bought the hybrid motor. It cost me $800 from a salvage dealer — at the time that was basically half of my money, so when it arrived, I knew I was in for it.

---

## Clutch And Shifting Challenges

Ok, so when you're shifting gears on a normal car, the clutch separates the input shaft of the transmission from the engine. This is important, because when you're shifting, you're basically insisting that the ratio of input shaft speed vs output shaft speed be different from a prior gear.

In order for this statement to be true, either the car (output shaft) needs to change its speed to match the car, or the engine (input shaft) needs to change its speed to match the ground. Since we really care about the vehicle speed, and because the inertia of the vehicle is so massive compared to the engine's inertia, the whole speed-of-the-vehicle aspect wins out. While it is easier to change the engine speed compared to changing the vehicle speed, it's still fairly hard to force the engine to rapidly change speed — this causes a lot of undue wear on the transmission. However, once the engine is separated via clutch from the transmission input shaft, it suddenly becomes very easy to change the speed of the input shaft, because the input shaft itself has such a small rotational inertia. This is how most manual vehicles operate. BUT, if you happen to attach an electric motor's rotor DIRECTLY onto the input shaft, even if you clutch in, the electric motor's rotor is still preventing the input shaft from any quick changes in speed. Even worse, the rotor is also being commanded torque by the ECU, etc — which tosses a wrench in things.

My friends suggested adding the second clutch to alleviate this issue — letting the input shaft be free. I rejected this idea because short clutches were very hard to find and were expensive. Even dual-plate clutches were fairly tall, not to mention very expensive. I believed that intelligent control could solve all of these problems with ease. I was half-right ... software could fix these issues, but I had to invest the time to develop and tune it.

The shifting behavior in the early days was very rough. You had to ensure your foot was off the throttle, and regen had to be off. The synchros did their best to manage, but they very quickly grew tired of forcing a speed change on the uncooperative motor. Shifting, especially into second gear (the most common shift in the testing stage), was getting very difficult — almost impossible. The solution was to make the motor speed-match itself to ensure a perfect shift. This required a hardware change, as I needed to know the shifter position. So, I built a gearshift position sensor PCB. I cover the development of this in a page on the website.

## Battery Pack

For the batteries, I was heavily influenced by where I worked at the time: EV West. I was told I would be given an employee discount on the batteries, so I limited my selection to those in stock. Because I was not making an EV, my focus was only on batteries with a high S-count-to-capacity ratio, with high power output. This aligned well with a battery pack that was abundant in our warehouse — these little 6P13S packs meant for electric scooters. These were nice because they were really power dense and the brick structure was easy to package. They came with nice JST connectors for BMS and thermistor output. I wanted to max out the possible power output of my inverter (100 kW). In order to do this, I napkin-mathed that I needed 12 battery modules to hit this output (peak amp output times nominal voltage). The Hyundai motor I chose ran at ~270 V, so I wanted to get in that neighborhood with the S count. This brought me to run the modules 2 in parallel, 6 in series, for a total of 12p78s.

Wiring the BMS was a huge PITA. There were 78 voltage leads and 12 thermistor wires I needed to keep track of. The BMS (Dilithium BMS) also had some specific rule about how to order the cells. For instance, each BMS satellite could hold 24 voltage leads, but each needed 4x cells minimum to be connected to power the board. Also, any extra unused cell voltage inputs needed to be tied to the highest used cell voltage input in parallel. Plugging in either of the 2x 14-pin connectors in the wrong spot would cause the unit to burn out, so putting this together felt a bit like rocket surgery. It took days, and any mistakes meant I had to re-wire large portions of the harness. Before connecting any hardware together, I series'd the modules I had into full pack-voltage with 24AWG wires (for safety — any short would burn out the wires instantly), and checked the voltage of each voltage lead individually and manually. I did this about 5 times before I was confident enough to hook up the actual BMS hardware to my batteries.

SOC estimation is a bit of a sore spot for me right now. Initially, I built the system with a TBS Expert Pro battery monitor gauge. That worked somewhat well, but didn't jibe well with the manual-shutoff-switch I insist on using when the vehicle is dormant. Using this switch kept resetting the monitor to 100% SOC. It was also hard to get data off of this onto other devices, as it depended on a somewhat proprietary serial interface. So, my most reliable SOC estimation was from unloaded voltage. I realize, especially now while working at an EV company, that this is distasteful, but I figured as long as I stay between 20% and 90% SOC, I'm not really pushing the boundaries of the cells into any dangerous zones where intensive monitoring would be needed.

## Inverter

I actually started out the project trying to make a Gen3 Prius inverter work, using hardware/software from OpenInverter. I tried for about 4 months to "make sinewaves," to no avail. It got to the point where diagnosing this inverter invaded my every thought, kept me awake at night, preventing me from studying — and most importantly, was holding up this project from proceeding. With the DIY solution falling through, I decided it would be best for the project and my sanity, to go with an inverter solution that was more off-the-shelf. The PM100DX had a lot of formula-student-derived testing and experience behind it, and had a pretty solid ECU built-in with CAN bus functionality and a lot of other very useful features, such as the option to run in torque-mode or speed-mode. Once I got this unit running, it was a short time to get the Hyundai motor spinning.

## System Wiring And First Power-Up

How I put the HV system together was largely bounded by packaging. The Jensen Healey, being a roadster, is not quite boundless in terms of potential battery space. I debated between a couple spots — under the trunk space and behind the seats. I steered away from the sub-trunk-space for two reasons: One, out of fear that fire propagation would have more dire consequences if heat propagated to the fuel tank, just above in the trunk. Two, the weight distribution wasn't great, adding ~150 lbs to what was essentially just the rear axle. Behind-the-seat storage fared better in both of these categories, at the price of ~2" of legroom.

The only limits on current are judged by either fuse size or by parameters in the PM100DX. I chose fuses and current limits that allowed for short, sustained (~30 s) bursts of 100 kW. These current limits are de-rated if inverter or motor temp gets too high. Battery temperature, surprisingly, has never really been an issue. The cells keep fairly cool, with the passive air cooling within the pack.

There were no real surprises when I hooked it all up — it all went fairly well.

## Control Integration

That's a good question! And the answer is no, actually. When I first started with the hybrid system, the throttle was actually controlled via an analog throttle "pot" signal. During my tenure at EV West, I actually designed the throttle pot I was using. The idea for the device was to convert a linear cable input into an analog signal for an ECU/VCU to read. So, it's a little bit fate that I had an application for my assembly, immediately after I had designed it. I picked one up from EV West and used it as a hand throttle. The bifurcation of the EV control and the gas control was pretty helpful in the beginning, but eventually, the need to shift, and steer, and apply EV throttle became too much for my two hands.

So, I realized I'd need to integrate the EV throttle into the MaxxECU so that it could all be blended smoothly. I switched the PM100DX from analog-control mode to CAN-control mode, rendering the pot sensor vestigial. On the MaxxECU side, I made a 2D User Table, plotting my throttle position sensor (TPS) signal from my ITB's against engine RPM. Commanded torque would rise with TPS signal in a delayed somewhat linear fashion. RPM really only affected the regen torques on the higher up end. I didn't want to have the same regen at high rpm's as I did at the low end, as this may cause some over-voltage conditions.

There was no meaningful regen during shift events — this wouldn't work well, as when I'm shifting, the motor is disconnected from the wheels. Engine torque and motor torque blending was never a real issue. The engine has no issue driving up a hill or down a hill — it just deals. The motor regen-ing or boosting is no different.

## References & Supporting Material

- Datasheets
- CAD screenshots
- Wiring diagrams
- Firmware configs
- External resources


