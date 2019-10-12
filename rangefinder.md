# Wemos D1 Range Finder
This project involved communicating between 2 micro controllers - one a range finder, the other a stoplight.  The stoplight's project can be found [here](https://craightonh.github.io/school-blog/mcstoplight).  Nothing changes with that project, so the documentation in this tutorial will only cover techniques used to build the range finder.

This tutorial is not intended to be a codebase for the project, but a high-level overview of its requirements.  For the complete codebase of my project, visit my [GitHub repo](https://github.com/CraightonH/wemos-range-finder)

## Objectives
* Build an IoT sensor using GPIO pins for inputs
* Establish a machine-to-machine (M2M) communication protocol
* Design an IoT interaction between a sensor and an actuator.

## Materials
The materials used in this lab were:
* Laptop
* 1x Wemos D1 mini microcontroller
* 1x HC-SR04 ultrasonic speaker
* 1x breadboard
* 4x wires
* Fritzing (for breadboard diagram)

## References
* [Coding with UltraSonic Speaker](http://www.circuitbasics.com/how-to-set-up-an-ultrasonic-range-finder-on-an-arduino/) - This site contains a good explanation and example of how to use the UltraSonic speaker.  
* [HTTP Web Request with Arduino](https://github.com/arduino-libraries/ArduinoHttpClient/blob/master/examples/SimpleGet/SimpleGet.ino) - This is the Repo for an HTTPClient library for the Arduino.  It contains an example of how to use it along with the code for the library itself.

## Procedures
### Setup Wemos D1 Mini Development Environment
Arduino has an IDE for development with their chipsets.  The following steps document how to setup this development environment:
1. Connect your micro controller to your computer via a mini USB to USB cable
2. Assuming Windows OS, open Device Manager and expand Serial, then right click on the item and select Update Driver.  After installation the item’s name should change to CH340 USB to serial controller
3. Add the Wemos D1 mini’s library to the Arduino IDE.  Select File > Preferences and enter `http://arduino.esp8266.com/stable/package_esp8266com_index.json` into the Additional Boards Manager URLs field.
4. Add the “LOLIN Wemos D1 R2 & mini” board to your project.  Click on Tools > Board and find the aforementioned board.
5. Add the ArduinoHttpClient library.  Click on Tools > Manage Libraries > Search for "ArduinoHttpClient" and click install.

### Implement Range Finding Logic
#### Calculating Distance
Because we're building a range finder, we need to calculate the distance between our UltraSonic speaker and the object in front of it.  The formula for that calculation is:
```
D: distance
S: speed
T: time
D = ST
```
But what's the speed?  In our case, the speed we will use is the speed of sound through air since we want to calculate the distance the sound waves traveled.  

#### Calculating Speed of Sound
As explained in the [UltraSonic Speaker tutorial](http://www.circuitbasics.com/how-to-set-up-an-ultrasonic-range-finder-on-an-arduino/), speed travels at a constant `331.4 m/s` at 0C and 0% humidity.  The formula to calculate the speed of sound is
```
C: Calculated speed of sound
S: Constant speed of sound @ 0C and 0% humidity
T: Temperature in Celsius
H: % Humidity
C = S + (.606T) + (.0124H)
```
Because this project doesn't include a temperature and humidity sensor, we'll just use reasonable numbers for each to estimate:
```
C: Calculated speed of sound
S: 331.4
T: 20
H: 50
C = 331.4 + (.606*20) + (.0124*50) = 344.02 m/s
```

#### Range Finding
Now our program needs to use the distance calculation to determine how far away an object is.  Remember, these audio waves are traveling away from the speaker AND then back.  But we only want the distance they traveled one way, otherwise we'll think the object is twice as far away as it actually is!  So our final calculation will be:
```
D: distance
S: speed
T: time
D = S(T/2)
```

### Design the Circuit
Assuming the Wemos D1 has male headers soldered into its pin slots, plug it into the breadboard.  Then use wires to connect the micro controller to the ultrasonic speaker as depicted in the following diagram:
![Breadboard Wiring](https://github.com/CraightonH/school-blog/blob/master/RangeFinderDiagram.png?raw=true)

## Thought Questions
1.	**What are some key differences between developing this lab on a Raspberry Pi, and developing on Arduino?**
The biggest difference is with Arduino, you’re locked into a specific language – C – while with the Pi, you can choose pretty much any language you want.  Additionally, because the Pi has an OS on it, you can start and stop your program at will, while with the Arduino you plug it in and it loops through your program until it no longer receives power.  
2.	**What are the strengths and trade-offs of each of these platforms?** 
The strength of the Pi is it’s versatility in implementing the software – if a language can run on an ARM processor, you can use it and implement your project in a familiar-to-you fashion.  Tradeoffs of the Pi are its size and cost.  At about $40, it’s an expensive wifi-enabler and it would be clunky to place near the item you want wifi enabled in some cases.  
The microcontroller excels in size and cost.  It’s extremely cheap and easy to attach to a device to make it wifi-enabled.  The big downside is you must program the microcontroller in C and implementing the code isn’t as familiar a process since the code doesn’t run on top of an OS.
3.	**How familiar were you with the Arduino platform prior to this lab?** 
I had never written code for an Arduino before this lab.  It was fun and not as difficult as I first expected thanks to all the libraries available for it.
4.	**What was the biggest challenge you overcame in this lab?**
The hardest part of the lab was implementing the timer for the cycle function.  In the raspberry pi I implemented a while loop in a background thread, but you don’t get multiple threads on this platform.  So it took awhile for me to wrap my mind around making my own timer with the D1’s built-in clock while not blocking the only thread available to the program.
5.	**Please estimate the total time you spent on this lab and report.**
Coding and wiring took me about 3-4 hours. This report took me about 1.5 hours.

## Certification of Work
I certify that the solution presented in this lab represents my own work.

Craighton Hancock

## Appendix
### Appendix A - Code
All of the code for this project can be found at my [GitHub repo](https://github.com/CraightonH/wemos-stoplight).