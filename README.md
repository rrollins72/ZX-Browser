# ZX-Browser
ZX-Browser for ZX80 program loading based on FSbrowser for 8266
**Problems with loading programs**
I kept having a LOT of problems with loading software. I had just a couple .wav files to try. Most programs are saved as bit images in .o or .p files. I found a program to convert the file to audio, or to create a .wav file from the image file, for output from the PC computer to the ZX80 cassette input. The output from my simple USB audio adapter plugged into my test laptop was not nearly enough to drive the cassette input. Looking at the circuit and tracing components on the ZX80 told me why. Not sufficient drive to the LS365 (IC10) to receive any data. The LS365 needs at least 2.0V for a guaranteed high input with a 5V supply. I am getting less than 1.5V (see image). This is too low. I wondered about that.
![image](https://user-images.githubusercontent.com/76188172/128247404-e8e4868f-4c75-41ee-93e4-f492407f0b37.png)
**Output of USB audio output as seen at R1**
The drive to this circuit would need to develop at least 3V across the input resistor (180Ω ) to give an input large enough for the LS365 (IC10) to receive reliably. I needed at a minimum a small power amplifier to hook to the output of the laptop or USB adapter or another approach. 
**File considerations**
I also wanted to have a simple way to access files that I might want to load into the ZX80. Storing them on the laptop is easy, but they have to be output to the analog output with a special program that converts the digital image to analog, or to a .wav file (which can be very large since the data rate is so slow). So having a small dedicated storage capability with data conversion would be very nice.
Having played with the ESP8266 a little, and having played with the FSBrowser example (see https://github.com/esp8266/Arduino/tree/master/libraries/ESP8266WebServer/examples/FSBrowser). I decided I would use this as a basis for what I wanted to do next.
**What I want**
1.	Store all the ZX80  programs on the ESP8266 file space (they are very small, and I don’t have a lot)
a.	There is more than 3Mbyte of space available on ESP8266 module I am using. This could easily be sufficient storage for a couple hundred ZX80 programs.
b.	Programs can be stored in native binary .o, .80, .p, .81 or .p81 format
2.	Program the ESP8266 to create an ideal output that duplicated the cassette output
3.	Drive the cassette input to the ZX80 with a simple FET switch and a convenient power source to give sufficient drive voltage
I had a couple Wemos D1 clone parts that I had been playing with, so I decided to use that as my processing base. (Search Amazon for ‘d1 mini esp8266’ for several vendors).
