# Function compute with OSS trigger

Welcome to this tutorial! In this document you will learn about how to use
[Alibaba Cloud Function Compute](https://www.alibabacloud.com/product/function-compute) with 
[OSS](https://www.alibabacloud.com/product/oss) via [OSS triggers](https://www.alibabacloud.com/help/doc-detail/62922.htm).

## Summary

1. [Introduction](#introduction)
2. [Prerequisite](#prerequisite)
3. [Function Compute architecture and overall procedure](#function-compute-architecture-and-overall-procedure)
    1. [Abstract](#abstract)
    2. [Architecture and procedure](#architecture-and-procedure)
    3. [OSS events](#oss-events)
    4. [OSS event format](#oss-event-format)
4. [Steps](#steps)
    1. [Create an OSS bucket](#create-an-oss-bucket)
    2. [Create a function](#create-a-function)
    3. [Test the function](#test-the-function)
    4. [Create an OSS trigger](#create-an-oss-trigger)
    5. [Test the trigger](#test-the-trigger)
5. [Further reading](#further-reading)

## Introduction

Alibaba Cloud Function Compute is an event-driven and fully-managed compute service. With Function Compute, you can quickly build any type of applications or services without considering management or Operations and Maintenance (O&M). You can complete a set of backend services for processing multimedia data in just few days.

Function Compute integrates different services in an event-driven manner. When the event source service triggers an event, the associated function is automatically called to process the event.

You can trigger function invocations via events from [OSS](https://www.alibabacloud.com/product/oss), [Log Service](https://www.alibabacloud.com/product/log-service), [API Gateway](https://www.alibabacloud.com/product/api-gateway) or [Table Store](https://www.alibabacloud.com/product/table-store); or invoke functions directly via [the Function Compute SDK and REST API](https://www.alibabacloud.com/help/doc-detail/52878.htm). With these services and features, you can easily build elastic, reliable, and secure applications. For more information about the type of event sources supported by Function Compute, see [Trigger List](https://www.alibabacloud.com/help/doc-detail/74707.htm).

## Prerequisite
Please [create an Alibaba Cloud account](https://www.alibabacloud.com/help/doc-detail/50482.htm) and
[obtain an access key id and secret](https://www.alibabacloud.com/help/faq-detail/63482.htm).

You should also visit the [Function Compute console](https://fc.console.aliyun.com) and the
[OSS one](https://oss.console.aliyun.com) at least once, in order to activate these services.

## Function Compute architecture and overall procedure

### Abstract
Alibaba Cloud Object Storage Service (OSS) allows you to securely store massive amounts of data in the cloud at low costs. With HTTP RESTful APIs, you can easily store and access data on the Internet.

Function Compute integrates seamlessly with OSS. You can configure functions for different types of events so that OSS can automatically call a function after a certain event is detected.

### Architecture and procedure
To build a service in Function Compute, follow these steps:

![Workflow](images/Workflow.png "Workflow")

1. Create a service.
2. Create a function. Program an application by using function code.
3. Trigger function execution. Use an event to trigger the execution of a function.
4. View function execution logs.
5. View service monitoring and alarming.

### OSS events
After an event is detected, OSS encodes the event information into a JSON string and passes it to an event processing function. The following table lists the events that are supported by OSS.

| Event        | Description           | Remarks  |
| ------------- | ------------- | ----- |
| oss:ObjectCreated:PutObject      | A function is triggered after the OSS PutObject API operation creates an object. | PutObject is used to upload files. For more information, see [PutObject](https://www.alibabacloud.com/help/doc-detail/31978.htm). |
| oss:ObjectCreated:PutSymlink     | A function is triggered after the OSS PutSymlink API operation creates an object. |  PutSymlink is used to create a symbolic link that points to the TargetObject on OSS. You can use this symbolic link to access the TargetObject. For more information, see [PutSymlink](https://www.alibabacloud.com/help/doc-detail/45126.htm). |
| oss:ObjectCreated:PostObject | A function is triggered after the OSS PostObject API operation creates an object. | PostObject is used to upload files to a specific bucket using the HTML form. For more information, see [PostObject](https://www.alibabacloud.com/help/doc-detail/31988.htm). |
| oss:ObjectCreated:CopyObject | A function is triggered after the OSS CopyObject API operation creates an object. | CopyObject is used to copy an existing object in OSS into another object. For more information, see [CopyObject](https://www.alibabacloud.com/help/doc-detail/31979.htm). |
| oss:ObjectCreated:InitialMultipartUpload | A function is triggered after the OSS InitialMultipartUpload API operation is executed. | Before you transmit data in Multipart Upload mode, you must call the InitialMultipartUpload API operation to notify OSS to initialize a Multipart Upload event. For more information, see [InitiateMultipartUpload](https://www.alibabacloud.com/help/doc-detail/31992.htm).|
| oss:ObjectCreated:UploadPart | A function is triggered after the OSS UploadPart API operation is executed. | After a Multipart Upload event is initialized, you can upload data in parts based on the specified object name and Upload ID. For more information, see [UploadPart](https://www.alibabacloud.com/help/doc-detail/31993.htm). |
| oss:ObjectCreated:UploadPartCopy | A function is triggered after the OSS UploadPartCopy API operation is executed. | UploadPartCopy is used to upload a part by copying data from an existing object. For more information, see [UploadPartCopy](https://www.alibabacloud.com/help/doc-detail/31994.htm). |
| oss:ObjectCreated:CompleteMultipartUpload | A function is triggered after the OSS CompleteMultipartUpload API operation is executed. | After all data parts are uploaded, you must call the CompleteMultipartUpload API operation to complete the Multipart Upload of the entire file. For more information, see [CompleteMultipartUpload](https://www.alibabacloud.com/help/doc-detail/31995.htm).|
| oss:ObjectCreated:AppendObject | A function is triggered after the OSS AppendObject API operation is executed. | AppendObject is used to upload files in appending mode. For more information, see [AppendObject](https://www.alibabacloud.com/help/doc-detail/31981.htm). |
| oss:ObjectCreated:* | A function is triggered after any of the preceding ObjectCreated API operations is executed. | A function is triggered after any of the preceding ObjectCreated API operations is executed. |
| oss:ObjectRemoved:DeleteObject | A function is triggered after the OSS DeleteObject API operation deletes an object. | DeleteObject is used to delete a certain object. For more information, see [DeleteObject](https://www.alibabacloud.com/help/doc-detail/31982.htm). |
| oss:ObjectRemoved:DeleteObjects | A function is triggered after the OSS DeleteObjects API operation deletes multiple objects. | DeleteObjects is used to delete files in batches. |
| oss:ObjectRemoved:AbortMultipartUpload | A function is triggered after the OSS AbortMultipartUpload API operation stops a Multipart Upload event. | AbortMultipartUpload is used to stop a Multipart Upload event based on the Upload ID that you provide. For more information, see [AbortMultipartUpload](https://www.alibabacloud.com/help/doc-detail/31996.htm). |

### OSS event format
When an OSS event triggers a function to execute, the OSS event format serves as the input parameter event of the function. You can obtain the event information in the code. 

```json
{
  "events": [
    {
      "eventName": "ObjectCreated:PutObject",
      "eventSource": "acs:oss",
      "eventTime": "2019-01-16T12:46:37.000Z",
      "eventVersion": "1.0",
      "oss": {
        "bucket": {
          "arn": "acs:oss:ap-southeast-1:229814243474603096:fc-with-oss-trigger",
          "name": "fc-with-oss-trigger",
          "ownerIdentity": "229814243474603096",
          "virtualBucket": ""
        },
        "object": {
          "deltaSize": 122539,
          "eTag": "688A7BF4F233DC9C88A80BF985AB7329",
          "key": "source/serverless.png",
          "size": 122539
        },
        "ossSchemaVersion": "1.0",
        "ruleId": "9adac8e253828f4f7c0466d941fa3db81161e853"
      },
      "region": "ap-southeast-1",
      "requestParameters": {
        "sourceIPAddress": "140.205.128.221"
      },
      "responseElements": {
        "requestId": "58F9FF2D3DF792092E12044C"
      },
      "userIdentity": {
        "principalId": "229814243474603096"
      }
    }
  ]
}
```

## Steps

In this sample, you can learn how to connect Object Storage Service (OSS) with Function Compute by using OSS triggers. It assumes that you have signed up for Function Compute and OSS.

In this example, a file prefixed with `source/` and stored in the specified OSS bucket can call the resize function automatically. This function scales the image and stores the processing result in the `processed/` directory of the same bucket. For example, the function processes `source/serverless.png` as `processed/serverless.png`.

### Create an OSS bucket
Before you follow these steps to create an Object Storage Service (OSS) bucket, make sure that you have activated OSS:

1. Log on to the [OSS console](https://oss.console.aliyun.com/).
2. See ***OSS topic*** [Create a bucket](https://www.alibabacloud.com/help/doc-detail/31885.htm) to create a bucket.

    In this sample, we select the Singapore region, set the name of the OSS bucket to fc-with-oss-trigger, set __Storage Class__ to __Standard Storage__, and set __ACL__ to __Private__.

    >__Note__: The __Storage Class__ and region cannot be changed once a bucket is created.

    ![Create OSS bucket](images/fc-create-bucket.png "Create OSS bucket")

3. Click the bucket you created. On the displayed page, click the __Files__ tab, click __Create Directory__, and set the directory name to source. Click __OK__.

    ![Create OSS Folder](images/fc-oss-create-folder.png "Create OSS Folder")

4. In the /source directory, upload an image. In this example, the serverless.png image is uploaded.

    ![Upload File to OSS](images/fc-oss-upload-file.png "Upload File to OSS")


### Create a function
>__Note__: Your function and the OSS bucket must be in the same region. For example, if the OSS bucket is in Singapore, set the region of the service to Singapore.

1. Log on to the [Function Compute console](https://fc.console.aliyun.com/).
2. See topic [Service operations](https://www.alibabacloud.com/help/doc-detail/73337.htm) to create a service.
  
    In this example, we select the Singapore region, set the service name to __fc-with-oss-trigger-demo__, select the __fc-with-oss-trigger-demo-log__ log project, select the __fc-with-oss-trigger-demo-log-project__ Logstore, role operation to Create new role, and system policies to `AliyunOSSFullAccess` and `AliyunLogFullAccess`.

    ![Create Function Compute Service](images/fc-create-fc-service.png "Create Function Compute Service")

    ![Create Function Compute Service Config](images/fc-create-fc-service-config.png "Create Function Compute Service Config")

3. See topic [Function operations](https://www.alibabacloud.com/help/doc-detail/73338.htm) to create a function.
        
    In this example, we select the __Empty Function__ template, create no triggers, set the function name to __resize__, runtime environment to __Python__, and leave other parameters to their default values.

    ![Create Function Compute Function](images/fc-create-function.png "Create Function Compute Function")

4. Edit your function code. In this example, we set the function code as follows.

```python
# -*- coding: utf-8 -*-
import oss2, json
from wand.image import Image

def resize(event, context):
    evt = json.loads(event)
    creds = context.credentials
    # Required by OSS SDK
    auth=oss2.StsAuth(
        creds.access_key_id,
        creds.access_key_secret,
        creds.security_token)
    evt = evt['events'][0]
    bucket_name = evt['oss']['bucket']['name']
    endpoint = 'oss-' +  evt['region'] + '.aliyuncs.com'
    bucket = oss2.Bucket(auth, endpoint, bucket_name)
    objectName = evt['oss']['object']['key']
    # Processed images will be saved to processed/
    newKey = objectName.replace("source/", "processed/")
    remote_stream = bucket.get_object(objectName)
    if not remote_stream:
        return
    remote_stream = remote_stream.read()
    with Image(blob=remote_stream)  as img:
        with img.clone() as i:
            i.resize(128, 128)
            new_blob = i.make_blob()
            bucket.put_object(newKey, new_blob)
```
### Test the function
Before creating a trigger, you can simulate the execution process by triggering an event. In this step, it simulates the execution process of Function Compute when an object in the source/ directory is created in the OSS bucket. You can use this method for debugging and testing.

To test the function in the Function Compute console, follow these steps:
1. Log on to the [Function Compute console](https://fc.console.aliyun.com/).
2. On the code execution page, click __Trigger Event__.
3. Set the trigger __event__ as OSS trigger event.
4. Click __Invoke__.
5. After the function is executed successfully, you can find the `processed` directory named under the corresponding OSS bucket. This directory contains the serverless.png processed image.
6. In the __Test Event__ dialog box, select __OSS Template__, and edit your `arn`, `name`, `ownerIdentity`, and `key` parameters in the event template accordingly.

![Function Compute Invoke Function](images/fc-invoke-function.png "Function Compute Invoke Function")

### Create an OSS trigger
1. Log on to the [Function Compute console](https://fc.console.aliyun.com/).
2. Click __Triggers__ on the code execution page.
3. Set the trigger type as __Object Storage Service (OSS)__, and select the new bucket.
4. Select `oss:objectCreated:*` as the trigger event, and __source/__ as the prefix.

    This configuration indicates that the function is triggered immediately when a new object is created with a source/ prefix in the bucket.
5. In Invocation Role Management, select __Select an existing role__. The system provides a role named `AliyunOSSEventNotificationRole` for the OSS trigger, and you can select this role directly as the trigger role.
    
    The trigger needs to set a trigger role to authorize the execution of the function. OSS needs to play this role to trigger the function. For more information on permissions, see [User permissions](https://www.alibabacloud.com/help/doc-detail/52885.htm).

![Function Compute Create Trigger](images/fc-create-trigger.png "Function Compute Create Trigger")

### Test the trigger
After the OSS trigger is set, you can test the entire project. You can upload a new image to the corresponding `source/` directory in the Bucket in OSS console, and you find a new resized image of the same name in the `processed/` directory.

![Function Compute Test Trigger](images/fc-test-trigger.png "Function Compute Test Trigger")

## Further reading
* [Help documents](https://www.alibabacloud.com/help/doc-detail/52895.htm)
* [OSS API Documents](https://www.alibabacloud.com/help/doc-detail/31947.htm)