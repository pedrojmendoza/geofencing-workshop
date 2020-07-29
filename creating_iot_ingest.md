The next section is about routing the messages coming from the devices and figuring out if the device's position is within an authorized geofence or not.

Lets first propagate the geofences geometries from the DDB table where our webapp's API has stored these into an S3 bucket that can be accessed by our geospatial queries system.

1. Create a new S3 bucket, make sure you create it on the same region you created the previous resources and select a globally unique name for your bucket. You can follow the instructions at [Creating a bucket](https://docs.aws.amazon.com/AmazonS3/latest/gsg/CreatingABucket.html)

2. We will need a baseline JSON file to store the geofences geometries. Initially it will only contain some metadata and will be incrementally extended as the new geofences are added using the webapp. 

    2.1. Open the [S3 console](https://s3.console.aws.amazon.com/s3). 

    2.2. Click on your bucket name (the one you just created) to open it.

    2.3. Click on *Create Folder*, name it *Canada* and click *Save*

    2.4. The baseline file is part of this repository and is named *regions.json*, you can download it by running `wget https://raw.githubusercontent.com/pedrojmendoza/geofencing-workshop/master/regions.json` and then upload it to your S3 bucket (inside the *Cananda* folder) by dragging and dropping it.

3. Next, create a new IAM role for the lambda function that will be performing the data propagation.

    3.1. Open the [roles page](https://console.aws.amazon.com/iam/home?#/roles) in the IAM console.

    3.2. Choose Create role.

    3.3. Create a role with the following properties.
    - Trusted entity – Lambda.
    - Permissions – *AWSLambdaDynamoDBExecutionRole* and *AmazonS3FullAccess* (please note that these permissions are not recommended for a PROD environment as these are too permissive).
    - Role name – *lambda-dynamodb-s3-role*

4. Now, lets create a new Lambda function.

    4.1. Open the [Lambda console](https://console.aws.amazon.com/lambda/)

    4.2. Choose *Create function* and use the following parameters.
    - Enter *DdbToS3ForSpatialQuerying* as function name.
    - Select *Python 3.8* as Runtime. 
    - Select *Use an existing role*, pick the *lambda-dynamodb-s3-role* from the dropdown and click on *Create function*

    4.3. Once the function is created, replace its code with the below code, update the <REPLACE_WITH_YOUR_BUCKET_NAME> literal with the S3 bucket you created above and click on *Save* (make sure you remove any leading space on the pasted code).

       import boto3
       import json

       print('Loading function')

       def lambda_handler(event, context):
         print("Received event: " + json.dumps(event, indent=2))

         s3 = boto3.resource('s3')
         regions_s3_object = s3.Object('<REPLACE_WITH_YOUR_BUCKET_NAME>', 'Canada/regions.json')
         regions = json.loads(regions_s3_object.get()['Body'].read().decode('utf-8'))

         for record in event['Records']:
           if 'NewImage' in record['dynamodb']:
             new_name = record['dynamodb']['NewImage']['name']['S']
             new_geometry = json.loads(record['dynamodb']['NewImage']['geometry']['S'])

             lons_lats = []
             for lat_lon in new_geometry:
               lon_lat = []
               lon_lat.append(lat_lon[1])
               lon_lat.append(lat_lon[0])
               lons_lats.append(lon_lat)

             new_rings = []
             new_rings.append(lons_lats)

             new_feature = {}
             new_feature['attributes'] = {'NAME': new_name}
             new_feature['geometry'] = {'rings': new_rings}
             regions['features'].append(new_feature)

         #print("extended regions: " + json.dumps(existing_regions, indent=2))
         regions_s3_object.put(Body=(bytes(json.dumps(regions).encode('UTF-8'))))

         return True

    4.4. Finally, connect your lambda with the DDB table.
    - Click on *+ Add trigger* and select the *DynamoDB* trigger configuration from the dropdown.
    - Select the geofences table (should start with *Geofence-* and click on *Add*.

5. Lets define a new geofence using the web app we created before

    5.1. Go to the URL for the published application.
    
    5.2. Create a new user to be able to access the application.
    
    5.3. Once in the application, draw a new geofence covering the Ottawa area (close to what is shown in this [picture](./Ottawa.png)) and save it with the name *Ottawa*

6. Now that we have our data syncronized in S3, we can proceed and create the Athena resources to point to the S3 object with the geometries so we can execute queries against it.

    6.1. Go to the [Athena console](https://console.aws.amazon.com/athena/)

    6.2. In order for Athena to be able to properly operate, you need to provide a location for query results. Click on *set up a query result location in Amazon S3* and enter the S3 URI of your bucket like the following *s3://<REPLACE_WITH_YOUR_BUCKET_NAME>/*

    6.3. Enter the following code in the query editor (make sure you replace the placeholder with your bucket name).

        CREATE external TABLE IF NOT EXISTS regions
         (
         NAME string,
         BoundaryShape binary
         )
        ROW FORMAT SERDE 'com.esri.hadoop.hive.serde.JsonSerde'
        STORED AS INPUTFORMAT 'com.esri.json.hadoop.EnclosedJsonInputFormat'
        OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
        LOCATION 's3://<REPLACE_WITH_YOUR_BUCKET_NAME>/Canada/';

    6.4. Click on *Run query*

7. Similarly to step 3, now lets create a new IAM role for the lambda function that will be performing the spatial querying.

    7.1. Open the [roles page](https://console.aws.amazon.com/iam/home?#/roles) in the IAM console.

    7.2. Choose Create role.

    7.3. Create a role with the following properties.
    - Trusted entity – Lambda.
    - Permissions – *AmazonS3FullAccess*, *AmazonAthenaFullAccess*, *AWSIoTConfigReadOnlyAccess* and *AWSGlueConsoleFullAccess* (please note that these permissions are not recommended for a PROD environment as these are too permissive).
    - Role name – *lambda-spatial-query-role*.

8. We are almost done, next step is create a new lambda function for querying the geofences using as input the coordinates of a device.

    8.1. Open the [Lambda console](https://console.aws.amazon.com/lambda/)

    8.2. Choose Create function and use the following parameters.
    - Enter *SpatialQuery* as function name.
    - Select *Python 3.8* as Runtime. 
    - Select *Use an existing role*, pick the *lambda-spatial-query-role* from the dropdown and click on *Create function*

    8.3. Once the function is created, replace its code with the below code, update the <REPLACE_WITH_YOUR_BUCKET_NAME> literal with the S3 bucket you created above and click on *Save* (make sure you remove any leading space on the pasted code).

       import time
       import boto3

       # number of retries
       RETRY_COUNT = 10

       def lambda_handler(event, context):
         print(event)

         # get input
         device = event['device']
         lon = event['lon'] #lon = -75.49
         lat = event['lat'] #lat = 45.4

         # created query
         query = "SELECT regions.name FROM default.regions WHERE ST_CONTAINS (regions.boundaryshape, ST_POINT(%s, %s))" % (lon, lat)

         # athena client
         athena = boto3.client('athena')

         # Execution
         response = athena.start_query_execution(
           QueryString=query,
           QueryExecutionContext={
             'Database': "default"
           },
           ResultConfiguration={
             'OutputLocation': 's3://<REPLACE_WITH_YOUR_BUCKET_NAME>',
           }
         )

         # get query execution id
         query_execution_id = response['QueryExecutionId']
         #print(query_execution_id)

         # get execution status
         for i in range(1, 1 + RETRY_COUNT):
           # get query execution
           query_status = athena.get_query_execution(QueryExecutionId=query_execution_id)
           query_execution_status = query_status['QueryExecution']['Status']['State']

           if query_execution_status == 'SUCCEEDED':
             #print("STATUS:" + query_execution_status)
             break

           if query_execution_status == 'FAILED':
             raise Exception("STATUS:" + query_execution_status)

           else:
             #print("STATUS:" + query_execution_status)
             time.sleep(0.2)
          
         else:
           athena.stop_query_execution(QueryExecutionId=query_execution_id)
           raise Exception('TIME OVER')

         # get query results
         result = athena.get_query_results(QueryExecutionId=query_execution_id)
         #print(result)

         # get data
         region = {}
         if len(result['ResultSet']['Rows']) == 2:
           region['name'] = result['ResultSet']['Rows'][1]['Data'][0]['VarCharValue'] # region's name
           print("Device found inside a region: " + str(region))
           
           iot = boto3.client('iot')
           describe_thing_response = iot.describe_thing(thingName=device)
           allowed_regions = describe_thing_response['attributes']['AllowedRegions']
           print("Allowed regions for device: " + allowed_regions)

           if region['name'] in allowed_regions:
             print("Device found inside a valid region")
             region['inside_a_valid_region'] = True
           else:
             print("Device found outside a valid region")
             region['inside_a_valid_region'] = False
         else:
           print("Device not found inside any region")    
           region['inside_a_valid_region'] = False

         return region

    8.4. Click *Edit* on the *Basic settings* section and increase the Timeout to be *30* seconds.

9. Finally, create a new rule in IoT Core

    9.1. Open the [IoT console](https://console.aws.amazon.com/iot/)

    9.2. Click on *Act* and then on *Rules*

    9.3. Click on *Create* and enter the following values.
    - Name - *Geofencing*
    - Rule query statement, enter the below query. Make sure to replace the region and account number placeholders with the correct one (you can also copy and paste the ARN of the function created in 7). - `SELECT topic(3) as device, timestamp()/1000 as timestamp, lon, lat, aws_lambda("arn:aws:lambda:<REPLACE_WITH_YOUR_REGION>:<REPLACE_WITH_YOUR_ACCOUNT_NUMBER>:function:SpatialQuery", {"device":topic(3),"lon":lon,"lat":lat}) as geofencing_result FROM 'data/geofencing/+/geolocation'`
    - Click on *Add actions* (under the *Set one or more actions* section) and select the *Republish to an AWS IoT topic* action.
    - Click on *Configure action* and enter *data/geofencing/processed* as destination topic.
    - Click on *Create Role* and enter *iot-republish* as role name.
    - Click on *Add action* and finally on *Create rule*
