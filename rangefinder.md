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
5. Add the ArduinoHttpClient library.  Click on Tools > Manage Libraries > Search for `ArduinoHttpClient` and click install.

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
C = 331.4 + (.606*20) + (.0124*50) = 344.02 m/s ~ 344 m/s
```

#### Range Finding
Now our program needs to use the distance calculation to determine how far away an object is.  Remember, these audio waves are traveling away from the speaker AND then back.  But we only want the distance they traveled one way, otherwise we'll think the object is twice as far away as it actually is!  So our final calculation will be:
```
D: distance
S: speed
T: time
D = S(T/2)
```

#### Flashing the Light
If you read through the stoplight tutorial, you'd probably notice that there isn't a flash endpoint.  After thinking through the issue, I decided not to change the code for that project - the stoplight simply provides an API to turn lights on and off.  The client is in charge of implementing additional logic.  Therefore, the sensor will decide when and how to flash the light.  Here's a snippet of code since the code describes the process better than words:
```
if (distance <= 10) {
    if (FLASH_RED) {
      FLASH_RED = false;
      makeRequest("red");
    } else {
      FLASH_RED = true;
      makeRequest("off");
    }
  }
```
By simply maintaining a variable of the what the next operation should be, we can tell the stoplight to flash or not.

### Design the Circuit
Assuming the Wemos D1 has male headers soldered into its pin slots, plug it into the breadboard.  Then use wires to connect the micro controller to the ultrasonic speaker as depicted in the following diagram:

![Breadboard Wiring](https://github.com/CraightonH/school-blog/blob/master/RangeFinderDiagram.png?raw=true)

## Thought Questions
1.	**Think of the interaction between your devices. How well would this scale to multiple devices? Is it easy to add another sensor? Another actuator?**
This would not scale well at all.  While this project didn't include an API for the range finder, I would likely build one if I were to set this up at home.  The API could then allow remote control of the different distances without having to reflash the device.  Adding a 3rd device, with it's own API, could mean that all other devices need to be updated to see it's API to allow for interaction with it.  It is not practical to maintain an API for each device and update each device when another is added to the system.
2.	**What are strengths and weaknesses of the tennis-ball-on-a-string system that Don had originally? What are strengths and weaknesses of the IoT system that he developed? What enhancements would you suggest?** 
The strength of the tennis ball is that it's 1) cheap, 2) effective, and 3) requires very little to no maintenance.  The strength of the IoT system is that it's 1) cheap, 2) effective, and 3) easily configurable (assuming an API that allows for changing distance configurations).  It's weakness is that if Don invents a new interaction with this existing system, he has to update all devices to account for this new interaction.  Additionally, these cheap IoT devices are more likely to break than a tennis ball which makes this parking system fragile - only one of the 4 devices needs to break and you no longer have a parking system.  
3.	**What was the biggest challenge you overcame in this lab?**
I had 1 major challenge during the lab which was my microcontroller kept throwing errors and failing.  At first I suspected my code was written poorly, but after about 2 hours, I determined that it was insufficient power to the microcontroller.  Sure enough, once I plugged the microcontroller into a wall outlet instead of a USB port on my computer, everything worked like a charm.
5.	**Please estimate the total time you spent on this lab and report.**
Coding and wiring took me about 4-5 hours. This report took me about 1.5 hours.

## Certification of Work
I certify that the solution presented in this lab represents my own work.

Craighton Hancock

## Appendix
### Appendix A - Code
All of the code for this project can be found at my [GitHub repo](https://github.com/CraightonH/wemos-range-finder).