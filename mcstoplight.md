# Wemos D1 Mini Stoplight
This project involved creating a webserver on a Wemos D1 Mini to control an LED stoplight.

This tutorial is not intended to be a codebase for the project, but a high-level overview of its requirements.  Code is presented in places to help get the project started.  For the complete codebase of my project, visit my [GitHub repo](https://github.com/CraightonH/wemos-stoplight)

## Objectives
* Reinforce enumerating requirements from use cases and user stories 
* Become familiar with the Arduino platform and its programming methodologies
* Program an Arduino-based microcontroller to use the GPIO pins to control LEDs
* Develop iteratively, beginning with a minimum viable product and add functionality until the requirements are met

## Materials
The materials used in this lab were:
* Laptop
* 1x Wemos D1 mini microcontroller
* 1x LED stoplight
* 1x breadboard
* LucidChart (for state diagram)
* Fritzing (for breadboard diagram)

## References
* [HTML in C variable](https://circuits4you.com/2016/12/16/esp8266-web-server-html/) - This site shows a technique to write an HTML page into a variable and how to use that in a sketch.
* [ESP8266 Webserver Documentation](https://github.com/esp8266/Arduino/tree/master/libraries/ESP8266WebServer) - This is the library’s git repo.  It documents its usage and you can see the code they use to make the library work as well. 

## Procedures
### Setup Wemos D1 Mini Development Environment
Arduino has an IDE for development with their chipsets.  The following steps document how to setup this development environment:
1. Connect your micro controller to your computer via a mini USB to USB cable
2. Assuming Windows OS, open Device Manager and expand Serial, then right click on the item and select Update Driver.  After installation the item’s name should change to CH340 USB to serial controller
3. Add the Wemos D1 mini’s library to the Arduino IDE.  Select File > Preferences and enter `http://arduino.esp8266.com/stable/package_esp8266com_index.json` into the Additional Boards Manager URLs field.
4. Add the “LOLIN Wemos D1 R2 & mini” board to your project.  Click on Tools > Board and find the aforementioned board.

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
Assuming the Wemos D1 has male headers soldered into its pin slots, plug it into the breadboard.  With this configuration, red is driven by pin D4, yellow is driven by pin D3, and green is driven by pin D2.
![Breadboard Wiring](https://github.com/CraightonH/school-blog/blob/master/wemos-diagram.png?raw=true)

While this image depicts 3 distinct LEDs each with a resistor in the circuit, mine was a bit different as I used a stoplight-shaped PCB that had built-in LEDs and resistors.

## Thought Questions
1.	**What are some key differences between developing this lab on a Raspberry Pi, and developing on Arduino?**
The biggest difference is with Arduino, you’re locked into a specific language – C – while with the Pi, you can choose pretty much any language you want.  Additionally, because the Pi has an OS on it, you can start and stop your program at will, while with the Arduino you plug it in and it loops through your program until it no longer receives power.  
2.	**What are the strengths and trade-offs of each of these platforms?** 
The strength of the Pi is it’s versatility in implementing the software – if a language can run on an ARM processor, you can use it and implement your project in a familiar-to-you fashion.  Tradeoffs of the Pi are its size and cost.  At about $40, it’s an expensive wifi-enabler and it would be clunky to place near the item you want wifi enabled in some cases.  
The microcontroller excels in size and cost.  It’s extremely cheap and easy to attach to a device to make it wifi-enabled.  The big downside is you must program the microcontroller in C and implementing the code isn’t as familiar a process since the code doesn’t run on top of an OS.
3.	**How familiar were you with the Arduino platform prior to this lab?** 
I had never written code for an Arduino before this lab.  It was fun and not as difficult as I first expected thanks to all the libraries available for it.
4.	**What was the biggest challenge you overcame in this lab?**
The hardest part of the lab was implementing the timer for the cycle function.  In the raspberry pi I implemented a while loop in a background thread, but you don’t get multiple threads on this platform.  So it took awhile for me to wrap my mind around making my own timer with the D1’s built-in clock while not blocking the only thread available to the program.
5.	**Please estimate the total time you spent on this lab and report.**
Coding and wiring took me about 3-4 hours. This report took me about 1.5 hours.

## Certification of Work
I certify that the solution presented in this lab represents my own work.

Craighton Hancock

## Appendix
### Appendix A - Code
All of the code for this project can be found at my [GitHub repo](https://github.com/CraightonH/wemos-stoplight).