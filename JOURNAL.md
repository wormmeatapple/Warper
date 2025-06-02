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


