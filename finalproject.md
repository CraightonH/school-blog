# Automated Humidifier
This is the semester final project that I chose to do.  It uses all the tricks we've learned in class until now to bring an automated humidifier system together.

## Objectives
* Unique device designed and built to meet your specific need
* Connects to the existing event bus system and automation system already established
* At least one non-web/non-app interface (voice, etc)

## Materials
The materials used in this lab were:
* 1x Wemos D1 mini microcontroller
* 1x Wemos D1 mini DHT22 shield
* 1x WiFi socket (sell for < $10)
* Raspberry Pi (with home assistant installed)

## References
* [DHT22 shield](https://diyprojects.io/shield-wemos-d1-mini-dht11-dht22-arduino-code-esp-easy/#.Xe7_3ehKjAQ) - This is a tutorial that walks through the steps to getting this working.
* The DHT22 example from the Arduino IDE was also extremely helpful in understanding how to write code to interface with the sensor.

## Procedures
Because the DHT22 is built onto a shield, it just fits on top of the Wemos as depicted in the following image.  Just line up the pin numbers from the wemos and shield:

![DHT22 Shield](https://www.projetsdiy.fr/wp-content/uploads/2017/01/4-shield-dht22-wemos-d1-mini-monte-empile-assemble.jpg)

A high level overview is shown with the following diagram:
![Logical Diagram](https://www.lucidchart.com/publicSegments/view/75fc38f6-875b-40c0-bcb6-1e8872f9d5c8/image.png)

### Coding the Sensor
The libraries necessary are from adafruit: Adafruit Unified Sensor and DHTSensor.  
1. Click on Tools > Manage Libraries
2. Search for Adafruit Unified Sensor and install the latest version
3. Search for DHTSensor and install the latest version
4. Include the DHT library in your sketch (it depends on the unified sensor but you don't need to include the library)

```
#include "DHT.h"
```

5. Setup the library:

```
#define DHTPIN 2     // Digital pin connected to the DHT sensor
#define DHTTYPE DHT22   // DHT 22  (AM2302), AM2321
DHT dht(DHTPIN, DHTTYPE);  // initialize sensor object

void setup(void) {
    Serial.begin(115200);
    dht.begin();  // start sensor
}
```

6. Get the data:

```
void loop(void) {
  float humidity = dht.readHumidity(); // get the humidity as a percentage
  float temperature = dht.readTemperature(true); // get the temperature in F.  false for C.
  if (isnan(humidity) || isnan(temperature)) {
      Serial.println("failed getting sensor data");
      return;
  }
  // send data to MQTT topic here
  delay(2000); // the sensor is only accurate to every 2 seconds
}
```

As you can see, the libraries make this sensor extremely easy to work with.  Once you have this data, all you need to do is choose which MQTT topic to send the data to.

### Create Home Assistant Sensor
To visualize the data collected by the sensor, we need a sensor object to represent the data in the topic.  In previous projects this was done on the Wemos by sending a special config message to an MQTT topic which Home Assistant interpreted and turned into a sensor object.  Let's do it a different way this time.

1. Install Configurator via the Add-on Store
2. Once installed, click on Configurator in the sidebar and click the folder icon in the top left side of the menu
3. Click on the configuration.yaml file
4. Add a block at the end of the file that looks like the following:

```
sensor:
  - platform: mqtt
    name: "Bedroom Temperature"
    state_topic: "home/bedroom/temp"
    unit_of_measurement: '°F'
    device_class: "temperature"
  - platform: mqtt
    name: "Bedroom Humidity"
    state_topic: "home/bedroom/hum"
    unit_of_measurement: '%'
    device_class: "humidity"
```

5. Restart the Home Assistant server via Configuration > System Controls > Restart (Note: this doesn't restart the host system, just the Home Assistant service)

Just like that you should have a sensor object that represents the data found in those topics.

### Setup Socket
I thought a lot about using the relay to control the humdifier directly, but that meant damaging the humidifer to splice the relay into it.  I then thought about making my own power outlet (there are many tutorials out there to do this), but then thought for the time and money it would take to build, I may as well just buy a smart socket for ~$8.  

I chose one that integrates with IFTTT so that I could leverage Home Assistant's automations.  Follow the instructions provided by the manufacturer on how to setup IFTTT integrations with the socket.  Then create the following relationships in IFTTT:
1. Login to IFTTT
2. Create new applet
3. This = Webhook named "socket_on"
4. That = Smart Life device on
5. Create new applet
6. This = Webhook named "socket_off"
7. That = Smart Life device off

Now IFTTT can turn the socket on and off.

### Setup Home Assistant Automation
With IFTTT controlling the socket and our humidity sensor pushing data to `home/bedroom/hum`, we can create an automation in home assistant to fire that ifttt webhook when the humidity drops to less than 47%.

1. Create new Automation
2. Trigger = Numeric State of sensor.bedroom_humidity below 47% (alternatively, read the MQTT topic)
3. Action = Call Service ifttt.trigger with service data of "event: socket_on"
4. Create new Automation
5. Trigger = Numeric State above 47%
6. Action = Call Service ifttt.trigger with service data of "event: socket_off"

### Final Steps
Now you just need to plug in the humidifier to the socket and turn the power switch on and leave it on always.  The socket will be turned on and off which will in turn provide power to the humidifer or cut it.  Then plug in your sensor in the same room.

Just like that, you have a working automated humidifer!  Because the automations trigger off of humidity levels which are actual values, you can speed up the process by putting the sensor directly in the humidifier's stream while it's on which will eventually trigger a high enough humidity to turn off the socket.

## Certification of Work
I certify that the solution presented in this lab represents my own work.

Craighton Hancock

## Appendix
### Appendix A - Sensor Code
All of the code for this project can be found at my [GitHub repo](https://github.com/CraightonH/wemos-mqtt-temp-hum-sensor).
### Appendix B - How to Setup Home Assistant
The instructions for setting up Home Assistant can be found at [a previous project](https://craightonh.github.io/school-blog/homeassistant).
