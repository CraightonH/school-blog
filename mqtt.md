# MQTT Driven Garage Parking System
This project involved communicating between 3 micro controllers - one a range finder, another a stoplight, and the last a door sensor.  The stoplight's project that is rewritten to work with an MQTT bus can be found [here](https://github.com/CraightonH/wemos-mqtt-stoplight).  The range finder with similar changes can be found [here](https://github.com/CraightonH/wemos-mqtt-range-finder).  And finally, the code for the door sensor can be found [here](https://github.com/CraightonH/wemos-mqtt-door-sensor).

This tutorial is not intended to be a codebase for the project, but a high-level overview of its requirements.  For the complete codebase of my project, visit the repos linked above.

## Objectives
* Implement an Event Hub for publish/subscribe notifications between devices
* Develop a communications protocol for devices across the event bus
* Establish more complex conditions for the actuator involving multiple sensors 

## Materials
The materials used in this lab were:
* Laptop
* 3x Wemos D1 mini microcontroller
* 1x PCB Stoplight
* 1x Magnetic Contact Switch
* 1x HC-SR04 ultrasonic speaker
* 3x breadboard
* 4x wires
* Fritzing (for breadboard diagrams)
* LucidChart (for state diagram)

## References
* [Intro to MQTT and Arduino](https://m2mio.tumblr.com/post/30048662088/a-simple-example-arduino-mqtt-m2mio) - This site contains a simple explanation and example of how to use MQTT with Arduino.  
* [PubSubClient example](https://gist.github.com/igrr/7f7e7973366fc01d6393) - This is a complete code example of how to use the PubSubClient.
* [PubSubClient Repo](https://github.com/knolleary/pubsubclient/blob/master/examples/mqtt_esp8266/mqtt_esp8266.ino) - This is the repo for the PubSubClient which contains more instructional information and examples.

## Procedures
### Install MQTT server
You can easily install mosquitto server on Ubuntu:
1. Add the mosquitto apt repo:
`sudo apt-add-repository ppa:mosquitto-dev/mosquitto-ppa`
2. Update the apt cache:
`sudo apt-get update`
3. Install mosquitto server:
`sudo apt-get install mosquitto`
4. Run mosquitto in the background:
`mosquitto -d`

Along with the server, two useful tools were installed that help you test MQTT right away.  Open two more terminals and in one type:
```
mosquitto_sub -t /# -v
```
That listens to all sub topics of `/` and prints verbose output.  In the second terminal type:
```
mosquitto_pub -t /test -m "testing"
```
This will print output in the `mosquitto_sub` terminal that looks like:
```
/test testing
```
This shows that the message `testing` was received in the `/test` topic.  MQTT is as simple as that!

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