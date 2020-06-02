---
layout: post
title:  "Room Multi-Sensor: Part 3"
author: ant
categories: [ Room multi-sensor ]
tags: [Room multi-sensor]
image: assets/images/projects/roommultisensor/roommultisensor3-1.jpg
description: "Part 3 of my multi-sensor project"
featured: false
hidden: false
---

## Room Multi-Sensor: Part 3

For Part 3 of this series, we will focus on connecting the multi-sensor to a little rf24 radio module, and using that to send the sensor readings to my OpenHab server. (In my case, that's a Raspberry Pi under my stairs, also attached to a rf24 module, to receive the signals!)

Part of the reason this post has been so delayed is I've been trying to think of the best way to document this process, but the reality it, there is already a post by HomeAutomationForGeeks, which you can find  [HERE](http://homeautomationforgeeks.com/project/rf24hardware.shtml). This is the guide I used religiously when first experimenting with Raspberry Pi and Arduino, and communicating between the 2.

If you follow that guide, you should end up with an Arduino circuit that looks like this:


![multisensor-rf24connection]({{site.baseurl}}/assets/images/projects/plantsensor/plantsensor3-1.jpg)

You may have noticed that the pin connections on my Arduino don't exactly match that of the HAFG tutorial. That's because the SPI pins on the Arduino Mega are in a different place.

**MISO = 50**

**MOSI = 51**

**SCK = 52**

I've also slightly tweaked the C++ code running on my Raspberry Pi, in order to receive broadcasts from more than 1 sensor unit. (In this post, I have 2 units transmitting readings. The eventual aim is to have 1 in each room of my house.)

	if (header.type == '1') {
                // Read the message
                network.read(header, &message, sizeof(message));
                // Print it out in case someone's watching
                printf("Temperature received from node %i: %f \n", header.from_node, message.temperature);
        printf("Humidity received from node %i: %f \n", header.from_node, message.humidity);
        printf("Light received from node %i: %i \n", header.from_node, message.light);
        printf("Motion received from node %i: %i \n", header.from_node, message.motion);
        printf("Door received from node %i: %i \n", header.from_node, message.dooropen);
                // "sprintf" is a way to format a text string and then write he result to char array
                // (an array of characters is virtually the same as a string)
                // The string we're making is a command to push the temperature data on a mosquitto channel
                char buffer [50];
                        sprintf (buffer, "mosquitto_pub -t office/temperature -m \"%f\"", message.temperature);
                        system(buffer);
                        sprintf (buffer, "mosquitto_pub -t office/humidity -m \"%f\"", message.humidity);
                        system(buffer);
                        sprintf (buffer, "mosquitto_pub -t office/light -m \"%i\"", message.light);
                        system(buffer);
                        sprintf (buffer, "mosquitto_pub -t office/motion -m \"%i\"", message.motion ? 1 : 0);
                        system(buffer);
                        sprintf (buffer, "mosquitto_pub -t office/door -m \"%i\"", message.dooropen ? 1 : 0);
                        system(buffer);
            }
      else if (header.type == '2') {
                // Read the message
                network.read(header, &message, sizeof(message));
                // Print it out in case someone's watching
                printf("Temperature received from node %i: %f \n", header.from_node, message.temperature);
        printf("Humidity received from node %i: %f \n", header.from_node, message.humidity);
        printf("Light received from node %i: %i \n", header.from_node, message.light);
        printf("Motion received from node %i: %i \n", header.from_node, message.motion);
        printf("Door received from node %i: %i \n", header.from_node, message.dooropen);
                // "sprintf" is a way to format a text string and then write he result to char array
                // (an array of characters is virtually the same as a string)
                // The string we're making is a command to push the temperature data on a mosquitto channel
                char buffer [50];
                        sprintf (buffer, "mosquitto_pub -t lounge/temperature -m \"%f\"", message.temperature);
                        system(buffer);
                        sprintf (buffer, "mosquitto_pub -t lounge/humidity -m \"%f\"", message.humidity);
                        system(buffer);
                        sprintf (buffer, "mosquitto_pub -t lounge/light -m \"%i\"", message.light);
                        system(buffer);
                        sprintf (buffer, "mosquitto_pub -t lounge/motion -m \"%i\"", message.motion ? 1 : 0);
                        system(buffer);
                        sprintf (buffer, "mosquitto_pub -t lounge/door -m \"%i\"", message.dooropen ? 1 : 0);
                        system(buffer);
            } else {
                // This is not a type we recognize
                network.read(header, &message, sizeof(message));
                printf("Unknown message received from node %i\n", header.from_node);
            }
            

So you can see from this that I've modified it to push sensor readings to different MQTT channels, depending on a number assigned to the unit's header type. The multi-sensor in my office is Unit 1, while the Lounge unit is Unit 2.

Therefore I've had to slightly modify the code on the Arduino to allow this. The sketch for my Lounge unit looks like this:

	//Include the DHT library (Available as DHT Sensor Library to add)
	#include < DHT.h >

	#include < RF24Network.h >
		#include < RF24.h >
	#include < SPI.h >

	//DHT11 (Temp/Humidity sensor) is connected to pin 2 on the Arduino
	#define DHTPIN 2

	//Define the model of DHT sensor (For the DHT library)
	#define DHTTYPE DHT11   // DHT 11 

	DHT dht(DHTPIN, DHTTYPE);

	//Photocell (Light sensor) is connected to Analogue pin 0 (A0) on the Arduino
	byte photocellPin = A0;

	//PIR (Motion sensor) is connected to pin 4 on the Arduino
	byte pirPin = 4;

	//Reed switch (Door sensor) is connected to pin 6 on the Arduino
	byte switchPin = 6;

	// Radio with CE & CSN connected to pins 7 & 8
	RF24 radio(7, 8);
	RF24Network network(radio);

	// Constants that identify this node and the node to send data to
	const uint16_t this_node = 2;
		const uint16_t parent_node = 0;

	// Time between packets (in ms)
	const unsigned long interval = 1000;  // every sec

	// Structure of our message
	struct message_1 {
	  float temperature;
	  float humidity;
	  byte light;
	  bool motion;
	  bool dooropen;
	};
	message_1 message;

	// The network header initialized for this node
	RF24NetworkHeader header(parent_node);

	void setup() {
	// Set up the Serial Monitor
	  Serial.begin(9600);

  
	  // Initialize the DHT library
	  dht.begin();

	  // Calibrate PIR
	  pinMode(pirPin, INPUT);
	  //digitalWrite(pirPin, LOW);
	  Serial.print("Calibrating PIR ");

	  // Activate the internal Pull-Up resistor for the door sensor
	  pinMode(switchPin, INPUT_PULLUP);

    // Initialize all radio related modules
	  SPI.begin();
	  radio.begin();
	  delay(5);
	  network.begin(90, this_node);

	}

	void loop() {

	  // Update network data
	  network.update();
  
	  // Read humidity (percent)
	  float h = dht.readHumidity();
	  // Read temperature as Celsius
	  float t = dht.readTemperature();
	  // Read temperature as Fahrenheit
	  float f = dht.readTemperature(true);

	  // Read photocell
	  int p = analogRead(photocellPin);
	  // Testing revealed this value never goes below 50 or above 1000,
	  //  so we're constraining it to that range and then mapping that range
	  //  to 0-100 so it's like a percentage
	  p = constrain(p, 50, 1000);
	  p = map(p, 50, 1000, 0, 100);

	  // Read motion: HIGH means motion is detected
	  bool m = (digitalRead(pirPin) == HIGH);

		  // Read door sensor: HIGH means door is open (the magnet is far enough from the switch)
	  bool d = (digitalRead(switchPin) == HIGH);

	  //Print the sensor readings to the serial monitor to see they are working correctly
    Serial.print("Temperature: ");
    Serial.print(t);
    Serial.print("\n");
    Serial.print("Humidity: ");
    Serial.print(h);
    Serial.print("\n");
    Serial.print("Farenheit: ");
    Serial.print(f);
    Serial.print("\n");
    Serial.print("Light: ");
    Serial.print(p);
    Serial.print("\n");
    Serial.print("Motion: ");
    Serial.print(m);
    Serial.print("\n");
    Serial.print("Door Open: ");
    Serial.print(d);
    Serial.print("\n");
    Serial.print("\n");


    // We set it again each loop iteration because fragmentation of the messages might change this between loops
    header.type = '2';    

    // Construct the message we'll send
    message = (message_1){ t, h, p, m, d };

    // Writing the message to the network means sending it
    if (network.write(header, &message, sizeof(message))) {
      Serial.print("Message sent\n"); 
      digitalWrite(13, HIGH);   // turn the LED on (HIGH is the voltage level)
      delay(1000);              // wait for a second
      digitalWrite(13, LOW); 
    } 
    else {
      Serial.print("Could not send message\n"); 
    }

	  //Add a 2 second delay to allow reading of the serial output
    delay(2000);

I also added these items to Openhab, and created a simple sitemap to put them on the screen.

## **Items:**

	Number  Node01Temperature   "Temperature [%.1f F]"  <temperature>   (GF_Living)     { mqtt="<[mymosquitto:office/temperature:state:default]" }
	Number  Node01Humidity  "Humidity [%.1f %%]"    <bath>  (GF_Living)     { mqtt="<[mymosquitto:office/humidity:state:default]" }
	Number  Node01Light "Light [%d %%]" <sun>   (GF_Living)     { mqtt="<[mymosquitto:office/light:state:default]" }
	Number  Node01Motion    "Motion [MAP(motion.map):%s]"   <shield>    (GF_Living)     { mqtt="<[mymosquitto:office/motion:state:default]" }
	Number  Node01Door  "Door [MAP(door.map):%s]"   <frontdoor> (GF_Living) { mqtt="<[mymosquitto:office/door:state:default]" }

	Number  Node02Temperature   "Temperature [%.1f F]"  <temperature>   (GF_Living)     { mqtt="<[mymosquitto:lounge/temperature:state:default]" }
	Number  Node02Humidity  "Humidity [%.1f %%]"    <bath>  (GF_Living)     { mqtt="<[mymosquitto:lounge/humidity:state:default]" }
	Number  Node02Light "Light [%d %%]" <sun>   (GF_Living)     { mqtt="<[mymosquitto:lounge/light:state:default]" }
		Number  Node02Motion    "Motion [MAP(motion.map):%s]"   <shield>    (GF_Living)     { mqtt="<[mymosquitto:lounge/motion:state:default]" }
	Number  Node02Door  "Door [MAP(door.map):%s]"   <frontdoor> (GF_Living) { mqtt="<[mymosquitto:lounge/door:state:default]" }

## **Sitemap:**

	sitemap default label="Multi-Sensors"
	{
    Frame label="Office Sensor" {
        Text item=Node01Temperature
        Text item=Node01Humidity
        Text item=Node01Light
        Text item=Node01Motion
        Text item=Node01Door
    }
    Frame label="Lounge Sensor" {
        Text item=Node02Temperature
        Text item=Node02Humidity
        Text item=Node02Light
        Text item=Node02Motion
	        Text item=Node02Door
    }

	}

Now, when both these sensor units start broadcasting, my sitemap looks like this:

![multisensorsitemap]({{site.baseurl}}/assets/images/projects/plantsensor/plantsensor3-2.jpg)

TODO: Still need to calibrate the sensors somehow. Believe it not, those 2 readouts are from units next to each other on my desk. Hmmmmm. Or perhaps I've had a brain-fart and confused my Celsius, Fahrenheit and Humidity readings......