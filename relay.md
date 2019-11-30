# MQTT Relay
This is the final project in this 6-part series.  It combines techniques learned up to this point and heavily utilizes MQTT and Home Assistant.  This project adds a relay to the system with the intent to control the garage door opener remotely and integrated with their rest of our home automation.  It also includes how to control the relay through an external service. 

## Objectives
* Build a controller to interface with the existing garage door opener. (You don’t actually have to connect the relay to a garage door opener, but should be able to show that it is performing the necessary action to toggle the garage door opener.)
* Integrate the garage system with at least one external service (IFTTT, Adafruit,io, etc) to introduce some additional functionality of your choice.
* Utilize a non-web interface for the garage door opener

## Materials
The materials used in this lab were:
* Laptop
* 1x Wemos D1 mini microcontroller
* 1x Wemos D1 mini relay shield
* 1x breadboard
* 3x wires
* Raspberry Pi (with home assistant installed)

## References
* [Relay Shield Wiring](https://arduino.stackexchange.com/questions/36714/wemos-d1-relay-shield) - This SO question included a diagram of how the individual wired his relay.
* [Home Assistant MQTT Publish Service](https://www.home-assistant.io/docs/mqtt/service/) - This is the Home Assistant MQTT Publish Service documentation.
* [Home Assistant Rest API Docs](https://developers.home-assistant.io/docs/en/external_api_rest.html) - This shows how to use the Rest API in Home Assistant.
* [Youtube iOS Shortcuts walkthrough](https://www.youtube.com/watch?v=HB7kDIElJYE) - Shows how to work with REST APIs using iOS Shortcuts.

## Procedures
The following diagram shows how to wire the D1 mini to the relay shield:
![Relay Diagram](https://github.com/CraightonH/school-blog/blob/master/RelayDiagram.png?raw=true)

### Code
This is a very simple circuit which makes it very simple code.  The relay will need to listen to some topic that drives the relay to turn it on and off.  Because the relay doesn't determine whether the door is open or not, we do not need to publish a status of this device.  If you would like more visibility on this device specifically, you certainly could publish to a topic that would display this device's state.  We're more concerned with whether the door is open or not, and we already have a sensor checking whether the door is open or not which is what we'll listen to for that information. 

The important bits for this to work properly are a few variables as follows:
```
const int RELAYPIN = D1;
String devID = "esp8266-relay";
const char* doorActionTopic = "/garage/door/action";
char message_buff[100];
```
With a function to drive the relay:
```
void setRelay(String message) {
  Serial.println(message);
  if (message == "off") {
    Serial.println("turning off relay");
    digitalWrite(RELAYPIN, LOW);
  } else if (message == "on") { // require explicit on
    Serial.println("turning on relay");
    digitalWrite(RELAYPIN, HIGH);
  } else {
    pubDebug(String("Wrong relay message received: " + message));  
  }
}
```
A function to listen to the topic and then fire `setRelay()`:
```
void callback(char* topic, byte* payload, unsigned int length) {
  int i = 0;
  for(i = 0; i < length; i++) {
    message_buff[i] = payload[i];
  }
  message_buff[i] = '\0';
  String message = String(message_buff);
  Serial.println(message);
  setRelay(message);
}
```
And setup the pin on the Wemos and the above callback:
```
void sendHassDeviceConfig() {
  client.publish("homeassistant/sensor/garage/door_relay/config", "{\"name\": \"Garage Door Relay\", \"state_topic\": \"/garage/door/action\"}");
}

void reconnectMQTT() {
  while(!client.connected()) {
    if (client.connect((char*) devID.c_str(), "[yourmqttusername]", "[yourmqttpassword]")) {
      Serial.println("Connected to MQTT server");
      String message = devID;
      message += ": ";
      message += WiFi.localIP().toString();
      if (client.publish(logInfoTopic, message.c_str())) {
        Serial.println("published successfully");
      } else {
        Serial.println("failed to publish");
      }
      client.subscribe(doorActionTopic);
      sendHassDeviceConfig();
    } else {
      Serial.println("MQTT connection failed");
      delay(5000);
    }
  }
}

void setup(void) {
  Serial.begin(115200);
  findKnownWiFiNetworks();
  pinMode(RELAYPIN, OUTPUT);
  client.setCallback(callback);
  setRelay("off"); // start the relay off
}

void loop(void) {
  if (!client.connected()) {
    reconnectMQTT();
  }
  client.loop();
}
```
Code not shown is what you need for wifi connections which has been shown in past tutorials.

Now test out the relay by publishing `on` and `off` messages to the `/garage/door/action` topic.  You should hear a click when the relay turns on and off as well as a red LED toggle accordingly.  

### Setup External Service
Once the code is working, you're now ready for the fun part - hooking into an external service.
For the last project, I provided a tutorial on how to setup IFTTT.  Since I have an iPhone and I want integration with the native services of the iPhone, I chose to setup iOS Shortcuts for this project.  

[This Youtube video](https://www.youtube.com/watch?v=HB7kDIElJYE) is really good at showing how to integrate with an external weather API which uses a lot of the features we need to hook into Home Assistant.

#### Create a Long-Lived Access Token
I highly recommend doing this on your iOS device as the token is *long*.  
1. Navigate to your home assistant portal in a browser, ie. `http://[homeassistant]:8123`
2. Click on your profile
3. Scroll to the bottom and click Create Token under Long-Lived Access Tokens
4. Name it something rememberable like `iOS Shortcuts`
5. Copy the token into a note on your phone temporarily so you don't lose it.

#### Create Open Garage Door Shortcut
On your iOS device:
1. Open Shortcuts
2. Create Shortcut
3. Tap the `+` to add new item
4. Search for URL and add it
5. Set the URL to `[homeassistant URL]/api/services/mqtt/publish`
6. Tap the `+` to add new item
7. Search for Get Contents of URL and add it
8. Tap Show More
9. Change the Method to POST
10. Add header of `content-type: application/json`
11. Add header of `Authorization: Bearer [your token from the last step]`
12. Add Request Body of JSON for:
```
topic: /garage/door/action
payload: on
```
13. Test by listening to that topic and verifying you receive a message.  Additionally, the relay should also turn on.
14. (Optional) Tap the `+` to add new item
15. (Optional) Search for Wait and add it
16. (Optional) Change wait to 5 seconds
17. (Optional) Repeat steps 3-4
18. (Optional) Set the URL to `[homeassistant URL]/api/states/sensor.garage_door`
19. (Optional) Repeat steps 6-8
20. (Optional) Make sure the Method is GET
21. (Optional) Copy the same headers from steps 10-11
22. (Optional) Tap the `+` to add new item
23. (Optional) Search for Get Dictionary Value and add it
24. (Optional) Change the Dictionary config to read like: Get `Value` for `state` in `Contents of URL`
25. (Optional) Tap the `+` to add new item
26. (Optional) Search for Text and add it
27. (Optional) Type: Garage door is `Dictionary Value`
28. (Optional) Tap the `+` to add new item
29. (Optional) Search for Speak Text and add it
30. (Optional) Make sure it Speaks from the Text module above.
31. (Optional) Test again, this time after 5 seconds Siri will say what the state is of the garage door.

Save this and name it `Open Garage Door`.  Note that because this is an iOS shortcut, you can say "Hey Siri, open garage door" and she will invoke this shortcut and she will then respond after 5 seconds with the state of the garage door (ideally being open).

#### Create Close Garage Door Shortcut
1. Duplicate the Open Garage Door shortcut
2. Modify the JSON payload to `off`
3. Rename it to `Close Garage Door`

Now you can ask Siri to "close garage door" and after 5 seconds she will respond with the state of the door.  Note that this is just a proof-of-concept.  In reality, you'd have to change the wait time to be sufficient to allow the door to close before querying for the door's state.

## Thought Questions
1.	**What services did you consider to integrate into your project and why?**

I thought about IFTTT which would have been nice because it would share the integration across multiple devices, but ultimately landed on Shortcuts because of its integration with my phone.  I liked the idea of being able to control my garage door entirely with my voice, especially since I'd likely be opening the door while driving. 

2.	**What services would you like to integrate in the future?**

I would like to integrate location services into the shortcut so it would automatically open the door when I get close to my house.  There are some challenges to this though, like the door opening when I'm driving *by* my house without intending to drive into my garage.  It would be hard to automatically get the door to open in all the scenarios I would want it to *and* all the scenarios I wouldn't want it to.

3.	**What was the biggest challenge you overcame in this lab?**

The hardest part was figuring out how to wire the relay.  Wiring 5V and ground seemed clear enough, but I still don't understand why my control wire works and I couldn't really find anywhere that explains how to control the relay.  I took about 30 minutes researching how to control the relay before just jamming wires into place and seeing if it worked - which it did.  The rest of the project was extremely straightforward and relatively simple.

4.	**Please estimate the total time you spent on this lab and report.**

Building the server and learning how to configure it took 2 hours and writing the report took 2 hours.

## Certification of Work
I certify that the solution presented in this lab represents my own work.

Craighton Hancock

## Appendix
### Appendix A - Relay Code
All of the code for this project can be found at my [GitHub repo](https://github.com/CraightonH/wemos-mqtt-relay).
### Appendix B - How to Setup Home Assistant
The instructions for setting up Home Assistant can be found at [a previous project](https://craightonh.github.io/school-blog/homeassistant).
