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

### Implement Stoplight Logic
Now that we're using the MQTT server to drive our devices instead of web servers, we need to put logic into the stoplight so it can choose which light turns on and when.  Our logic operates as follows:
1. Only update the light if the garage door is open, otherwise turn the light off.
2. Change the light color based on the same criteria as the previous projects.
With those criteria, our main logic can be broken down into the following function:
```
void handleDistance(int distance) {
  if (DOOR_OPEN) {
    if (distance <= 10) {
      if (!FLASH) {
        startTimer();
      }
      FLASH = true;
    }
    if (distance > 10 && distance <= 20) {
      redOn();
    }
    if (distance > 20 && distance <= 30) {
      yellowOn();  
    }
    if (distance > 30 && distance <=50) {
      greenOn();
    }
    if (distance > 50) {
      off();
    }
  } else {
    off();
  }
}
```

#### Setup MQTT topic listeners
But how do we get the variables `distance` and `DOOR_OPEN`?  We get them by listening to different topics.
```
const char* parkDistanceAllTopic = "/garage/park/distance/#";
const char* parkDistanceTopic = "/garage/park/distance";
const char* parkDistancePTopic = "/garage/park/distance/p";
const char* garageDoorAllTopic = "/garage/door/#";
const char* garageDoorTopic = "/garage/door";
const char* garageDoorPTopic = "/garage/door/p";
const char* stoplightStatusTopic = "/garage/stoplight/status";
const char* stoplightStatusPTopic = "/garage/stoplight/status/p";
const char* stoplightActionTopic = "/garage/stoplight/action";
```
Notice that in all cases, there is an extra topic the a `/p` appended to it.  This allows us to "p"eriodically send the state of the device.  In case the stoplight microcontroller ever resets, it can be updated to the correct state within time period "p".
With the above topics defined, we can listen to certain ones with the following:
```
client.subscribe(stoplightActionTopic);
client.subscribe(parkDistanceAllTopic);
client.subscribe(garageDoorAllTopic);
```
NOTE: `stoplightActionTopic` isn't necessary, but it is useful for testing the stoplight's functionality without all of the other required data in the logic.

Finally, we need a callback function that gets called when a message is received:
```
char message_buff[100];
void callback(char* topic, byte* payload, unsigned int length) {
  int i = 0;
  for(i = 0; i < length; i++) {
    message_buff[i] = payload[i];
  }
  message_buff[i] = '\0';
  String message = String(message_buff);
  String strTopic = String(topic);
  if (strTopic == parkDistanceTopic || strTopic == parkDistancePTopic) {
    handleDistance(message.toInt());
  } else if (strTopic == garageDoorTopic || strTopic == garageDoorPTopic) {
    updateGarageDoor(message); # sets DOOR_OPEN
  } else if (strTopic == stoplightActionTopic) { # used for testing without extra data
    if (message == "red") {
      redOn();  
    } else if (message == "yellow") {
      yellowOn();  
    } else if (message == "green") {
      greenOn();  
    } else if (message == "off") {
      off();
    } else if (message == "cycle") {
      cycle();
    }
  }
}
```
With these basic building blocks in place, your stoplight can now respond to updates on the MQTT server.

### Setup Range Finder
The range finder is much simpler since all the logic is in the stoplight.  This device simply reads data from the sensor and sends it into the MQTT topic.  The logic for calculating the distance remains the same as the last project.  What remains is simply publishing the data to the topic:
```
void pubDistance(String distance) {
  if (millis() > timer + 1000) {
    timer = millis();
    client.publish(parkDistanceTopic, distance.c_str());
  }
}

void loop(void) {
  long duration, distance;
  digitalWrite(TRIGPIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIGPIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIGPIN, LOW);
  duration = pulseIn(ECHOPIN, HIGH);
  distance = (duration/2) * SPEED_OF_SOUND;
  pubDistance(String(distance));
}
```
Notice that we use a timer to prevent constantly publishing the data.  Every second, data will be published into the `parkDistanceTopic`.

### Setup Door Sensor
This is the simplest sensor of all to setup and it's the new addition to this project.  In a nutshell:
1. Send the door state periodically.
2. Send the door state when it changes.
```
void pubDoorStatePeriodic(bool doorClosed) {
  if (millis() > timer + 5000) {
    timer = millis();
    pubDoorState(doorPTopic, doorClosed);
  }
}

void pubDoorState(const char* topic, bool doorClosed) {
    String door = "open";
    if (!doorClosed) {
      door = "closed";  
    }
    client.publish(topic, door.c_str());
}

void loop(void) {
  if (digitalRead(SWITCHPIN) == LOW) {
    doorClosed = false;
  } else {
    doorClosed = true;  
  }
  if (prevDoorClosed != doorClosed) {
    prevDoorClosed = doorClosed;
    pubDoorState(doorTopic, doorClosed);  
  }
  pubDoorStatePeriodic(doorClosed);
}
```
When the sensor is low, the door is open.  When the sensor is high, the door is closed.
With all this data now being sent to the MQTT server, the stoplight works completely autonomously.

### Design the Circuits
#### Stoplight
The circuit for the stoplight doesn't change.  Refer to the documentation on setting it up [here](https://craightonh.github.io/school-blog/mcstoplight).

#### Range Finder
Like the stoplight, this circuit doesn't change.  Refer to its documentation [here](https://craightonh.github.io/school-blog/rangefinder).

#### Door Sensor
This circuit is extremely simple.  Wire it according to the following diagram:


## Thought Questions
1.	**How does the communication between devices change with the shift to using an event hub? How does this facilitate greater scalability? What things did you do to learn to use MQTT?**

2.	**What are strengths and weaknesses of the direct communication between the sensor and actuator? What are strengths and weaknesses of the event hub? Which do you feel is better for this application?**

3.	**What was the biggest challenge you overcame in this lab?**

4.	**Please estimate the total time you spent on this lab and report.**
Coding and wiring took me about 8-10 hours. This report took me about 3 hours.

## Certification of Work
I certify that the solution presented in this lab represents my own work.

Craighton Hancock

## Appendix
### Appendix A - Stoplight Code
All of the code for this project can be found at my [GitHub repo](https://github.com/CraightonH/wemos-mqtt-stoplight).
### Appendix B - Range Finder Code
All of the code for this project can be found at my [GitHub repo](https://github.com/CraightonH/wemos-mqtt-range-finder).
### Appendix B - Door Sensor Code
All of the code for this project can be found at my [GitHub repo](https://github.com/CraightonH/wemos-mqtt-door-sensor).
