---
layout: post
title:  "Part Three - Storing data in Firebase and Google Cloud"
author: ant
categories: [ Cloud connected multi-sensor ]
tags: [Cloud Connected multi-sensor]
image: assets/images/projects/cloudconnectedroomsensor/cloudconnectedroomsensor3-3.png
description: "Cloud connected room sensor - part 3"
featured: true
hidden: false
---

## Part Three - Storing data in Firebase and Google Cloud 


Now now we’re sending sensor data to Google Pubsub, we need to create a table in BigQuery, and create Firebase Functions to push messages from the PubSub subscription to it to store.

First, you’ll need to go to your [Google Cloud Console](https://console.cloud.google.com/), navigate to BigQuery, and create a new dataset. I’ve called mine _room_sensor_data_. Then, you’ll need to create a table, with a field for each piece of sensor data I’m sending from my sensor units.

![create BigQuery Table.PNG]({{site.baseurl}}/assets/images/projects/cloudconnectedroomsensor/cloudconnectedroomsensor3-1.png)

Now, go to the [Firebase console](https://console.firebase.google.com/), and create a new Project, and add Firebase to your Google Cloud project:

![Create Firebase project.PNG]({{site.baseurl}}/assets/images/projects/cloudconnectedroomsensor/cloudconnectedroomsensor3-2.png)
  
Next, you’ll need to follow the steps listed [here](https://firebase.google.com/docs/cli/) to install Firebase Command Line Tools and initialise the project in Firebase.

**Note: Make sure you run these commands from the folder you wish to contain your Firebase code!**

Once you’ve initialised your project, run the following command in the Firebase CLI:

	firebase functions:config:set 
	bigquery.datasetname="room_sensor_data"
	bigquery.tablename="sensor_data”

Obviously, you’ll need to amend those lines to use the names of your BigQuery Dataset and table you created earlier.  
Next, you’ll need to create a ‘functions’ sub-directory in the folder you’re running Firebase CLI from. Inside that Functions folder, create a file called index.js, and add the following code to it:

	const functions = require('firebase-functions');
	const admin = require('firebase-admin');
	const bigquery = require('@google-cloud/bigquery')();
	const cors = require('cors')({ origin: true });

	admin.initializeApp(functions.config().firebase);

	const db = admin.database();

	/**
	 * Receive data from pubsub, then 
	 * Write telemetry raw data to bigquery
	 */
	exports.pubsubReceiveTelemetry = functions.pubsub
	  .topic('room-sensors')
	  .onPublish((message, context) => {
	    const attributes = message.attributes;
	    const payload = message.json;

	    const deviceId = attributes['deviceId'];

	    const data = {
	      humidity: payload.hum,
	      temp: payload.temp,
	      light: payload.light,
	      deviceId: deviceId,
	      timestamp: context.timestamp
	    };

	    if (
	      payload.hum < 0 ||
	      payload.hum > 100 ||
	      payload.temp > 100 ||
	      payload.temp < -50 ||
	      payload.pir > 1 ||
	      payload.pir < 0 ||
	      payload.light > 100 ||
	      payload.light < 0
	    ) {
	      // Validate and do nothing
	      return;
	    }

	    return Promise.all([
	      insertIntoBigquery(data),
	      updateCurrentDataFirebase(data)
	    ]);
	  });

	/** 
	 * Maintain last status in firebase
	*/
	function updateCurrentDataFirebase(data) {
	  return db.ref(`/devices/${data.deviceId}`).set({
	    humidity: data.humidity,
	    temp: data.temp,
	    light: data.light,
	    lastTimestamp: data.timestamp
	  });
	}

	/**
	 * Store the data in bigquery
	 */
	function insertIntoBigquery(data) {
	  const dataset = bigquery.dataset(functions.config().bigquery.datasetname);
	  const table = dataset.table(functions.config().bigquery.tablename);

	  return table.insert(data);
	}

What we’re doing with this function is firstly initialising the Firebase connection, using the config values we set previously. We’re then effectively adding a trigger to our PubSub topic, which runs whenever a message is published to it.  
We create an object called data, which adds the sensor values to the deviceId and a timestamp of when the message was published . Storing these additional value to BigQuery will help with querying and reporting on the data in the future.

We then do a quick check on the sensor values, to make sure they are within an acceptable range. For example, we want to check that the temperature value is between between 0 and 100 degrees Celsius. Any value outside of this range means something has gone wrong with the sensor, therefore we don’t save it to BigQuery as where possible, we only want to store correct data values.

We then call 2 methods: updateCurrentDataFirebase, which sends this latest reading to Firebase Datastore, and insertIntoBigquery, which adds the data as a new row in out BigQuery table created earlier.

You’ll notice that I'm not referencing any motion values in these functions. Something seems to have gone wrong with the PIR sensor I’m using. So until I get that fixed, I don’t really want to save those values anywhere.

If you save your index.js file, and then from within Firebase Command Line Tools run the following command:

	firebase deploy

You should then see the functions being deployed to your Firebase project.

From here, you can go to the [Firebase Console](https://console.firebase.google.com/), navigate to Database, to see the latest values received by your updateCurrentDataFirebase method:

![Firebase latest values.PNG]({{site.baseurl}}/assets/images/projects/cloudconnectedroomsensor/cloudconnectedroomsensor3-3.png)

And from BigQuery, you can now run the following query to see data stored in your table:

	SELECT * FROM  `henderson-home-iot.room_sensor_data.sensor_data`  order by timestamp desc LIMIT 1000

Obviously, you’ll need to use your own project name, as well as the values for your BigQuery dataset and table. But once you do that, you should see some results like so…..

![BigQuery data.PNG]({{site.baseurl}}/assets/images/projects/cloudconnectedroomsensor/cloudconnectedroomsensor3-4.png)