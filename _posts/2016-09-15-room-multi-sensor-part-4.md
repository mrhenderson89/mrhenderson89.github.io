---
layout: post
title:  "Room Multi-Sensor: Part 4"
author: ant
categories: [ Room multi-sensor ]
tags: [Room multi-sensor]
image: assets/images/projects/roommultisensor/roommultisensor4-1.jpg
description: "Part 4 of my multi-sensor project"
featured: false
hidden: false
---

## Room Multi-Sensor: Part 4

Just a quick update this time, as I decided to modify some of my code to add a time-stamp to the sensor updates. I'm hoping this will come in handy when I start trying to run these units on battery power. A time readout for the last time a unit transmitted sensor readings will allow me to see exactly how the batteries last, and how much impact I can make with energy-saving enhancements down the line.

So in order to add these to OpenHab, I made the following changes to the main() method of the C++ script on the Raspberry Pi:

      time_t rawtime;
      struct tm * timeinfo;
      
      time ( &rawtime );
      timeinfo = localtime ( &rawtime );

**NOTE:** Make sure you include <time.h> and < string > at the top of the file.

Then I added the following to the lines publishing the sensor readings to MQTT:

	sprintf (buffer, "mosquitto_pub -t office/updateTime -m \"%s\"", asctime(timeinfo));
	system(buffer);

	sprintf (buffer, "mosquitto_pub -t lounge/updateTime -m \"%s\"", asctime(timeinfo));
	system(buffer);

I then added a few extra items to OpenHab:

	String  Node01LastUpdate    "Last Updated : [%s]"   (GF_Living) { mqtt="<[mymosquitto:office/updateTime:state:default]" }

	String  Node02LastUpdate    "Last Updated : [%s]"   (GF_Living) { mqtt="<[mymosquitto:lounge/updateTime:state:default]" }

Also don't forget to add the items to your sitemap.

So now when I run this script on the Pi (After re-running the Makefile again!), I have a sitemap which looks like this:


![sitemaptimestampscreenshot]({{site.baseurl}}/assets/images/projects/plantsensor/plantsensor4-1.jpg)

So if you've followed the previous steps in these blog posts, this should all be fairly straight forward. Although if you are lost, or I've missed some step or other, feel free to leave a comment or get in touch, and I'll try and help out.