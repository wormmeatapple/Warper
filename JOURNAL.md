![image](https://github.com/user-attachments/assets/56f343f7-dc10-4703-97e1-2791a81e0c27)## May 30th

<p>First things first is to start researching. My research into circuit bending led to an answer I really didn't like, no one knows how to predict it. You just mix, modify and disrupt data signals in the hopes it makes something cool. This is not ideal but it is fineee, because I know (slightly) how a camera sensor works.</p>

<p>A camera sensor (at least the one I plan to use, a CCD) is really just an array of photodiodes that all output a little voltage depending on how much light they received. This voltage is moved into a buffer and then moved into a seperate buffer that handles one pixel at a time. That little pixel is amplified, and then processed into digital format and displayed. All of this is handled by little signal pulses, and my thinking is that if I can incorporate pentiometers with tiny ranges in resistance I can mess up some of the thinking and boom, bends formed. I'll also add ways to short two data lines together, just for the fun of it.</p>

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

