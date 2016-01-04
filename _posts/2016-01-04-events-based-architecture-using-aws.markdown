---
layout: post
title: "Event-based architecture using AWS (SNS and SQS) and C#"
date: 2016-01-04 00:00
comments: true
categories: 
---

# Example event based architecture using AWS (SNS and SQS) and C#

## Introduction

The following code is an example of an event based architecture using an SNS topic per event. It consists of:

* `Producer` - A system which produces events to an SNS topic.
* `Consumer` - A system which consumes messages placed onto an SQS queue by an SNS topic.

The two events we are interested in are:

* `INCIDENT_CREATED` 
* `INCIDENT_UPDATED`

We will create consumers which do the following:

* Consumer 1 - Send an email on `INCIDENT_CREATED`
* Consumer 2 - Send an SMS on `INCICENT_CREATED`
* Consumer 3 - Update a cache on `INCIDENT_CREATED` and `INCIDENT_UPDATED`

Instead of wrapping these into a single application, we will put these consumers in their own application ([single responsibility principle](https://en.wikipedia.org/wiki/Single_responsibility_principle)).

Consumers 1 and 2 are simple, and comprise of 1 topic, 1 queue and one consumer.

![one-topic-one-queue](http://i.imgur.com/DwNcoXU.png)

Consumer 3 is a little more complex in that it comprises of 2 topics, 1 queue and one consumer.

![two-topic-one-queue](http://i.imgur.com/bJLnbEM.png)

The source code for our examples can be found on my github account:

[https://github.com/darbio/aws-event-based-architecture](https://github.com/darbio/aws-event-based-architecture)

# Producer

## SNS Topics

1. Create a topic in the AWS console called `INCIDENT_CREATED`.
1. Create a topic in the AWS console called `INCIDENT_UPDATED`.

## Producer application

1. Create a new visual studio console application project called `EventProducer`.
2. Add a reference to the `AWSSDK.SimpleNotificationService` nuget package.
3. We will create a simple producer which will publish messages to the SNS topics we created:

```
string INCIDENT_CREATED_ARN = "arn:aws:sns:ap-southeast-2:835265634988:INCIDENT_CREATED";
string INCIDENT_UPDATED_ARN = "arn:aws:sns:ap-southeast-2:835265634988:INCIDENT_UPDATED";

using (var client = new Amazon.SimpleNotificationService.AmazonSimpleNotificationServiceClient(Amazon.RegionEndpoint.APSoutheast2))
{
    while (true)
    {
        Console.WriteLine("Press 1 to create a new incident, or 2 to update an existing incident.");
        var key = Console.ReadKey();

        if (key.KeyChar == '1')
        {
            Console.WriteLine("Creating incident.");
            client.Publish(INCIDENT_CREATED_ARN, "Incident created.");
        }
        else if (key.KeyChar == '2')
        {
            Console.WriteLine("Updating incident");
            client.Publish(INCIDENT_UPDATED_ARN, "Incident updated.");
        }
        else
        {
            Console.WriteLine("Unrecognised action.");
        }
    }
}
```

# Consumers

## Consumer 1 - Send an email on `INCIDENT_CREATED`

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

If you run this, you will be able to use the `Publish to topic` button in the SNS console, or the producer application to send a message to the consumer.

## Consumer 2 - Send an SMS on `INCIDENT_CREATED`

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

If you run this, you will be able to use the `Publish to topic` button in the SNS console, or the producer application to send a message to the consumer.

## Consumer 3 - Update a cache on `INCIDENT_CREATED` and `INCIDENT_UPDATED`

### SQS Queues

This will create a consumer which updates a cache on incident created or updated.

1. In the SQS console, create a queue for our consumer called `CACHE_ON_INCIDENT_ALL`.
2. Using the `Queue actions` drop down, create an SQS subscription to our `INCIDENT_CREATED` SNS topic for this SQS Queue.
2. Using the `Queue actions` drop down, create an SQS subscription to our `INCIDENT_UPDATED` SNS topic for this SQS Queue.
3. Take note of the Queue URL. We will need this later.

Our SQS queue will now receive any messages which are published to the SNS topic `INCIDENT_CREATED`.

### Consumer application

1. Create a new visual studio console application project called `CACHE_ON_INCIDENT_ALL`.
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
                Console.WriteLine("Updating Cache");

                // Delete our message so that it doesn't get handled again
                var receiptHandle = response.Messages[i].ReceiptHandle;
                client.DeleteMessage(queueUrl, receiptHandle);
            }
        }
    }
}
```

If you run this, you will be able to use the `Publish to topic` button in the SNS console, or the producer application to send a message to the consumers. When you publish to `INCIDENT_CREATED` all consumers should receive the event. When you publish to `INCIDENT_UPDATED` only consumer 3 should receive the event.

# Wrap up

In this example we have created an event based solution which allows the production of events to an SNS topic, and for multiple consumers to consume events from a producer.

![aws-topics](http://i.imgur.com/HkPxUvY.png)

![aws-queues](http://i.imgur.com/3XCiCRT.png)

![final-state](http://i.imgur.com/iovbnG8.png)

![applications](http://i.imgur.com/yberkNh.png)
