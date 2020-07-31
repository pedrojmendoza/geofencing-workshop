So, you are almost there ... final step in the workshop is extending our IoT rules to notify the relevant parties when a device is outside its expected geofence.

1. Lets create a new rule in IoT Core to react to msgs on the processed topic.

    1.1. Open the [IoT console](https://console.aws.amazon.com/iot/)

    1.2. Click on *Act* and then on *Rules*

    1.3. Click on *Create* and enter the following values.
    - Name - *GeofencingAlarm*
    - Rule query statement, enter the below query. - `SELECT * FROM 'data/geofencing/processed' WHERE NOT geofencing_result.inside_a_valid_region`
    - Click on *Add actions* (under the *Set one or more actions* section) and select the *Republish to an AWS IoT topic* action.
    - Click on *Configure action* and enter *data/geofencing/alarms* as destination topic.
    - Click on *Select Role* and enter *iot-republish* as role name.
    - Click on *Add action*
    - Click again on *Add action* and select the *Send a message as an SNS push notification* action.
    - Click on *Configure action* and, under the *SNS target* section, click on *Create* to create a new SNS topic.
    - Enter *GeofencingAlarms* as topic name and click on *Create*.
    - Select *JSON* as *Message format*
    - Click on *Create role*, enter *iot-sns* as role name and click on *Create role*.
    - Click on *Add action*
    - Click on *Create rule*
    
2. Next step is adding a subscription to our just-created SNS topic to send notifications via email.

    2.1. Open the [SNS console](https://console.aws.amazon.com/sns/)
    
    2.2. Click on *Topics* and then click on the topic we just created (*GeofencingAlarms*).
    
    2.3. Click on *Create subscription*, select *Email* under *Protocol* dropdown, enter your email address in the *Endpoint* input and click on *Create subscription*.
    
    2.4. You should receive an email asking for confirmation on the just created-subscription, click on othe link and your subscription should be ready (please keep an eye on yoour Junk email rules as it might be wrongly categorized).
    
    2.5. Next time the device is outside its geofence, a new email will be send to your email address.
    
 Congrats, you have reached the end of the workshop!
    
    

 
