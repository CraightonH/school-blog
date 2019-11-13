# Home Assistant
This project was built on the last by connecting a web front end to the 3 microcontrollers.  This front end also ran a backend mqtt server to which it and the 3 microcontrollers connected.  Because of this, the UI was able to display data gathered from the microcontrollers and perform automations based on that data.

## Objectives
* Design a system that facilitates automation with the microcontrollers 

## Materials
The materials used in this lab were:
* Laptop
* 3x Wemos D1 mini microcontroller
* 1x PCB Stoplight
* 1x Magnetic Contact Switch
* 1x HC-SR04 ultrasonic speaker
* 3x breadboard
* 4x wires
* Raspberry Pi

## References
* [Home Assistant Installation Instructions](https://www.home-assistant.io/getting-started/) - This contains the documentation on how to install the HassIO image onto a raspberry pi.
* [Home Assistant MQTT Discovery Docs](https://www.home-assistant.io/docs/mqtt/discovery/) - This documents how to create home assistant entities for MQTT topics.

## Procedures
### Install Home Assistant
Follow the [Home Assistant Installation Instructions](https://www.home-assistant.io/getting-started/) document.
If you intend to use WiFi with your raspberry pi:
1. Create a folder named `CONFIG` in the root directory of the filesystem on the SD card
2. Create a folder named `network` in the `CONFIG` folder
3. Create a file named `my-network` in the `network` folder
4. Copy the following template into the `my-network` file and change `MY_SSID` and `MY_WLAN_SECURITY_KEY`:
```
[connection]
id=my-network
uuid=72111c67-4a5d-4d5c-925e-f8ee26efb3c3
type=802-11-wireless

[802-11-wireless]
mode=infrastructure
ssid=MY_SSID
# Uncomment below if your SSID is not broadcasted
#hidden=true

[802-11-wireless-security]
auth-alg=open
key-mgmt=wpa-psk
psk=MY_WLAN_SECRET_KEY

[ipv4]
method=auto

[ipv6]
addr-gen-mode=stable-privacy
method=auto
```
5. Install the SD card into the pi and turn it on
6. Login with `root` (no password)
7. Type `login`
8. Type `ip addr` to get the IP of the pi and then on a separate device on the same network, navigate in a browser to `[ip addr]:8123`
9. You should see the Home Assistant startup screen and after some time you will be prompted to create your user account.

### Install MQTT Server
1. Once logged in, open the sidebar and click on Hass.io.
2. Click on "Add-on Store"
3. Search in the official store for "MQTT Broker"
4. Click the Install button.
5. While it installs, review the instructions on how to configure it.
6. Once installed, the bottom of the page will show the MQTT server's config file.  For now, turn SSL settings to false and anonymous logins to false.

### Implement Stoplight Logic
The following is a state diagram of the logic that needs to be performed:
![State Diagram](https://www.lucidchart.com/publicSegments/view/d6d9ff62-378e-42ae-a006-b06407a8869c/image.png)

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

![Breadboard Wiring](https://github.com/CraightonH/school-blog/blob/master/MagneticSensorDiagram.png?raw=true)

Note, the magnetic sensor is a simple device with two wires.  It doesn't matter which wire is put into ground or not.  The diagram may look confusing because there is no device and that's because I was unable to find the sensor in Fritzing.  Suffice it to say, you only need to put one of the wires in ground and one in another pin that can be read.


## Thought Questions
1.	**How does the communication between devices change with the shift to using an event hub? How does this facilitate greater scalability? What things did you do to learn to use MQTT?**

Using an event hub greatly increases the scalability of this system because you don't have to create a web API on each device and code each device to know IP addresses of all other devices and the endpoints each device exposes.  Instead, all the devices can subscribe to topics on the event hub and any future device can interact with the actuator by simply sending data that the actuator would expect in that topic.  I got my feet wet with MQTT by installing it on my laptop and using the `mosquitto_pub` and `mosquitto_sub` utilities that are installed with the server.
2.	**What are strengths and weaknesses of the direct communication between the sensor and actuator? What are strengths and weaknesses of the event hub? Which do you feel is better for this application?**

The strength of direct communication is that data is exchanged only when necessary.  An actuator might only request data from the sensor at some interval and that's the only data sent over the network, while with an event hub, now the sensor is pushing data into the hub whether the actuator needs it now or not which increases network traffic.  The weakness of direct communication is that it's not manageable at scale.  Once you work with 3 or more devices, it becomes cumbersome to refactor code to account for communication between all the devices.  
3.	**What was the biggest challenge you overcame in this lab?**

The biggest challenge I had was choosing how to organize my topics.  Once I got the hang of MQTT concepts, my mind started thinking about how I could dynamically set config on the devices in something like a `/[devID]/config` topic which could change values of any global variables, like the distance thresholds for changing the stoplight color, for example.  I also have an elasticsearch cluster running at home that monitors my public websites and services, so I was excited to setup `/log/[info|debug|warning|error]` topics that I could have some other subscriber process send to my ELK cluster to gain better visibility on my devices and get alerted when they fail.  I think about 6-7 hours of my total time was spent testing how to make all of that happen, none of which made it into my codebase because I saw that it would take far longer to set all that up than I wanted to spend for this project.  But I am excited to implement those concepts for my personal projects.
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
### Appendix C - Door Sensor Code
All of the code for this project can be found at my [GitHub repo](https://github.com/CraightonH/wemos-mqtt-door-sensor).
