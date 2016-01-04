---
layout: post
title: "Event-based architecture using AWS (SNS and SQS)"
date: 2016-01-04 00:00
comments: true
categories: 
---

# Example event based architecture using AWS (SNS and SQS)

The following code is an example of an event based architecture using an SNS topic per event. It consists of:

* `Producer` - A system which produces events to an SNS topic.
* `Consumer` - A system which consumes messages placed onto an SQS queue by an SNS topic.

The two events we are interested in are:

* `INCIDENT_CREATED` 
* `INCIDENT_UPDATED`

We will create consumers which does the following:

* Send an email on `INCIDENT_CREATED`
* Send an SMS on `INCICENT_CREATED`

Instead of wrapping these into a single application, we will put these consumers in their own application ([single responsibility principle](https://en.wikipedia.org/wiki/Single_responsibility_principle)).

# INCIDENT_CREATED

## SNS Topic

1. Create a topic in the AWS console called `INCIDENT_CREATED`.

## Consumer 1

### SQS Queues

This will create a consumer which sends an email on incident created.

1. In the SQS console, create a queue for our consumer called `EMAIL_ON_INCIDENT_CREATED`.
2. Using the `Queue actions` drop down, create an SQS subscription to our `INCIDENT_CREATED` SNS topic for this SQS Queue.
3. Take note of the Queue URL. We will need this later.

Our SQS queue will now receive any messages which are published to the SNS topic `INCIDENT_CREATED`.

### Consumer application

1. Create a new visual studio console application project called `EMAIL_ON_INCIDENT_CREATED`.
2. Add a reference to the `AWSSDK.SQS` nuget package.
3. We will create a simple consumer which will consume messages from the SQS queue we created:

```
string queueUrl = "YOUR_QUEUE_URL";
using (var client = new Amazon.SQS.AmazonSQSClient(Amazon.RegionEndpoint.APSoutheast2))
{
    while (true)
    {
        // Get the messages
        var response = client.ReceiveMessage(queueUrl);

        // Check our response
        if (response.Messages.Count > 0)
        {
            for (int i = 0; i < response.Messages.Count; i++)
            {
                // Send an email
                Console.WriteLine("Sending email");

                // Delete our message so that it doesn't get handled again
                var receiptHandle = response.Messages[i].ReceiptHandle;
                client.DeleteMessage(queueUrl, receiptHandle);
            }
        }
    }
}
```

If you run this, you will be able to use the `Publish to topic` button in the SNS console to send a message to the consumer.

## Consumer 2

### SQS Queues

We will repeat the above process, except this time we will create a queue called `SMS_ON_INCIDENT_CREATED`.

1. In the SQS console, create a queue for our consumer called `SMS_ON_INCIDENT_CREATED`.
2. Using the `Queue actions` drop down, create an SQS subscription to our `INCIDENT_CREATED` SNS topic for this SQS Queue.
3. Take note of the Queue URL. We will need this later.

Our SQS queue will now receive any messages which are published to the SNS topic `INCIDENT_CREATED`.

### Consumer application

1. Create a new visual studio console application project called `SMS_ON_INCIDENT_CREATED`.
2. Add a reference to the `AWSSDK.SQS` nuget package.
3. We will create a simple consumer which will consume messages from the SQS queue we created:

```
string queueUrl = "YOUR_QUEUE_URL";
using (var client = new Amazon.SQS.AmazonSQSClient(Amazon.RegionEndpoint.APSoutheast2))
{
    while (true)
    {
        // Get the messages
        var response = client.ReceiveMessage(queueUrl);

        // Check our response
        if (response.Messages.Count > 0)
        {
            for (int i = 0; i < response.Messages.Count; i++)
            {
                // Send an email
                Console.WriteLine("Sending SMS");

                // Delete our message so that it doesn't get handled again
                var receiptHandle = response.Messages[i].ReceiptHandle;
                client.DeleteMessage(queueUrl, receiptHandle);
            }
        }
    }
}
```

If you run this, you will be able to use the `Publish to topic` button in the SNS console to send a message to the consumers. Both should receive the event.
