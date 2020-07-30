Once we have the data flowing into IoT Core and the spatial query components in place, the next step is to extend our rules to include the ability to visualize the geolocation of these devices.

1. Next, we can extend the IoT Rule we created before to also send the data coming from the devices (and enriched with geofencing information) to our just created IoT Analytics flow

    1.1. Open the [IoT console](https://console.aws.amazon.com/iot/)

    1.2. Click on *Act* and then on *Rules*
    
    1.3. Click on the previously created rule named *Geofencing*
    
    1.4. Click on *Add action* and select the *Send a message to IoT Analytics* action and click on *Configure action*.
    
    1.5. Select *Quick create IoT Analytics resources*, enter *geofencing* for *Resources prefix* and click on *Quick create*
    
    1.6. Click on *Add action*

2. Now, we can configure the dataset being used for visualization.

    2.1. Open the [IoT Analytics console](https://console.aws.amazon.com/iotanalytics/)

    2.2. Select the *Data sets* section and click on the just created dataset (named *geofencing_dataset*).

    2.3. Click on *Edit* next to the *SQL query* section.
    - Paste `SELECT device, timestamp, lat, lon, geofencing_result.inside_a_valid_region FROM geofencing_datastore` into the *Query* input
    - Click on *Save*

    2.4. Click on *Edit* next to the *Delta window* section and enter the following values.
    - Select *Delta time* from the *Data selection window* dropdown
    - Enter *-60* in the *Offset* input
    - Enter *from_unixtime(timestamp)* in the *Timestamp expression* input 
    - Click on *Save*
  
    2.5. Click on *Add schedule* next to the *Schedule* section and enter the following values.  
    - Select *Every 1 minute* from the dropdown and click
    - Click on *Save*
    
    2.6. Click on *Actions* and then on *Run now* to get the dataset initially populated
    
3. Finally, lets configure the visualization using QuickSight.

    3.1. Open the [QuickSight console](https://quicksight.aws.amazon.com/)
    
    3.2. Click on the icon on the upper/right corner to display the drop-down menu and select the region where youo have deployed your IoT Analytics dataset.
    
    3.2. Click again on the icon on the upper/right corner to display the drop-down menu and select *Manage QuickSight*.
    
    3.3. Click on *SPICE capacity* and then click on *Purchase more capacity*.
    
    3.4. Enter 1 GB and click on *Purchase SPICE capacity*.
    
    3.5. Click on the QuickSight logo on the upper/left corner to go back to landing page.

    3.6. Click on *Manage data* and then on *New dataset*.
    
    3.7. Click on *AWS IoT Analytics*.
    
    3.8. Select the *geofencing_dataset* and click on *Create data source*.
    
    3.9. Click on *Visualize*.
    
    3.10. Within the *Visual types* section, select the *Points on map* visualization.
    
    3.11. Drag and drop the *lat* and *lon* fields from the *Fields list* and into the *Geospatial* container on the right.
    
    3.12. Drag and drop the *inside_a_valid_region* field from the *Fields list* and into the *Color* container on the right.
    
    3.13. You should get a map similar to [this](QuickSightMap.png) rendering points near by the Ottawa region mixing both positions inside the geofence as well a outside (with different colours).
    
    

