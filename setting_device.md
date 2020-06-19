The next step in the workshop is to provision a new device in AWS IoT Core.

First of all, you should create a new thing to represent your device. Please follow the instructions available in the [Create a thing](https://docs.aws.amazon.com/iot/latest/developerguide/create-aws-thing.html) page. Lets name our thing *GeofencedDevice1*.

Next, we should create a new set of credentials that will correspond to the identity or our device. Please refer to the [Create and activate a device certificate](https://docs.aws.amazon.com/iot/latest/developerguide/create-device-certificate.html) page for detailed instructions. Make sure you download the credential's files.

Once we have the certificate created, we can move and create a new policy to define the actions allowed for the device. You can follow the instructions at [Create an AWS IoT Core policy](https://docs.aws.amazon.com/iot/latest/developerguide/create-iot-policy.html). When creating the statement for the Publish action, lets use the *data/geofencing/GeofencedDevice1/geolocation* topic name.

Finally, lets associate the policy, certificate and device. To do that, please follow the instructions at the [Attach an AWS IoT Core policy to a device certificate](https://docs.aws.amazon.com/iot/latest/developerguide/attach-policy-to-certificate.html) and [Attach a certificate to a thing](https://docs.aws.amazon.com/iot/latest/developerguide/attach-cert-thing.html) pages.

Now we have our device properly configured in AWS IoT Core.
