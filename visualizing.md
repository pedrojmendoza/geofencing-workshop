Once we have the data flowing into IoT Core and the spatial query components in place, the next step is to extend our rules to include the ability to visualize the geolocation of these devices.

1. Next, we can extend the IoT Rule we created before to also send the data coming from the devices (and enriched with geofencing information) to our just created IoT Analytics flow

    1.1. Open the [IoT console](https://console.aws.amazon.com/iot/)

    1.2. Click on *Act* and then on *Rules*
    
    1.3. Click on the previously created rule named *Geofencing*
    
    1.4. Click on *Add action* (under the *Set one or more actions* section) and select the *Send a message to IoT Analytics* action and click on *Configure action*.
    
    1.5. Select *Quick create IoT Analytics resources*, enter *geofencing* for *Resources prefix* and click on *Quick create*
    
    1.6. Click on *Add action*

2. Now, we can configure the dataset being used for visualization.

    2.1. Select the *Data sets* section and click on the just created dataset (named *geofencing_dataset*).

    2.2. Click on *Edit* next to the *SQL query* section.
    - Paste `SELECT device, timestamp, lat, lon, geofencing_result.inside_a_valid_region FROM Geofencing_datastore` into the *Query* input
    - Click on *Save*

    2.3. Click on *Edit* next to the *Delta window* section and enter the following values.
    - Select *Delta time* from the *Data selection window* dropdown
    - Enter 300 in the *Offset* input
    - Enter *from_unixtime(timestamp)* in the *Timestamp expression* input 
    - Click on *Save*
  
    2.4. Click on *Add schedule* next to the *Schedule* section and enter the following values.  
    - Select *Every 5 minutes* from the dropdown and click
    - Click on *Save*
    
3. Finally, lets configure the visualization using QuickSight ...

<TODO>

