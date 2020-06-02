---
layout: post
title:  "Room Multi-Sensor: Part 1"
author: ant
categories: [ Room multi-sensor ]
tags: [Room multi-sensor]
image: assets/images/projects/roommultisensor/roommultisensor2-6.jpg
description: "Part 1 of my multi-sensor project"
featured: false
hidden: false
---

## Room Multi-Sensor: Part 1

I've mentioned in a previous post on this blog that about half the job of creating a home automation system is essentially an issue of data gathering. That is, harvesting information from around the house, which can be used later to govern rules/automation, or simply help the user/certain devices to make decisions.  
 <!--more-->
With this in mind, I want to create some sort of multi-sensor, which I can place in rooms around my house, to provide useful data about that room. Specifically:

1.  **Temperature**: That's Home Automation law isn't it? If it has a circuit, it'll have a temperature sensor in it somewhere.
2.  **Humidity**: See above. Temperature and humidity data will be useful when dealing with thermostats, as well as helping to issue reminders when doors and windows might need shutting.
3.  **Light**: Will give a readout of the current light level, which will be useful when programming lighting rules analyzing sleep patterns and such like
4.  **Motion**: Not something I'm planning on utilizing immediately, but could be valuable i integrating security into the system later.
5.  **Door**: Similar to motion sensor, will provide more use in the future, but it's worth including it at this stage, to avoid a major hassle down the line.

## Existing Solutions

There are products out there which could provide similar features, such as:

[](https://www.amazon.co.uk/Philio-Tech-PST02-1C-Illumination-Temperature/dp/B00SV5JCTG/)

![Philio Tech PST02-1C](https://images.squarespace-cdn.com/content/v1/56f6894837013b5f81836f61/1465982218048-ZCOB5XS0YN7619MO0MBS/ke17ZwdGBToddI8pDm48kODEHMGUBRgRRplOmqRomK1Zw-zPPgdn4jUwVcJE1ZvWhcwhEtWJXoshNdA9f1qD7Xj1nVWs2aaTtWBneO2WM-vKKmLGZcam4wNLVFkW8ocU3wCuzBZdxfWEqO227KYmPQ/Philio+Tech+PST02-1C?format=300w)

## Philio Tech Z-Wave 3-in-1 Multi-Sensor

[Amazon](https://www.amazon.co.uk/Philio-Tech-PST02-1C-Illumination-Temperature/dp/B00SV5JCTG/)

**Sensors:**  Temperature, Light, Door

**Price:**  £35

[](https://www.amazon.co.uk/Aeon-Labs-Multisensor-Z-Wave-Plus/dp/B0141FQDJQ/)

![Aeon Labs Multisensor 6](https://images.squarespace-cdn.com/content/v1/56f6894837013b5f81836f61/1465982253111-KFGKJ8LZ5TEPYKC7KK6F/ke17ZwdGBToddI8pDm48kODEHMGUBRgRRplOmqRomK1Zw-zPPgdn4jUwVcJE1ZvWhcwhEtWJXoshNdA9f1qD7Xj1nVWs2aaTtWBneO2WM-vKKmLGZcam4wNLVFkW8ocU3wCuzBZdxfWEqO227KYmPQ/Aeon+Labs+Multisensor+6?format=300w)

## Aeon Labs Multisensor 6

[Amazon](https://www.amazon.co.uk/Aeon-Labs-Multisensor-Z-Wave-Plus/dp/B0141FQDJQ/)

**Sensors:**  Temperature, Humidity, Light, Motion

**Price:**  £45

So while I could just buy a solution off the shelf, I reckon a cost of around £40 per unit is a little high, especially when you want to put several of them around the house. And besides, where is the fun in that?

## **NEXT:**  Attaching sensors to an Arduino.