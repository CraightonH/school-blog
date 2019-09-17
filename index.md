# Raspberry Pi Stoplight
This project involved creating a webserver on a raspberry pi to control an LED stoplight via the pi's GPIO pins.

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
As seen above, each state has 4 possible ways to be entered - whenever the `/stoplight/[color|off]` command is called from any state.  Each state has 4 possible states it can transition to - every state by calling `/stoplight/[color|off]`. 

### Design the Circuit
Wire the raspberry pi GPIO pins to 3 LEDs according to the below diagram.  Note that this diagram shows individual LEDs with resistors included in the circuit.  My LEDs came as a PCB in the shape of a stoplight with resistors included on the board.
![Breadboard Wiring](https://github.com/CraightonH/school-blog/blob/master/LEDStoplight.png)