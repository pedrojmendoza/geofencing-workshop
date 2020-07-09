Now we are ready to simulate a real device. We will use the same [Cloud9](https://aws.amazon.com/cloud9/) environment as we used for deploying the geofences web app.

On your Cloud9 env, open a Terminal window and follow the steps below:

1. Install AWS IoT SDK for Python -> `sudo pip install AWSIoTPythonSDK`

2. Create a new file, paste the content below on it and save it as *spatialPub.py*

        ```
        from AWSIoTPythonSDK.MQTTLib import AWSIoTMQTTClient
        import logging
        import time
        import argparse
        import json
        import random

        # Read in command-line parameters
        parser = argparse.ArgumentParser()
        parser.add_argument("-e", "--endpoint", action="store", required=True, dest="host", help="Your AWS IoT custom endpoint")
        parser.add_argument("-r", "--rootCA", action="store", required=True, dest="rootCAPath", help="Root CA file path")
        parser.add_argument("-c", "--cert", action="store", dest="certificatePath", help="Certificate file path")
        parser.add_argument("-k", "--key", action="store", dest="privateKeyPath", help="Private key file path")
        parser.add_argument("-id", "--clientId", action="store", dest="clientId", default="GeofencedDevice1", help="Targeted client id")
        parser.add_argument("-t", "--topic", action="store", dest="topic", default="data/geofencing/GeofencedDevice1/geolocation", help="Targeted topic")

        args = parser.parse_args()
        host = args.host
        rootCAPath = args.rootCAPath
        certificatePath = args.certificatePath
        privateKeyPath = args.privateKeyPath
        clientId = args.clientId
        topic = args.topic

        if not args.certificatePath or not args.privateKeyPath:
            parser.error("Missing credentials for authentication.")
            exit(2)

        # Port defaults
        port = 8883

        # Configure logging
        logger = logging.getLogger("AWSIoTPythonSDK.core")
        logger.setLevel(logging.DEBUG)
        streamHandler = logging.StreamHandler()
        formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
        streamHandler.setFormatter(formatter)
        logger.addHandler(streamHandler)

        # Init AWSIoTMQTTClient
        myAWSIoTMQTTClient = AWSIoTMQTTClient(clientId)
        myAWSIoTMQTTClient.configureEndpoint(host, port)
        myAWSIoTMQTTClient.configureCredentials(rootCAPath, privateKeyPath, certificatePath)

        # AWSIoTMQTTClient connection configuration
        myAWSIoTMQTTClient.configureAutoReconnectBackoffTime(1, 32, 20)
        myAWSIoTMQTTClient.configureOfflinePublishQueueing(-1)  # Infinite offline Publish queueing
        myAWSIoTMQTTClient.configureDrainingFrequency(2)  # Draining: 2 Hz
        myAWSIoTMQTTClient.configureConnectDisconnectTimeout(10)  # 10 sec
        myAWSIoTMQTTClient.configureMQTTOperationTimeout(5)  # 5 sec

        # Connect and subscribe to AWS IoT
        myAWSIoTMQTTClient.connect()
        time.sleep(2)

        # Publish to the same topic in a loop forever
        loopCount = 0
        while True:
            message = {}

            # Ottawa neighbourhood
            message['lon'] = random.randrange(-7584, -7540)/100
            message['lat'] = random.randrange(4530, 4553)/100

            message['sequence'] = loopCount
            messageJson = json.dumps(message)
            myAWSIoTMQTTClient.publish(topic, messageJson, 1)
            print('Published topic %s: %s\n' % (topic, messageJson))
            loopCount += 1
            time.sleep(5)
        ```
        
3. Copy the certificate and private key for the device to the same folder in your Cloud9 env where you have created the above file. Make sure you rename the file names to be *GeofencedDevice1.cert.pem* and *GeofencedDevice1.private.key*       

4. Download the root CA for the AWS IoT Core's certificate -> `curl https://www.amazontrust.com/repository/AmazonRootCA1.pem > root-CA.crt`

5. Start the device simulation (make sure you replace the placeholder with the correct endpoint from your AWS IoT Core, including region) -> `python spatialPub.py -e <YOUR-AWS-IOT-CORE-ENDPOINT> -r root-CA.crt -c GeofencedDevice1.cert.pem -k GeofencedDevice1.private.key`
