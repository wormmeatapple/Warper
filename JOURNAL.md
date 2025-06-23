---
title: "Warper"
author: "wormmeatapple"
description: "A CCD camera packed full of circuit bending opportunities!"
created_at: "30th/5/2025"
time taken: ? hours
---

## May 30th

<p>First things first is to start researching. My research into circuit bending led to an answer I really didn't like, no one knows how to predict it. You just mix, modify and disrupt data signals in the hopes it makes something cool. This is not ideal but it is fineee, because I know (slightly) how a camera sensor works.</p>

<p>A camera sensor (at least the one I plan to use, a CCD(charged coupling device)) is really just an array of photodiodes that all output a little voltage depending on how much light they received. This voltage is moved into a buffer and then moved into a seperate buffer that handles one pixel at a time. That little pixel is amplified, and then processed into digital format and displayed. All of this is handled by little signal pulses, and my thinking is that if I can incorporate pentiometers with tiny ranges in resistance I can mess up some of the thinking and boom, bends formed. I'll also add ways to short two data lines together, just for the fun of it.</p>

![image](https://github.com/user-attachments/assets/f9f5dbdd-8a70-4140-9766-6c44d5743d8e)


<p>My current biggest roadblock is finding a god dam image sensor that I can actually source and put into my pcb, so far the only one I'v found is about a 1/6 inch and has 258x299 pixels. Bruh.</p>

<p>Okay I've found a sensor that looks promising. Buttt it;s about 250 dollars.</p>

![image](https://github.com/user-attachments/assets/284be0a4-5cfb-4543-805f-eddfb83327e8)

<p>Aha! I can just buy one of these older cameras and take the sensor from them. Hopefully. I've found the datasheet for that sensor as well, so it should be good for me to start designing the drive circuits for them. First time I'll be doing anything so complex, but we'll see how it goes.</p>

> Time taken: Probably **1h 30m**
> - Not sure if I'll count this in my total, it was all just research
> - Also definitely longer than 1h 30m icl


## May 31th

<p>I've continued to research the datasheet, it's alot of new infomation but I'm getting through it steadily. So far I've figured out I need these signals;</p>

<p>- 0V to -7V; ~10 to 100khz; For the vertical registers; Vφ <br>
- 0V to +5V; 28.64 MHz; For the horizontal registers; Hφ <br>
- 0V to 3.3V; 28.64 MHz (but pulsed); For the reset gate; RGφ <br>
- 0V to +22V; 1 pulse; For the substrate clear; SUBφ
</p>

<p>So to grab a photo from the sensor a cycle like this happens. <br>
1. Clear the transistor substrate
2. Hold vertical rails at 0V to allow intergration (light accumulating in the diodes)
3. Pulse vertical rails to -7V to pull the charge away from the diodes
4. Immediatly begin read out (elaborated lower)
5. For each row of "infomation" in the vertical rows, pulse H1φ and H2φ to shift a pixel out
6. These pixels are amplified and fed into a analouge-digital converter
7. Once a row is done, pulse the Vφ and repeat
8. After everything has been read, pulse SUBφ to clear the substrate of leftover voltage
</p>

![image](https://github.com/user-attachments/assets/6de62e6c-150a-4858-bd3d-43c3439db30c)

<p>Boom, now you have a photo, kinda. It still have to be processed, but I'll burn that bridge when I come to it.</p>

<p><br>So, I need a way to produce **-7V**, **+5V**, **+22V** and **timings** for all the φ signals. (I like using the φ symbol instead of just saying clock, makes me feel like an engineer.)</p>

![image](https://github.com/user-attachments/assets/277d6e25-a385-4edf-b5d0-043498cef1c3)

<p>Bro... What even is this... Even @acon was confused at it 😭</p>


<p>I spent a really long time trying to decipher the datasheet, and then an embarrasingly long time making a custom symbol in Kicad...</p>

> Time taken: **3h 14m**
> - I swear this isn't inflated
> - Tommorow I should be ready to wire up a ton of parts
> - Still very much research stage

## June 2nd

<p>Okay time to wire things up. I'm hoping today I have at least the vertical clock signals all sorted.</p>

<p>I lied I'm doing the substrate clock it's way easier. I just need to boost the voltage from the batteries I'll use (probably 3.3V lipos) into 15V.</p>

<p>I'm also gonna use an IC called CXD3400 cause it handles regulating the voltage for the vertical clocks.</p>

![image](https://github.com/user-attachments/assets/fd19f545-f74b-4b77-9af7-b7b09c95ff25)

<p>Input and output pins for the SUB power on this messy circuit diagram.</p>

![image](https://github.com/user-attachments/assets/a36057cf-3f10-4f07-858e-30112cd54623)

<p>That took WAY too long, but that should be all good for the SUB clock.</p>

<p>Now I need negative voltage for the vertical clocks, so I'll use a charge pump. I choose a LMC7660 because apparently they're low noise and won't interfere with the image sensor. (who am I kidding it was the first thing that came up when I searched for charge pump)</p>

![image](https://github.com/user-attachments/assets/5e0438ee-42b8-4440-ad2e-6a2b46914bdc)

<p>Super easy. I might add a small transistor inbetween the battery and, well everything, so that I can limit power drain to ONLY when the MCU says to go go go. Oh I forgot to mention that I'm using the raspberry pi pico as my controller. It has all the gpio pins I could ever need, and it has the ability to produce the timing frequencies needed with relative ease! Plus I can change the firmware with a usb-c cable which is so much easier than using JTAG or something of the like.</p>

<p>Okay! Now all I have to do is tie in some outputs and fan others and the sensor should have vertical clock signals</p>

![image](https://github.com/user-attachments/assets/9ed15392-95a2-4893-b320-5c3996c3bdee)

<p>So the CXD3400 has 6 clock outputs, while the ICX238Al only uses 5... This is cause the CXD is designed to be used with like any CCD image sensor. This is fine, I just have to tie in the V1A and V1B inputs together to form one V1, and the same for V1A and V1B. I've added a tiny little resistor just as like short prevention, if anyone thinks that's unnecassary tell me but it doesn't hurt. Now the V2 output is a little more complex and annoying. It fans out fine, wiring to both V2 inputs, but I'm ninety percent sure that I'll have to split it into a 2phase signal in code later on.</p>

<p>Also side note, this schematic is so hard to read. If I redid this I'd probably organise it differently but sunk cost phallacy and all.</p>

![image](https://github.com/user-attachments/assets/dd1ef849-3967-4c12-af76-5da4f79ba70c)

<p>Way easier, wiring up all the timing inputs. I'm planning on using PIO in the pico for the timing of... well everything. It can produce signals as fast as the cpu runs (I'm pretty sure, don't quote me on that) and so it can easily handle the timing of the vertical registers and the substrate clear.</p>

<p>Honestly, I think the hardest bit is done now. Both the reset gate clock and the horizontal register clocks can be run on 3.3V, which is what the pico can supply! However I have a bio exam tomorow, and it's time to sleep. Here's what the schematic looks like currently.</p>

![image](https://github.com/user-attachments/assets/d7108a26-c5b8-4392-96a9-f75505e162ba)


> Time taken: **2h 52m**
> - Provided power rails for SUBφ and Vφ
> - Connected driver IC to inputs of sensor
> - Deciphered ancient texts (datasheets)
> - Attached PIO outputs to pulse inputs of driver IC

<p>I was reading over this after I commited my journal change when I realised this sensor is made for black and white cameras. Probably the biggest bruh I've ever let out. Tomorow I'll research if I can swap this out easily for a colour sensor. If not I have to do some weird programming stuff but I really would rather not for two reasons. 1. Adds longer to this project which has already consumed so much of my time just to reading datasheets 2. It would make the whole circuit bending part of this camera (which I haven't even TOUCHED on) a whole lot worse. I'll explain in the morning but for now. Bruh.</p>

## June 3rd

<p>Sigh. Bruh. I was thinking about my software solution to it being a black and white camera, but I realised even then I'd need to change some parts of the hardware in a way that I can't really do. So, I'm gonna try and find a colour sensor that runs on the same wiring.</p>

<p>I've found something similar, it's an ICX453AQ, and it has no public datasheet. This is probably the worst mistake I coulda made. There is a project called the Cam84, which is an astrophotography camera that uses the same sensor, but that's about all the infomation I have on this thing.</p>

<p>ICX285AQ!! It's the exact same sensor but colour! It's also only found in THOUSAND DOLLAR CAMERAS! When I was researching how I would get the sensor, I must have mistook the ICX453 for the ICX285, because the 452 can be found in 70 dollar cameras. I will still try and find a similar sensor so I dont have to change much, but if I can't we're gonna have to use the 453.</p>

<p>Whelp. All similar sensors are only used in industrial solutions. We've gotta use the 453.</p>

![image](https://github.com/wormmeatapple/Warper/blob/main/assets/ICX453AQ.png)

<p>This looks way more complex, but actually it's nearly the same. The vertical registers are now no longer split into 2 phases, instead the horizontal registers are split into 4 phases. It's easier if I show the pins like this.</p>

![image](https://github.com/user-attachments/assets/42f5675c-0471-4238-9a56-48738e3d3142)

<p>These photos are from this website, http://astroccd.org/2015/04/cam84/, and detail a PCB that uses the sensor as an astronomy camera. I want mine to be handheld, but the diagram is a good reference. I do need to remake the footprint and symbol which is slightly annoying.</p>

<p>They also use a different vertical driver, which is wayyy better it comes with a charge pump to produce voltages for the SUB clock. I'm lowkey gonna scrap my current PCB, and remake everything. It shouldn't take long with my new knowledge and awsomeness.</p>

<p>Okay, I'm remaking the schematic from scratch.</p>

<p>I found the CXD1267AN on snapeda! This should make it way easier!</p>

<p>Yeah I don't know why I thought that'd be easy. Kicad is genuinely the stupidest software I've ever used. I think I'd rather just make it by hand.</p>

<p>Oh phew I finally got it to work, that was driving me nuts it kept giving no permission errors when I'm literally the admin.</p>

<p>I stayed up too late and didn't write any journal entries, I'll fix it up tomorow.</p>

> Time taken: **3h**
> - I probably won't count this, it was alot of reading and searching through data

## June 4th

<p>Okay so I have a dilemma. The 453 sensor is awesome, 2 million pixels awesome. It's used in astronomy cameras and handheld DSLRs, but it doesn't have a datasheet and the only information I have on it is from a reverse engineered kiev astronomy camera. There is another sensor, the 415. It has a datsheet, a symbol already in Kicad, I can drive it exactly how it was meant to be driven. But it only has 490k pixels. It's used in security cameras, like cmon. <br> A sane person would choose the easy sensor, make a camera that works and be done with it, but this whole event is to make things that are out of your comfort zone. So I'm gonna use the hard sensor.</p>

![image](https://github.com/user-attachments/assets/e5249594-2f3c-4d79-9c4e-9c07df98b123)

<p>I've learnt what labels are some I'm gonna space this out and make it look clean and nice.</p>

<br>

<p>Okay, I'll wire up the CXD first. I'm using a CXD1267AN, as it has a charge pump inside it to produce the high voltages needed for the SUB clock. I needa supply it with around -7.4V, +5V and +15V. Here's the circuitry I'm using to get these power rails.</p>

![image](https://github.com/user-attachments/assets/cecdb2e8-45ba-4349-82fe-89d6202d507a)

<p>(I haven't added the +15V boost converter yet cause I'm still tryna find one I like.)</p>

<p>Then I just needa wire up the CXD according to the datasheet. While I'm at it I'll wire out the timing from my pico.</p>

![image](https://github.com/user-attachments/assets/6d299d68-67fe-46cd-8b5b-fd0d64e570bc)

<p>Looks pretty good! I have everything bunched together so it's easier to read on the photo, but I'll seperate them once I keep working on the schematic.</p>

<p>Now for the horizontal registers. Again, I have no idea what the actual desired voltage of this thing is, but the Kiev guy uses 6V, so I'll use 6V.</p>

![image](https://github.com/user-attachments/assets/496854dc-7f52-467d-9463-545fbf85cb11)

<p>Super easy, just had to do some tiny calculations to figure out the resistor values. The +6Vs gets wired into some dual mosfet driver, this boosts the 3.3V signal from pico into 6V signals, I'll wire that up now.</p>

<p>The wiring example for the sensor uses a MAX4428 as it's dual mos-driver, but this driver has one inverted output and one regular? I'm not exactly sure why this is needed, but since I don't have the datasheet I really can't check the waveforms needed so I just have to believe in the process. Kicad doesn't have the MAX, but it does have a MIC4428, which has the same amperage, bipolar inversion and mosfet driver nature.</p>

![image](https://github.com/user-attachments/assets/d5e588cb-9bf9-4f97-877a-449c6dd390cf)

<p>H1timing and H2timing just come from another GPIO pin on the pico, and H1 and H2 wire to the sensor like so.</p>

![image](https://github.com/user-attachments/assets/74dbfe95-983a-4cc3-9444-a117ef92d23a)

<p>Okay seems good for today.</p>

> Time taken: **2h**
> - Whole bunch of wiring
> - Much easier now that I know what I'm doing

## June 8th

<p>I'm gonna continue with the PCB today, add the 15V booster, start work on output stuff like that.</p>

<p>For the 15V booster, I'm gonna use the TPS65130 which can produce both positive and negative voltage and the ratings I want. It's also the IC used for 15V and -8V in the example circuit so I know it works.</p>

<p>I was so dialed I forgot to make little updates... whoops. Anyway, here's the +15V and new -8V regulator</p>

![image](https://github.com/user-attachments/assets/f053ee2f-df33-46a7-8a97-e121b9a705ab)

<p>Previously I was trying to seperate signal and power ground, but someone on #electronics said I should be fine to just have a large ground plane and distance my noisier packages so I'm going to do that. Unfortunatly that's all I have time for today.</p>

> Time taken: **54m**
> - Wired up a pretty complex power regulator
> - Changed things from seperate sig/pwr gnd to one gnd plane

![image](https://github.com/user-attachments/assets/7042b705-f7e8-4fa2-ad6a-f9c6108be777)

<p>Finally finished wiring up the sensor. Really quickly added in the RG signal (just from the pico) and the power supplies. Now I needa turn the raw output of this sensor into digital images.</p>

<p>First things first, the output needs to be amplified. Right now it's a couple millivolts worth of signal, we needa boost that</p>

![image](https://github.com/user-attachments/assets/8f8c6760-6049-4330-953e-f303013d2194)

<p>Ehhhhh?? I think this works? I'm not sure. Anyway, I spent a while researching the wiring for my output, but it's late so I'm gonna write all that up in the morning.</p>

> Time taken: **56m**
> - Wired up an op-amp for the first time
> - Ton of research on outputs of CCDs

## June 22nd
<p>Woah it's been a while. Been working on other projects (join #flag!!!) but I'm back to highway. Now I finally have to tackle the output for this thing. First I need a CDS circuit, or correlated double sampling circuit.</p>

<p>A CDS works by letting two caps fill up with voltage at different times, and then subtracting those voltages. Each cap is getting fed the CCD output, but a switch is between them that only lets them flow during non-overlapping clock cycles. We're gonna be subtracting the signal voltage by the reset voltage to eliminate any noise generated by the sensor just existing.</p>

![image](https://github.com/user-attachments/assets/bb4b39a1-37db-4b87-882c-ef966e7ba5e6)

<p>Should be good? It's hard to find a clear example circuit. But this should store the data from the CCD after a RESET and a H2 clock (which is the clock that moves voltage from the last horizontal buffer to the output buffer), and then subtract the two.</p>

<br>

<p>I'm gonna add an amplifier with an adjustable potentiometer gain. This'll be good for tuning the signal voltage + for when it's time to circuit bend (nearly forgot that's the whole point of this!)</p>

![image](https://github.com/user-attachments/assets/177bb313-7ab5-42a5-b3de-b5d93e200b66)

<p>I'm gonna add a ton of these pots. and switches, but I want them on the external case, so I'll add little holes to wire them to the board.</p>

<br>

<p>Now to convert analogue to digital, with an ADC! (you'll never guess what it stands for)</p>

<p>I'm gonna use the ADS8332, and all that's gonna do is turn the now amplified, filtered CCD signal and turn it into a SPI signal that the pico can read. Butttt that's all from me for today</p>

> Time taken: **34 mins** cause im stupid
> Added some noise filtering
> Ability to increase the amplitude of our CCD signal






