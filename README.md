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
I had a couple Wemos D1 clone parts that I had been playing with, so I decided to use that as my processing base. 

**Software Considerations**

The basic FSBrowser code makes the ESP8266 into a small webserver with access to the file system on the ESP8266. It provides capability to view the files and to edit them if desired. This means that you can access the files on the ESP8266 with your favorite browser and a knowledge of the IP address that is acquired by the ESP8266 from your WiFI router when the system is initialized. I have not chosen to run this with a web manager module, so the SSID and key are hard coded in the sketch. You will need to build the code in the Arduino IDE with your SSID and key. I also opted to use the SPIFFS file system for this implementation. These can be changed if you are of a mind to make it more robust.
ZX80 (and ZX81) data input and output
The ZX80 with the 4K ROM has a LOAD and SAVE command to use the cassette tape output and input. There are no file name parameters, so the data that is output and input are bit images of the data in memory. With an 8K ROM installed, the file structure has the capability for names. Here is a link that has a good summary of the cassette signals: https://problemkaputt.de/zxdocs.htm. Here is the section from that web site for file structure and signaling:

_________________________________
ZX81 Cassette File Structure

  x seconds    your voice, saying "filename" (optional)
  
  x seconds    video noise
  
  5 seconds    silence (only some clock cycles required for ZX8)
  
  1-127 bytes  filename (bit7 set in last char)
  
  LEN bytes    data, loaded to address 4009h, LEN=(4014h)-4009h.
  
  1 pulse      video retrace signal (only if display was enabled)
  
  x seconds    silence / video noise
  
The data field contains the system area, the basic program, the video memory, and VARS area.



ZX80 Cassette File Structure

  x seconds    your voice, saying "filename" (optional)
  
  x seconds    video noise
  
  5 seconds    silence (at least 0.5 seconds REQUIRED for ZX80)
  
  LEN bytes    data, loaded to address 4000h, LEN=(400Ah)-4000h.
  
  x seconds    silence / video noise
  
ZX80 files do not have filenames, and video memory is not included in the file.


File End

For both ZX80 and ZX81 the fileend is calculated as shown above. In either case, the last byte of a (clean) file should be 80h (ie. the last byte of the VARS area), not followed by any further signals except eventually video noise.

Bits and Bytes

Each byte consists of 8 bits (MSB first) without any start and stop bits, directly followed by the next byte. A "0" bit consists of four high pulses, a "1" bit of nine pulses, either one followed by a silence period.

  0:  /"\"/"\"/"\"/"\"________
  
  1:  /\/\/\/\/\/\/\/\/\________
  
Each pulse is split into a 150us High period, and 150us Low period. The duration of the silence between each bit is 1300us. The baud rate is thus 400 bps (for a "0" filled area) downto 250 bps (for a "1" filled area). Average medium transfer rate is approx. 307 bps (38 bytes/sec) for files that contain 50% of "0" and "1" bits each.

_________________________________

**ESP8266 code**

The ESP8266 will need to read a file from the file system, returning each byte (with low address bytes first), and then the data will be shifted out bit wise starting with the most significant bit of the byte. The output from the ESP8266 will be as described (4-pulses 150us high, 150us low with 1300us after last low period for a 0 bit, 9-pulses 150us high, 150us low with 1300us after last low period for a 1 bit. 
I added a file read and data send routine to the basic FSBrowser code, and also modified the file read routine so that the byte send function is called for each byte read from the file. This code is nothing fancy, but it does work nicely. Please see the attached sketch for details on the code structure. 
I also had to make a few very basic modifications to the JavaScript code to accept the various ZX80 file types. This is in the edit.htm file that is loaded into the ESP8266 data space. 
Note that FSBrowser uses the Ace.js text editor in edit.htm, so it requires internet access from the router that assigns the IP address to be fully capable. Without internet access it will default to a very basic text mode. 

