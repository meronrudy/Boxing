# Boxing Headgear Impact Force Monitor

![Image](https://github.com/meronrudy/Boxing/blob/master/1a.jpg)

[Youtube video link](https://youtu.be/L9z6fRinnuc)
# Intro
Project is an impact force monitor that I attached to a boxing headgear.
In addition to collecting impact data, if the hardware detects an impact deemed as too high, it turns on red LED.

Initial goals of the project were to:

1)Equip a wireless hardware device with an accelerometer to monitor potentially dangerous impact forces.

2)Develop Python script to convert three axis acceleration data into usable impact force measurements.

3)Embed circuitry/hardware within a competition Boxing headgear in order to collect data on and alert users of any impact forces above determined concussion safety thresholds.
# Hardware
Project uses a Raspberry Pi Zero Wireless; chosen because of its small size, that it can run off a battery pack but mostly because it was inexpensive and grad school isn't... Ideally would have gone with a small re-chargeable or coin cell powered bluetooth sensor. (yostlabs.com and mbientlab.com both really good and ready to go options w/ API)

The main added piece of hardware is an accelerometer, which measures acceleration or changes in speed (which is what is dangerous for the human body).  The other added hardware component is a single red LED.

https://www.raspberrypi.org/products/raspberry-pi-zero/

https://learn.sparkfun.com/tutorials/accelerometer-basics

https://www.raspberrypi.org/blog/how-to-use-an-led-with-raspberry-pi/
# Impact Force
Impact force is a force that delivers a shock or high impact in a relatively short period of time. It occurs when two entities collide. This collision is the result of one object falling onto, or slamming into, another object. This collision delivers a shock as energy that is transferred to the impacted entity(s). This energy is what causes damage to people who receive blows to the head; in the form of concussion.

# Program

![Image](https://github.com/meronrudy/Boxing/blob/master/img/a.png)

### Setup/Parameters for the Raspberry pi + LIS331 accelerometer
    #Setup LIS331 address
    addr = 0x19

    #Setup the acceleration range
    maxScale = 24

    #Setup the LED GPIO pin
    LED = 26

### Setup files to record the data to

    #Open file to save all data
    #(creates new file in same folder if none
    #and appends to existing file)
    allData = open("AllSensorData.txt", "a")

    #Open file to save alert data
    #(creates new file in same folder if none
    #and appends to existing file)
    alrtData = open("AlertData.txt", "a")

### Function to read the data from accelerometer

    def readAxes(addr):
    data0 = bus.read_byte_data(addr, OUT_X_L)
    data1 = bus.read_byte_data(addr, OUT_X_H)
    data2 = bus.read_byte_data(addr, OUT_Y_L)
    data3 = bus.read_byte_data(addr, OUT_Y_H)
    data4 = bus.read_byte_data(addr, OUT_Z_L)
    data5 = bus.read_byte_data(addr, OUT_Z_H)
    #Combine the two bytes and leftshit by 8
    x = data0 | data1 << 8
    y = data2 | data3 << 8
    z = data4 | data5 << 8
    #in case overflow
    if x > 32767 :
        x -= 65536
    if y > 32767:
        y -= 65536
    if z > 32767 :
        z -= 65536
    #Calculate the two's complement as indicated in the datasheet
    x = ~x
    y = ~y
    z = ~z
    return x, y, z

### Function to calculate g-force from acceleration data
    def convertToG(maxScale, xAccl, yAccl, zAccl):
        #Caclulate "g" force based on the scale set by user
        #Eqn: (2*range*reading)/totalBits (e.g. 48*reading/2^16)
        X = (2*float(maxScale) * float(xAccl))/(2**16);
        Y = (2*float(maxScale) * float(yAccl))/(2**16);
        Z = (2*float(maxScale) * float(zAccl))/(2**16);
        return X, Y, Z

### Function to determine if Impact force is "dangerous" and light LED
          def isDanger(timestamp, x, y, z):
            counter = 0
            x = long(x)
            y = long(y)
            z = long(z)

            if abs(x) > 9 or abs(y) > 9 or abs(z) > 9:
                    alrtData.write(str(timestamp) + "\t" + "x: " + str(x) + "\t" + "y: " + str(y) + "\t" + "z: " +  str(z) + "\n")         
                    GPIO.output(LED, GPIO.HIGH)
            elif abs(x) > 4 or abs(y) > 4 or abs(z) > 4:
                    while abs(x) > 4 or abs(y) > 4 or abs(z) > 4:
                        time_start = time.time()
                        counter = counter + 1
                        if counter > 4:
                            break
                    time_end = time.time()
                    if (counter > 4):
                        alrtData.write(str(timestamp) + "\t" + "x: " + str(x) + "\t" + "y: " + str(y) + "\t" + "z: " + str(z) + "\n")
                        GPIO.output(LED, GPIO.HIGH)

# Main Function
    def main():
    print ("Starting stream")
    while True:
        #initialize LIS331 accelerometer
        initialize(addr, 24)

        #Start timestamp
        ts = time.ctime()

        #Write timestamp to AllSensorData file
        allData.write(str(ts) + "\t")

        #Get acceleration data for x, y, and z axes
        xAccl, yAccl, zAccl = readAxes(addr)
        #Calculate G force based on x, y, z acceleration data
        x, y, z = convertToG(maxScale, xAccl, yAccl, zAccl)
        #Determine if G force is dangerous to human body & take proper action
        isDanger(ts, x, y, z)

        #Write all sensor data to file AllSensorData (as you probably guessed :) )
        allData.write("x: " + str(x) + "\t" + "y: " + str(y) + "\t" + "z: " + str(z) + "\n")

        #print G values (don't need for full installation)
        print "Acceleration in X-Axis : %d" %x
        print "Acceleration in Y-Axis : %d" %y
        print "Acceleration in Z-Axis : %d" %z
        print "\n"

        #Short delay to prevent overclocking computer
        time.sleep(0.2)

![Image](https://github.com/meronrudy/Boxing/blob/master/img/b.png)
![Image](https://github.com/meronrudy/Boxing/blob/master/img/c.png)

### Run until there is a keyboard interrupt
    try:
        while True:
            pass
    except KeyboardInterrupt:
        myprocess.kill()
        allData.close()
        alrtData.close()
        GPIO.cleanup()


if __name__ =="__main__":
    main()
    allData.close()
    alrtData.close()
    GPIO.cleanup()

# SHOUT OUTS

### Jason Thalken, PHD ( Author of "Fight Like a Physicist" ) <http://www.jasonthalken.com/>
### Jennifer Fox (aka jenfoxbot) Bicycle Helmet w/raspberry pi by jenfoxbot <jenfoxbot@gmail.com> Code is open-source, coffee/beerware
### John Eric Goff, PHD ( Author of "The Physics of Krav Maga" and "Gold Medal Physics - The science of sports") <http://johnericgoff.blogspot.com/>
