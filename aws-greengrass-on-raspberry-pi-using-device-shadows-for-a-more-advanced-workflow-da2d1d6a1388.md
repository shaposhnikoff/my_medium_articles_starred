Unknown markup type 10 { type: [33m10[39m, start: [33m50[39m, end: [33m65[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m57[39m, end: [33m72[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m74[39m, end: [33m89[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m55[39m, end: [33m66[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m61[39m, end: [33m72[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m16[39m, end: [33m46[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m50[39m, end: [33m81[39m }

# AWS Greengrass on Raspberry Pi: Using Device Shadows For a More Advanced Workflow

In a previous post, we created a Greengrass group with a sensor device/process that communicates with the core device. The core device processes the data and communicates with the cloud. In this post we’ll be focusing more on the workflow for managing the devices.

The original demo script starts sensor readings the minute we run it (and stops after a finite time, or when the process is manually killed). It also requires manual ssh into the node device to start/stop those scripts.

In a practical application, we would need a more resilient and centralized workflow to manage start/stop of readings [and more states]. You would consider sending those commands from the greengrass core, from the cloud, and eventually a user interface in an enterprise application.

The AWS IoT Thing Shadows service allow us to manage device states through the MQTT messages we’re already working with, even all the way from the cloud. It has some helpful features already built into AWS instead of having to create a custom state-system.

### Set up the start/stop functionality on the sensor, using Thing Shadows

Update the sensor.py script in the Sensor Pi ssh:

    import json
    import logging
    import random
    import time
    import uuid
    from AWSIoTPythonSDK.MQTTLib import AWSIoTMQTTShadowClient  # <Changed the import
    from AWSIoTPythonSDK.core.greengrass.discovery.providers import DiscoveryInfoProvider
    
    ### SAME AS BEFORE ###
    HOME_PATH = '/home/me/iot_thing/certs/'
    ca = HOME_PATH + 'root-CA.crt'
    crt = HOME_PATH + 'a123123123-certificate.pem.crt'
    key = HOME_PATH + 'a123123123-private.pem.key'
    IOT_ENDPOINT = 'a1231231231231.iot.us-west-2.amazonaws.com'
    CORE_ARN = 'arn:aws:iot:us-west-2:123123123123:thing/group1_core'
    
    # optional logging actions
    logger = logging.getLogger('AwsIoTPythonSDK.core')
    logger.setLevel(logging.DEBUG)
    streamHandler = logging.StreamHandler()
    formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
    streamHandler.setFormatter(formatter)
    logger.addHandler(streamHandler)
    
    # Discover the core
    diProvider = DiscoveryInfoProvider()
    diProvider.configureEndpoint(IOT_ENDPOINT)
    diProvider.configureCredentials(ca, crt, key)
    diProvider.configureTimeout(10)
    
    discoveryInfo = diProvider.discover('pi1')
    infoObj = discoveryInfo.toObjectAtGroupLevel()
    groupId = list(infoObj.keys())[0]  # Just getting the first group from our list of groups
    group = infoObj[groupId]
    core = group.getCoreConnectivityInfo(CORE_ARN)
    connectivityInfo = core.getConnectivityInfo('1')  # The arbitrary Id we put on our connection
    
    # Get groupCA, need that instead of the generic ca
    caList = discoveryInfo.getAllCas()
    _, ca_crt = caList[0]
    group_ca_path = '%s%s_CA_%s.crt' % (HOME_PATH, groupId, str(uuid.uuid4()))
    with open(group_ca_path, 'w') as group_ca_file:
        group_ca_file.write(ca_crt)
    
    ### NEW CODE STARTS BELOW ###
    # Set up shadow client before regular MQTT client:
    shadow_client = AWSIoTMQTTShadowClient('pi1')
    shadow_client.configureEndpoint(connectivityInfo.host, connectivityInfo.port)
    shadow_client.configureCredentials(group_ca_path, key, crt)
    shadow_client.connect()
    shadow = shadow_client.createShadowHandlerWithName(shadowName='pi1', isPersistentSubscribe=False)
    
    # Create an object to store state data from shadow callbacks
    class StateData:  # Python3
        def __init__(self):
            self.sensor_read = False
            self.shadow = {}
            # shadow.shadowDelete(self.delete, srcTimeout=5)  # Delete isn't supported in greengrass atm
            updateJson = {'state': {'desired': {'on': False}}}
            shadow.shadowUpdate(json.dumps(updateJson), self.update, srcTimeout=5)  # Set shadow to expected "off" status
            shadow.shadowRegisterDeltaCallback(self.delta)  # Subscribe to state changes requested from the cloud
    
        # def delete(self, payload, responseStatus, token):
        #    """delete is actually not supported in greengrass"""
        #     print('delete', responseStatus)
        #     print(json.loads(payload))        
            
        def delta(self, payload, responseStatus, token):
            """Listener for change requests from cloud"""
            print('delta', responseStatus)
            payloadDict = json.loads(payload)
            print(payloadDict)
            self.sensor_read = payloadDict['state'].get('on')
    
        # def get(self, payload, responseStatus, token):
        #    print('get', responseStatus)
        #    self.shadow = json.loads(payload)
        #    print(self.shadow)
            
        def update(self, payload, responseStatus, token):
            print('update', responseStatus)
            self.shadow = json.loads(payload)
            print(self.shadow)
            
    state = StateData() 
    
    def start_readings():
        # Don't connect to regular MQTT client until we want to actually start readings:
        # Also, we can just use the MQTT connection from the shadow config above:
        client = shadow_client.getMQTTConnection()
        counter = 0
        while state.sensor_read:
            data = {'test_data': random.randint(0, 100),
                    'reading_id': counter}
            print("Sending data", data)
            client.publish('pi1/topic_1', json.dumps(data), 0)
            counter += 1
            time.sleep(10)
            
    # Start waiting for device on / off
    while 1:
        if state.sensor_read:
            start_readings()
            print("Ended readings")
            break  # One-time start/stop. Stopping closes the program completely
    
    shadow_client.disconnect()

Notes:

* We changed the old client to the ShadowClient, but we can still do our old publish/subscribe methods using getMQTTConnection

* I created a class to provide callback functions and store the state data coming from the Shadow methods.

* There’s a script loop to keep the script running waiting for the “Sensor start/stop” command comes from the cloud.

* The “Sensor start command” starts another loop running sensor readings until a “Stop” command comes from the cloud.

* This script only allows one start & stop then ends the process.

* I shortened the sleep time, so we don’t wait as long when the script needs to shut down.

* I commented out delete and get, which we weren’t using in this workflow.

### Update the subscriptions for shadow events

The Greengrass subscriptions needs to be updated to allow the new MQTT topics. Run this script on the dev machine. 
Notes:

* The Source or Target for the shadow topics can be GGShadowService. It’s a Greengrass feature that holds a local shadow for devices in the core. It can sync your shadow state with the cloud, and also use Lambdas to update the local shadow.

* You have to subscribe to additional response events like update/accepted, update/rejected because the GGShadowService expects them. You get timeout errors if you don’t have them.

    import boto3
    import pprint

    pp = pprint.PrettyPrinter()
    gg = boto3.client('greengrass')
    
    group = gg.list_groups()['Groups'][0]  # I only have one group
    id = group['Id']
    gg.reset_deployments(GroupId=id)
    
    sub = gg.list_subscription_definitions()['Definitions'][0]
    current_subs = gg.get_subscription_definition_version(
        SubscriptionDefinitionId=sub['Id'],
        SubscriptionDefinitionVersionId=sub['LatestVersion']
    )
    # Loop through repetitive subscription setup:
    sub_list = current_subs['Definition']['Subscriptions']
    shadow_sub = '$aws/things/Pi1/shadow/'
    id_counter = 3
    for action in ['update']:  # could add 'get' here, and get/accepted etc below
        sub_list.append({'Id': str(id_counter),
                         'Source': pi1_arn,
                         'Subject': shadow_sub + action,
                         'Target': 'GGShadowService'})
        id_counter += 1
    for action in ['update/accepted', 'update/delta', 'update/rejected']:
        sub_list.append({'Id': str(id_counter),
                         'Source': 'GGShadowService',
                         'Subject': shadow_sub + action,
                         'Target': pi1_arn})
        id_counter += 1
    
    pp.pprint(sub_list)
    
    sub_version = gg.create_subscription_definition_version(
        SubscriptionDefinitionId=sub['Id'],
        Subscriptions=sub_list
    )
    old_group = gg.get_group_version(GroupId=id, GroupVersionId=group['LatestVersion'])
    group_kwargs = old_group['Definition']
    group_kwargs.update({'GroupId': id, 'SubscriptionDefinitionVersionArn': sub_version['Arn']})
    group_version = gg.create_group_version(**group_kwargs)
    
    deployment = gg.create_deployment(
        DeploymentType='NewDeployment',
        GroupId=id,
        GroupVersionId=group_version['Version']
    )
    print('Started final deployment step, printing the status in 10 seconds')
    time.sleep(10.5)
    pp.pprint(gg.get_deployment_status(
        DeploymentId=deployment['DeploymentId'],
        GroupId=id
    ))

### Interacting with this script

### UI:

Make sure the device is set up for shadow syncing (Should be set when we initially deployed).
To see that click groups:

![Shadow Syncing](https://cdn-images-1.medium.com/max/2276/1*1gSVqxrgT2woPmx8dyfq3A.png)*Shadow Syncing*

Then if you see “Local Shadow Storage Only” You’ll need to change it and redeploy

![Instructions for Syncing](https://cdn-images-1.medium.com/max/3340/1*Ru_9h-bneeMKP-IBwC00vQ.png)*Instructions for Syncing*

![Instructions for redeploying after changing Syncing settings](https://cdn-images-1.medium.com/max/3688/1*iMUYgk3egmeDUmiGKm0xIA.png)*Instructions for redeploying after changing Syncing settings*

Go to the device’s Shadow page in AWS. Something like: [https://us-west-2.console.aws.amazon.com/iotv2/home?region=us-west-2#/thing/Pi1](https://us-west-2.console.aws.amazon.com/iotv2/home?region=us-west-2#/thing/Pi1) 
You should see an empty (or old state):

![Device shadow original state](https://cdn-images-1.medium.com/max/3932/1*9WNdgjjhcm7KfRQ1RPjNjg.png)*Device shadow original state*

Run sensor.py 
You’ll see the AWS shadow is updated to ”on”: false

![Shadow editor](https://cdn-images-1.medium.com/max/3108/1*LNo9lh44wbMTZH-cto3xZw.png)*Shadow editor*

Click on the Edit button. Update the state, then hit Save in the right corner:

![Editing shadow](https://cdn-images-1.medium.com/max/3092/1*zP5HDv_dG-3bsVFKP_hi6Q.png)*Editing shadow*

If you check your terminal, sensor readings should have started printing, every 10 seconds.

To stop the script, Hit “Edit” again and change the state to ”on”: false. You should see the process stopped in ssh.

### Script / API:

In this script your dev machine hits the cloud API endpoints to update the thing’s cloud shadow, then AWS syncs that w/ Greengrass shadows.

    import argparse
    import boto3
    import pprint
    import json

    parser = argparse.ArgumentParser()
    parser.add_argument('device')
    parser.add_argument('command', choices=['on', 'off'])
    args = parser.parse_args()
        
    client = boto3.client('iot-data')
    payload = {
        'state': {
            'desired': {
                'on': args.command == 'on'
            }
        }
    }
    result = client.update_thing_shadow(
        thingName=args.device,
        payload=json.dumps(payload))
    print('result from cloud:')
    pp = pprint.PrettyPrinter()
    pp.pprint(json.loads(result['payload'].read()))

To run this:
1. python shadow_switch.py pi1 on
2. python shadow_switch.py pi1 off

### Summary

We updated our sensors to wait for a command to turn on sensor readings, and then a later command to turn off readings. This is leveraging Thing Shadows, which can be updated in the AWS Console or through API. The API gives inspiration to create an application to manage the devices.
