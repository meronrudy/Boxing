# Boxing Headgear Impact Force Monitor
 
! [Image} 1a.jpg

# Intro
Project is an impact force monitor that I put inside of a boxing headgear.
In addition to collecting impact data, if the hardware detects an impact deemed as too high, it turns on red LED.

Initial goals of the project were to:

1)Equip a wireless hardware device with an accelerometer to monitor potentially dangerous impact forces.

2)Develop Python script to convert three axis acceleration data into usable impact force measurements.

3)Embed circuitry/hardware within a competition Boxing headgear in order to collect data on and alert users of any impact forces above determined concussion safety thresholds.
# Hardware
Project uses a Raspberry Pi Zero Wireless; chosen because of its small size, that it can run off a battery pack but mostly because it is inexpensive.

The main added piece of hardware is an accelerometer, which measures acceleration or changes in speed (which is actually what is dangerous for the human body).  The other added hardware component is a single red LED. 

https://www.raspberrypi.org/products/raspberry-pi-zero/

https://learn.sparkfun.com/tutorials/accelerometer-basics

https://www.raspberrypi.org/blog/how-to-use-an-led-with-raspberry-pi/
# Impact Force
Impact force is a force that delivers a shock or high impact in a relatively short period of time. It occurs when two entities collide. This collision is the result of one object falling onto, or slamming into, another object. This collision delivers a shock as energy that is transferred to the impacted entity(s). This energy is what causes damage to people who receive blows to the head; in the form of concussion.

