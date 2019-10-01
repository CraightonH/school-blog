# Raspberry Pi Stoplight
This project involved creating a webserver on a raspberry pi to control an LED stoplight via the pi's GPIO pins.

This tutorial is not intended to be a codebase for the project, but a high-level overview of its requirements.  Code is presented in places to help get the project started.  For the complete codebase of my project, visit my [GitHub repo](https://github.com/CraightonH/pi-automation)

## Objectives
* Learn to enumerate requirements from use cases and user stories
* Learn to utilize the general purpose input/output (GPIO) pins on the Raspberry Pi to control LEDs
* Learn to develop a minimum viable product, and deliver new functionality through subsequent updates
until the requirements are met, and all of the deliverables are acceptable.
* Learn to use Github for progressive code commits and version tracking.

## Materials
The materials used in this lab were:
* Laptop
* Raspberry Pi
* 4x male to female wires
* 1x LED stoplight
* 1x breadboard
* LucidChart (for state diagram)
* Fritzing (for breadboard diagram)

## References
The following were useful resources in the development of this project:
* [Flask Documentation](http://flask.palletsprojects.com/en/1.1.x/quickstart/#a-minimal-application) - This site helps get you started with a python web server.
* [Python Threading Documentation](https://docs.python.org/3/library/threading.html) - This site documents python's threading library which is useful if your code has a blocking operation (loop).
* [Python GPIO Documentation](https://www.raspberrypi.org/documentation/usage/gpio/python/README.md) - This site documents several common use cases for simple GPIO projects such as wiring up LEDs.

## Procedures
### Setup Python Web Server
Follow the procedures to create a web server in python:
1. Ensure python is installed with necessary prerequisites by opening a terminal and typing the following:
```
sudo apt install python3 python3-pip
sudo pip3 install Flask requests RPi.GPIO
```
**NOTE:** All of the above packages should come preinstalled with a current version of Raspbian.
2. Get started with a very basic web server by creating a file named `server.py`:
```
from flask import Flask
app = Flask(__name__)
@app.route('/')
def hello_world():
    return 'Hello, World!'
```
3. Run the server:
```
python3 server.py
```
4. (Optional) Set server to run on port `80`.  By default the server will run on port `5000`.  Add the following to `server.py`:
```
if __name__ == "__main__":
  app.run(host="0.0.0.0", port=80, debug=True)
```
This requires that the server be run as root:
```
sudo python3 server.py
```

### Implement Stoplight Logic
The stoplight is very simple and can be modeled by the following state diagram:
![Stoplight State Diagram](https://www.lucidchart.com/publicSegments/view/034941ad-58dc-4ddf-9982-6051a4b32b6f/image.png)
As seen above, each state has 4 possible ways to be entered - whenever the `/stoplight/[color|off]` command is called from any state, where `color` is one of `red`, `yellow`, `green`.  Each state has 4 possible states it can transition to - every state by calling `/stoplight/[color|off]`. 

#### A Word on Cycling
Implementing logic to cycle through the lights like a stoplight normally would is simpler than depicted in the diagram above.  Start with an arbitrary light, red.  Red must always go to green which must always go to yellow which must always go to red, etc.  In less words:
```
Red => Green => Yellow => Red...
```

### Design the Circuit
Wire the raspberry pi GPIO pins to 3 LEDs according to the diagram below.  Note that this diagram shows individual LEDs with resistors included in the circuit.  My LEDs came as a PCB in the shape of a stoplight with resistors included on the board.
![Breadboard Wiring](https://github.com/CraightonH/school-blog/blob/master/LEDStoplight.png?raw=true)

After the wires are connected, and the web server is running with the ability to modify the GPIO pin signals, the lights should light up according to the logic presented in the state diagram.

## Thought Questions
1. **What language did you choose for implementing this project? Why?**
    I chose Python because it's a simple language with a lot of cross platform capability.  One of the main reasons is I can easily abstract the webserver from the GPIO portion of the project and run the server on some other computer, be it Linux or Windows, so that the server doesn't live or die with the pi being turned on or not.  Then the pi can contact the external web service to determine how it should manipulate the lights.
2. **What is the purpose of the resistor in this simple circuit? What would happen if you omitted it?**
    The resistor reduces the current in the circuit to protect the LED.  If it were not there, the LED would burn out much faster.
3. **What are practical applications of this device? What enhancements or modifications would you make?**
    This isn't a very practical device - I'm not going to keep a pi hooked up to an LED stoplight for fun.  However, the project gives introductory experience into working with GPIO pins on the pi and connecting the physical with the virtual.  An enhancement I made was creating a separate, non-web interface for the pi.  I have a 7" touch screen for the pi and I made a full screen GUI that interfaces with the webserver to change the lights.  The code can be found [here](https://github.com/CraightonH/pi-gui).
4. **Please estimate the total time you spent on this lab and report.**
    Coding and wiring took me about 3-4 hours (not counting the GUI I worked on).  This report took me about 5 hours.

## Certification of Work
I certify that the solution presented in this lab represents my own work.

Craighton Hancock

## Appendix
### Appendix A - Code
All of the code for this project can be found at my [GitHub repo](https://github.com/CraightonH/pi-automation).