# Integrating physical devices with IOTA — Pay the light (WiFi edition)

The second part in a series of beginner tutorials on integrating physical devices with the IOTA protocol.

![img](https://miro.medium.com/max/700/1*YF5cHGzDgGuKgIvJZRGFIQ.jpeg)

## Introduction

This is the second part in a series of beginner tutorials where we explore integrating physical devices with the IOTA protocol. In this tutorial we are basically recreating the use-case from the [first tutorial](https://medium.com/@hugogregersen/integrating-physical-devices-with-iota-83f4e00cc5bb). This time however, instead of having our relay module hard wired to a Raspberry PI, we will use a low cost WiFi module for signaling the relay.

If you haven’t read the [first tutorial](https://medium.com/@hugogregersen/integrating-physical-devices-with-iota-83f4e00cc5bb) in this series, you should read it before continuing as it reflects on the same basic use-case we are trying to recreate in this second tutorial.

## The Use Case

Now that our forward leaning hotel owner has his new IOTA powered refrigerator payment system installed and working as described in the [first tutorial](https://medium.com/@hugogregersen/integrating-physical-devices-with-iota-83f4e00cc5bb), he start noticing some issues that make the solution less than ideal. First of all, having a dedicated Raspberry PI for each refrigerator is quite expensive as each Raspberry PI costs about 30–40 USD. Secondly, if he wants to make changes to the Python code, or even change the IOTA payment address, he has to gain access to each PI and update them manually, one-by-one.

One solution to these problems would be to have a multi-channel relay with a central Raspberry PI functioning as a common control unit for all his refrigerators as described in the [first tutorial](https://medium.com/@hugogregersen/integrating-physical-devices-with-iota-83f4e00cc5bb). The bad news is that he would then have to provide physical wiring between the central PI and each refrigerator in the hotel. Also, there would be a limit to the number of refrigerators (relays) he could manage depending on the number of available GIO pin’s on his Raspberry PI.

After scratching his head over this problem for a few days he comes up with the obvious solution; What if he could turn ON/OFF the relays using some type of wireless communication, and at the same time have his Python code running on a central server for easy update and maintenance?

Lucky for him, this is exactly what we are gonna do in this second tutorial.

## The amazing ESP8266

The ESP8266 is a low-cost WiFi microchip with full TCP/IP stack and micro-controller capability that can be programmed using the Arduino IDE. Based on the ESP8266 chip, a large number of different WiFi modules in different shapes and sizes have been developed and are now available on the market at a very low cost, one example is the ESP-01. The ESP-01 with its internal programmable IO pins is perfect for simple low cost WiFi projects. The ESP8266 is also capable of running its own web server, which we will be using in this project for communicating with the WiFi module.

![img](https://miro.medium.com/max/225/1*G5op1P-JgKjvKoKFvXFhvQ.jpeg)

Notice that you can even get relay modules with the ESP-01 integrated for compact installation and minimum wiring.

![img](https://miro.medium.com/max/225/1*iomjofzeJupRK5RFJ7kAfA.jpeg)

*Note!*
*As described above, there are a large number of different WiFi modules based on the ESP8266 chip available on the market and most of them would probably work equally well for our project. The ESP-01 was highlighted for its low cost and small form factor. However, it should be noted that the ESP-01 does not have an integrated USB port for uploading our Arduino code/firmware to the micro-controller. Not having a built in USB port makes it a little complicated (especially without the ESP-01 interface module) for this beginners tutorial. So instead of using the ESP-01 i’m gonna use my trusted WeMos D1 WiFi board thru out this tutorial as it already have a built in USB port, power socket etc. making it basically plug-and-play.*

![img](https://miro.medium.com/max/272/1*8XkemkA0i2wP3cvuF-V_Wg.jpeg)

## The Arduino IDE

The easiest way of programming the ESP8266 is using the Arduino IDE. Inside the IDE, you simply select the type of ESP8266 board you have and you are ready to compile and upload code to the MCU. The programming syntax used inside the Arduino IDE is a C++ like programming language used for programming on the popular Arduino prototyping and development platform. The Arduino IDE can be downloaded for free [here](https://www.arduino.cc/en/Main/Software?). To learn more about configuring and programming your ESP8266 board using the Arduino IDE, you should take a look at the documentation that comes with your particular ESP8266 board.

## Components

In this section we will take a look at the components required to building the project. You should be able to acquire them at most electronic stores or on EBAY/AMAZON for less than 20 USD assuming you already own a computer capable of running Python. The computer will act as our central control unit communicating with the IOTA tangle and sending HTTP requests to our ESP8266 WiFi module. I case you followed the [first tutorial](https://medium.com/@hugogregersen/integrating-physical-devices-with-iota-83f4e00cc5bb)you probably still own a Raspberry PI that can function as the central computer in this tutorial.



**ESP8266 enabled WiFi module**
The ESP8266 enabled WiFi module is used for wireless communication and ON/OFF switching of our relay using its built in IO pins. There are many variants of these WiFi modules on the market and any one of them should work for our project. I recommend you pick up a module that has a built in USB port as it will make it easier when uploading code (arduino sketches) to your module. For the rest of this tutorial i will be using my WeMos D1 WiFi board. See [here](http://cyaninfinite.com/tutorials/getting-started-with-the-wemos-d1-esp8266-wifi-board/) to learn more about using the WeMos D1 WiFi board with the Arduino IDE.

![img](https://miro.medium.com/max/272/1*8XkemkA0i2wP3cvuF-V_Wg.jpeg)

**Relay**
The Relay is used to switch ON/OFF our power circuit and thereby our device (in this case the LED). To simplify our circuit we will be using a relay module (shield) that has all the required components, pins and connectors built in to the module. Notice that you can buy these modules with multiple relays (channels) that can be switched ON/OFF individually. This can be useful in cases where you need to manage multiple devices as discussed earlier.

![img](https://miro.medium.com/max/225/1*4ktyvSuyTd6_jlAmj2ZaOQ.jpeg)

**Breadboard**
The breadboard is used to wire up our circuit without having to do any soldering, making it easy to assemble and disassemble.

![img](https://miro.medium.com/max/251/1*Igi0Nd93Vu08Sb4N_jEV9Q.jpeg)

**Light Emitting Diode (LED)**
The LED will light up when powered and will be representing our physical device (refrigerator) is this project.

![img](https://miro.medium.com/max/225/1*7yFEMtUvCuieBwPzlCmQpA.jpeg)

**Resistor (330 ohm)**
The resistor is used to limit the current sent to our LED. Without the resistor you may damage the LED and/or the Raspberry PI. The type of resistor you should use depends on the type of LED and the amount of voltage you are providing to the circuit. In my case I’m using a 9V battery so a 330 ohm resistor should be fine. I suggest you research what type of resistor you should use depending on the components used in your version of the project.

![img](https://miro.medium.com/max/226/1*WmcyYVlVm0IJtKlvjSO7cQ.jpeg)

**Battery**
The battery is used to provide power to our power circuit. In my case am using a 9V battery.

![img](https://miro.medium.com/max/224/1*KR1UwBURPs_TN9ISPwmH2A.jpeg)

**Wires**
We also need some wires to hook it all up.

![img](https://miro.medium.com/max/225/1*zJ5LOJ-NyWiUYlMq-8s1tw.jpeg)

**QR Code**
A printed QR code of the IOTA payment address is handy if you want to pay the LED using a mobile IOTA wallet. You will find a QR code when generating new addresses using the IOTA wallet or by searching an existing address at [https://thetangle.org](https://thetangle.org/)

![img](https://miro.medium.com/max/163/1*0Birfy2lGcfHKVHgxvEEPQ.png)

**A computer**
A computer capable of running Python for communicating with the IOTA tangle and sending HTTP requests to our WiFi module. You can basically use any internet connected computer, including the Raspberry PI.

## Wiring the project

Now, Lets look at how to wire up the circuit used in this project.

![img](https://miro.medium.com/max/700/1*AdokhyEJbRnaQymSbbcE8A.png)



Connect the circuit as follows:

1. Connect 5V from the WeMos D1 to the VCC pin on the relay module.
2. Connect GND from the WeMos D1 to the GND pin on the relay module.
3. Connect D14/SDA/D4 on the WeMos D1 to the IN (Signal) pin on the relay module.
4. Connect the COM terminal on the relay module to the positive side (+) of the battery.
5. Connect the NO terminal on the relay module to the Anode (+) side of the LED having the resistor in between.
6. Connect the negative side (-) of the battery to the Kathode (-) side of the LED.

*Note!*
*Notice how the two pins on the LED have different lengths. The short pin represents the Kathode (-) side and the long pin represents the Anode (+) side of the LED.*

## Required Software and libraries

As appose to the first tutorial where we had all our software running on the Raspberry PI, we now need to split our code into to parts where we have some Python code running on the central control unit that interacts with the IOTA tangle and sends HTTP requests to our WiFi module. We will also need some Arduino code running on the WiFi module that listens for HTTP requests and handels the IO pins on our WiFi module.

In case you haven’t already and need to install Python on the central computer, you will find it here: https://www.python.org/downloads/

You also need to install the PyOTA API library on the central computer that will allow us to access the IOTA tangle using the Python programming language. The PyIOTA API library with installation instructions can be found here: https://github.com/iotaledger/iota.lib.py

For sending HTTP requests using Python you also need to install the httplib2 library on your central computer. The httplib2 with installation instructions can be found here: https://github.com/httplib2/httplib2

## The Arduino code

First, let’s look at the Arduino code that will be running on our ESP8266 WiFi module. What’s happening here is that we first define the PIN to be used for signaling the relay module. Secondly, we connect to the WiFi network using the ssid and password provided, so **it’s important that you update the code with your WiFi credentials**. Next, we start the HTTP server that allows us to listen for HTTP request, and finally we start listening for incoming HTTP requests, setting the relay PIN High or Low accordingly.

*Note!*
*In a real life scenario you would probably want to assign a static IP address to the WiFi module, preventing it from getting a new IP address in case of a power failure etc. I will not go into assigning static IP addresses in this tutorial, just notice that the actual IP address used by the WiFi module is printed in the Arduino serial monitor when powering up or resetting the module. Make a note of this IP address as we will use it in our Python code later on.*

```c++
/*
 *  This sketch demonstrates how to set up a simple HTTP-like server.
 *  The server will set a GPIO pin depending on the request
 *    http://server_ip/gpio/0 will set the GPIO2 low,
 *    http://server_ip/gpio/1 will set the GPIO2 high
 *  server_ip is the IP address of the ESP8266 Arduino module, will be 
 *  printed to Serial when the module is connected.
 */

// Include the ESP8266 library
#include <ESP8266WiFi.h>

// Specify your WiFi ssid and password
const char* ssid = "write your wifi ssid here";
const char* password = "write your wifi password here";

// Create an instance of the server
// specify the port to listen on as an argument
WiFiServer server(80);

// Define relay PIN
int ledPin = D2;

void setup() {
  Serial.begin(115200);
  delay(10);

  // Setup relay PIN
  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, LOW);
  
  // Connect to WiFi network
  Serial.println();
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
  
  WiFi.begin(ssid, password);
  
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected");
  
  // Start the server
  server.begin();
  Serial.println("Server started");

  // Print the IP address
  Serial.println(WiFi.localIP());
}

void loop() {
  // Check if a client has connected
  WiFiClient client = server.available();
  if (!client) {
    return;
  }
  
  // Wait until the client sends some data
  Serial.println("new client");
  while(!client.available()){
    delay(1);
  }
  
  // Read the first line of the request
  String req = client.readStringUntil('\r');
  Serial.println(req);
  client.flush();
  
  // Match the request
  bool val;
  if (req.indexOf("/gpio/0") != -1)
    val = LOW;
  else if (req.indexOf("/gpio/1") != -1)
    val = HIGH;
  else {
    Serial.println("invalid request");
    client.stop();
    return;
  }

  // Set GPIO2 according to the request
  digitalWrite(ledPin, val);
  
  client.flush();

  // Prepare the response
  String s = "HTTP/1.1 200 OK\r\nContent-Type: text/html\r\n\r\n<!DOCTYPE HTML>\r\n<html>\r\nGPIO is now ";
  s += (val)?"high":"low";
  s += "</html>\n";

  // Send the response to the client
  client.print(s);
  delay(1);
  Serial.println("Client disonnected");

  // The client will actually be disconnected 
  // when the function returns and 'client' object is detroyed
}
```

The Arduino source code for this project can be found [here](https://gist.github.com/huggre/aed0baaef3dc7cf8dd80d925da2b8482).

To upload the code above to your WiFi module you first have to install the Arduino IDE and make sure that you are able to communicate correctly with WiFi module using an USB cable. I will not go into details on setting up and configuring your board with the Arduino IDE as this varies from board to board. I suggest you confront the documentation related to your particular board. When your board is working correctly with the Arduino IDE, you simply download the code above, open the code in the Arduino IDE and press the Upload button.

## The Python code

Now, let’s look at the Python code running on our central computer. The good news is that the code is more or less the same as what we had in the first tutorial. Only difference is that instead of setting the Raspberry PI GIO pins High or Low, we now send an HTTP request to the WiFi module and let it take care of switching ON/OFF the relay. **Make sure you update the url_on and url_off variables with your WiFi module’s IP address. You will see this IP address in the Arduino IDE serial monitor when powering up or resetting the WiFi module.**

```python
# Import the httplib2 library and create a http object.
import httplib2
http = httplib2.Http()

# Define URL's used when sending http requests
url_on = 'http://192.168.100.9/gpio/1'
url_off = 'http://192.168.100.9/gpio/0'

# Imports some Python Date/Time functions
import time
import datetime

# Imports the PyOTA library
from iota import Iota
from iota import Address

# Make sure light is off at startup
response, content = http.request(url_off, 'GET')

# Function for checking address balance on the IOTA tangle. 
def checkbalance():

    print("Checking balance")
    gb_result = api.get_balances(address)
    balance = gb_result['balances']
    return (balance[0])

# URL to IOTA fullnode used when checking balance
iotaNode = "https://nodes.thetangle.org:443"

# Create an IOTA object
api = Iota(iotaNode, "")

# IOTA address to be checked for new light funds 
# IOTA addresses can be created using the IOTA Wallet
address = [Address(b'GTZUHQSPRAQCTSQBZEEMLZPQUPAA9LPLGWCKFNEVKBINXEXZRACVKKKCYPWPKH9AWLGJHPLOZZOYTALAWOVSIJIYVZ')]

# Get current address balance at startup and use as baseline for measuring new funds being added.   
currentbalance = checkbalance()
lastbalance = currentbalance

# Define some variables
lightbalance = 0
balcheckcount = 0
lightstatus = False

# Main loop that executes every 1 second
while True:
    
    # Check for new funds and add to lightbalance when found.
    if balcheckcount == 10:
        currentbalance = checkbalance()
        if currentbalance > lastbalance:
            lightbalance = lightbalance + (currentbalance - lastbalance)
            lastbalance = currentbalance
        balcheckcount = 0

    # Manage light balance and light ON/OFF
    if lightbalance > 0:
        if lightstatus == False:
            print("light ON")
            response, content = http.request(url_on, 'GET')
            lightstatus=True
        lightbalance = lightbalance -1       
    else:
        if lightstatus == True:
            print("light OFF")
            response, content = http.request(url_off, 'GET')
            lightstatus=False
 
    # Print remaining light balance     
    print(datetime.timedelta(seconds=lightbalance))

    # Increase balance check counter
    balcheckcount = balcheckcount +1

    # Pause for 1 sec.
    time.sleep(1)
```

The Python source code for this project can be found [here](https://gist.github.com/huggre/08090b9f5ad06707c91b2c0eeba990be)

## Running the project

To run the project, you first need to power up the WiFi module so that it starts listening for HTTP requests from the central computer.

Secondly you need to start the Python program on the central server.

Notice that Python program files uses the .py extension, so let’s save the Python file as **let_there_be_light_wifi.py** on the computer.

To execute the program, simply start a new terminal window, navigate to the folder where you saved *let_there_be_light_wifi.py* and type:

**python let_there_be_light_wifi.py**

You should now see the code being executed in your terminal window, displaying your current light balance and checking the LED’s IOTA address balance for new funds every 10 seconds.

## Pay the light

To turn on the LED you simply use your favorite IOTA wallet and transfer some IOTA’s to the LED’s IOTA address. As soon as the transaction is confirmed by the IOTA tangle, the LED should light up and stay on until the light balance is empty depending on the amount of IOTA’s you transferred. In my example I have set the IOTA/light ratio to be 1 IOTA for 1 second of light.

*Note!*
*If using a mobile wallet to pay the light you may consider printing a QR code that can be scanned for convenience whenever you want to pay the light.*

## Donations

If you like this tutorial and want me to continue making others, feel free to make a small donation to the IOTA address used in the Python code. Also, feel free to use the same IOTA address when building and testing your version of this project, so that whenever my LED (and yours) lights up it gives me (us) a nice reminder that someone else is using this tutorial.

![img](https://miro.medium.com/max/400/1*kV_WUaltF4tbRRyqcz0DaA.png)

> GTZUHQSPRAQCTSQBZEEMLZPQUPAA9LPLGWCKFNEVKBINXEXZRACVKKKCYPWPKH9AWLGJHPLOZZOYTALAWOVSIJIYVZ
