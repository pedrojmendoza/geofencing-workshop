The next step in the workshop is to provision a new device in AWS IoT Core.

1. First of all, lets create a new Thing Type to represent out devices. You can refer to [Thing types](https://docs.aws.amazon.com/iot/latest/developerguide/thing-types.html) for details on how to do it. In our case, we will define a Thing Type names *GeofencedDevice* and will define a single attribute named *AllowedRegions* within it to allow us to define where a given device is expected to be located. You can perform this action by running the following command in the CLI:

```
aws iot create-thing-type --thing-type-name "GeofencedDevice" --thing-type-properties "searchableAttributes=AllowedRegions"
```

2. Next, you should create a new thing to represent your device. Please follow the instructions available in the [Create a thing](https://docs.aws.amazon.com/iot/latest/developerguide/create-aws-thing.html) page. Lets name our thing *GeofencedDevice1* and make sure to select *GeofencedDevice* as the thing type for the new thing. Also set the value for the *AllowedRegions* thing attribute to be *Ottawa* (that should match the geofence's name you created before).

3. Once we have the device in place, we should create a new set of credentials that will correspond to the identity or our device. Please refer to the [Create and activate a device certificate](https://docs.aws.amazon.com/iot/latest/developerguide/create-device-certificate.html) page for detailed instructions. Make sure you download the credential's files and save it on a secure location.

4. Once we have the certificate created, we can move and create a new policy to define the actions allowed for the device. You can follow the instructions at [Create an AWS IoT Core policy](https://docs.aws.amazon.com/iot/latest/developerguide/create-iot-policy.html). When creating the statement for the Publish action, lets use the *data/geofencing/GeofencedDevice1/geolocation* topic name.

5. Finally, lets associate the policy, certificate and device. To do that, please follow the instructions at the [Attach an AWS IoT Core policy to a device certificate](https://docs.aws.amazon.com/iot/latest/developerguide/attach-policy-to-certificate.html) and [Attach a certificate to a thing](https://docs.aws.amazon.com/iot/latest/developerguide/attach-cert-thing.html) pages.

Now we have our device properly configured in AWS IoT Core.
