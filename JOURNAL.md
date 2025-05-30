## May 30th

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
