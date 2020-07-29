The next step in the workshop is to provision a new device in AWS IoT Core.

1. First of all, lets create a new Thing Type to represent out devices. You can refer to [Thing types](https://docs.aws.amazon.com/iot/latest/developerguide/thing-types.html) for details on how to do it. In our case, we will define a Thing Type names *GeofencedDevice* and will define a single attribute named *AllowedRegions* within it to allow us to define where a given device is expected to be located. You can perform this action by running the following command in the CLI:

```
aws iot create-thing-type --thing-type-name "GeofencedDevice" --thing-type-properties "searchableAttributes=AllowedRegions"
```

2. Next, you should create a new thing to represent your device. 

    2.1. Open the [IoT console](https://console.aws.amazon.com/iot/)

    2.2. Click on *Manage* and then on *Things*

    2.3. Click on *Create*, then on *Create a single thing* and enter the following values.
    - Name - *GeofencedDevice1*
    - Thing Type - *GeofencedDevice* as the thing type for the new thing. 
    - Set the value for the *AllowedRegions* thing's attribute to be *Ottawa* (that should match the geofence's name you created before).
    
    2.4. Click on *Create certificate* and make sure to download and save the certicate and private key files on a secure location.
    
    2.5. Make sure to click the *Activate* button to make the certificate active.
    
    2.6. Click on the *Attach a policy* button and, without selecting any policy, click on *Register Thing*

3. Once we have the thing and its certificate is created, you can move and create a new policy to define the actions allowed for the device. 

    3.1. Open the [IoT console](https://console.aws.amazon.com/iot/)

    3.2. Click on *Secure* and then on *Policies*

    3.3. Click on *Create* and enter the following values.
    - Name - *GeofencedDevice1-Policy*
    - Switch to *Advanced mode* and replace the policy document with the below definition (please adjust it to your AWS account number and selected region)
    
        {
          "Version": "2012-10-17",
          "Statement": [
            {
              "Effect": "Allow",
              "Action": [
                "iot:Publish",
                "iot:Receive"
              ],
              "Resource": [
                "arn:aws:iot:<YOUR_REGION>:<YOUR_AWS_ACCOUNT_ID>:topic/data/geofencing/GeofencedDevice1/geolocation"
              ]
            },
            {
              "Effect": "Allow",
              "Action": [
                "iot:Connect"
              ],
              "Resource": [
                "arn:aws:iot:<YOUR_REGION>:<YOUR_AWS_ACCOUNT_ID>:client/GeofencedDevice1"
              ]
            }
          ]
        }   
        
    3.4. Click on *Create* to finish with policy creation.        

4. Finally, lets associate the policy with the certificate associated with the device.

    4.1. Open the [IoT console](https://console.aws.amazon.com/iot/)

    4.2. Click on *Manage* and then on *Things*

    4.3. Select *GeofencedDevice1* thing by clicking on its name.
    
    4.4. Click on *Security* and then click on the certificate listed
    
    4.5. Click on the *Actions* drop down menu and select *Attach policy*
    
    4.6. Check on the checkbox next to *GeofencedDevice1-Policy* and click on *Attach*

Now we have our device properly configured in AWS IoT Core.
