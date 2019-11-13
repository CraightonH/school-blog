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
The following diagram shows a high level overview of what we are trying to achieve:
![HASS System Diagram](https://www.lucidchart.com/publicSegments/view/69190d3d-8310-4424-b62a-e69e10621620/image.png)

We'll install an MQTT broker and Home Assistant on the raspberry pi and integrate Home Assistant with IFTTT to connect to almost any external service we choose.

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

### Build Automations
I'll include a single automation because the sky's the limit here.  We'll integrate our home assistant with IFTTT, a well known connection platform.  To do this, we'll need another tool so we can edit files on Home Assistant.

#### Add Configurator
1. Click on Hass.io in the sidebar
2. Click on "Add-on Store"
3. Search in the official store for "Configurator"
4. Once installed, click the toggle for "Show in sidebar"

#### Setup IFTTT
1. Create IFTTT account if not already done
2. Navigate to [Webhooks](https://ifttt.com/maker_webhooks)
3. Click the Connect button
4. Navigate to [Webhook Settings](https://ifttt.com/maker_webhooks/settings)
5. Note the value after the last `/` (that's your IFTTT webhook key)
6. Click the Explore button and then click "Create your own applet"
7. Click the This icon and search for Webhooks
8. Follow the wizard and create an event named "car_parked"
9. Click That and search for email
10. Type the subject and body that you would like emailed to you when the "car_parked" event is called 

#### Add IFTTT Config
1. Navigate back to Home Assistant and click on Configurator in the sidebar
2. Click on the folder icon in the top left corner
3. Click on configuration.yaml
4.  Add the following to the bottom of the file

```
ifttt:
    key: [YOUR_KEY]
```
Click the floppy disk icon to save the file.

#### Add Automation
1. Click on Configuration > Automations
2. Title the Automation "Car Parked"
3. Add a State trigger tied to the stoplight entity you created where From=* and To=red and For=00:00:20 (20 seconds)
4. Add a Call Service action that calls ifttt.trigger with the following data:

```
event: car_parked
```
If your email included additional values, simply add them as `key: value` pairs on their own lines.
5. Test out your new automation by causing your stoplight to enter any state other than red and then the red state for at least 20 seconds.  IFTTT takes several minutes to send the email, so be patient.

With these basics, you can setup any number of automations with IFTTT.

## Thought Questions
1.	**Which version of Home Assistant did you choose to install?  (Docker-based Hass.io, Raspbian-based Hassbian, or install it yourself?) Why did you choose this particular version?**

I installed Hass.io on my raspberry pi because the pi is easily portable and could travel between home and class.  I thought it would be convenient to not have it installed on my laptop and I wasn't quite ready to host it on the internet at home.  When I am ready to host it online, I think I'll install it myself so I can choose whichever underlying OS I want.

2.	**How should you decide which logic to perform in Home Assistant versus coding the logic directly into the devices? What guiding principles would you establish for future devices?**

The logic of these devices should be as abstracted as possible, meaning that the devices should not choose to perform certain actions besides subscribing and listening to MQTT topics.  Ideally, each device listens for actions it should take in some topic meant to drive the device.  Whenever a device performs an action, it should publish that action back to the MQTT server so Home Assistant has the ability to make decisions based on that action.
3.	**What features do you like the most about Home Assistant?**

I like how simple it is to install and configure modules.  I'm also very impressed with its rich automation possibilites like detecting how long a state has been changed or additional conditions that can include data from other sensors.
4.	**Please estimate the total time you spent on this lab and report.**

Building the server and learning how to configure it took 6-8 hours and writing the report took 2-3 hours.

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
