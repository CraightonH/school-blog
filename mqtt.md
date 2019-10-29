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

### Connect Arduino to MQTT Server
#### Install PubSubClient Library
Open the Arduino IDE and do the following:
1. Click on Tools > Manage Libraries
2. Search for "PubSubClient"
3. Find `PubSubClient by Nick O'Leary` in the list and click Install
4. In your sketch, include the following lines:
```
#include <ESP8266WiFi.h>
#include <ESP8266WiFiMulti.h>
#include <PubSubClient.h>
WiFiClient wifiClient;
PubSubClient client([IP of mosquitto server], 1883, wifiClient);
```
Now you can use the `client` object to interact with the MQTT server.

#### 
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