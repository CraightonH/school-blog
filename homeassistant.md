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
3. Search in the official store for "Mosquitto Broker"
4. Click the Install button.
5. While it installs, review the instructions on how to configure it.
6. Once installed, the bottom of the page will show the MQTT server's config file.  For now, turn SSL settings to false and anonymous logins to false.
7. Click on Configuration in the Sidebar and then Users.  Then click the plus icon and create a new user to be used for authentication from the microcontrollers.  To increase security, create different users for each device.

### Connect Microcontrollers
In each device, only 2 changes need to be made:
1. Change the IP address of the MQTT server to be the same IP as the Home Assitant server.
2. Add the username and password for your device to the connection:

```
client.connect((char*) devID.c_str(), "[user]", "[password]")
```
where `[user]` is the username and `[password]` is the password created in the last step of the previous section.  Once the microcontrollers are turned on, they should begin to communicate with the MQTT broker on the Home Assistant.  

### Create Home Assistant Entities
To see the data produced by your devices, you first need to create entities for them.  The [Home Assistant Documentation](https://www.home-assistant.io/docs/mqtt/discovery/) is helpful for understanding how to set up the entities.  They provide examples of how to setup entities manually, but all that is necessary is to publish a message to a specific MQTT topic with the details of the device.  Your microcontroller can setup its own entity!

Because our microcontrollers have a function for reconnecting to the MQTT server which runs on startup and at any point if the connection is lost, we can add commands in this function to also setup the entity.  Add the following function to your microcontrollers and call this function in the reconnect function, replacing `[Text]` with values that make sense for the specific microcontroller:

```
void sendHassDeviceConfig() {
  client.publish("homeassistant/sensor/garage/[Type]/config", "{\"name\": \"[Name]\", \"state_topic\": \"/[Topic]\"}");
}
```

### Add Sensors to Dashboard
Now that entities are created, we can visualize their data!
1. Click on Overview in the sidebar
2. Click on the Context menu on the top right of the page and then click Configure UI
4. Click the Plus button
5. Click Sensor and open the Entity dropdown
6. Click on one of the microcontroller entities and click Save
7. Repeat steps 4-6 for the other 2 microcontrollers

Now you should see sensors that are displaying data from your microcontrollers.

### Add Automations


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
