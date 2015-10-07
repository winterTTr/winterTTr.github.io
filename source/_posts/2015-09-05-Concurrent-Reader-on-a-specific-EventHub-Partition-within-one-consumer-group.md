title: Concurrent Reader on a specific EventHub Partition within one consumer group
date: 2015-09-05 15:15:08
tags:
  - C#
  - Azure
  - Eventhub
  - Service Bus
  - Consumer Group
---


When using `Eventhub` in `Azure Service Bus`, there is many things you need understand, I will talk something specific about about concurrent reader on consumer group.


<!--more-->




As [Event Hubs Overview](http://blogs.msdn.com/controlpanel/blogs/posteditor.aspx/Event%20Hubs%20Overview) said:

> The publish/subscribe mechanism of Event Hubs is enabled through consumer groups. A consumer group is a view (state, position, or offset) of an entire Event Hub. Consumer groups enable multiple consuming applications to each have a separate view of the event stream, and to read the stream independently at its own pace and with its own offsets. In a stream processing architecture, each downstream application equates to a consumer group. If you want to write event data to long-term storage, then that storage writer application is a consumer group. Complex event processing is performed by another, separate consumer group. You can only access partitions through a consumer group. There is always a default consumer group in an Event Hub, and you can create up to 20 consumer groups for a Standard tier Event Hub.


> In order to consume events from an Event Hub, a consumer must connect to a partition. As mentioned previously, you always access partitions through a consumer group. **<font color="red">As part of the partitioned consumer model, only a single reader should be active on a partition at any one time within a consumer group.</font>** It is common practice when connecting directly to partitions to use a leasing mechanism in order to coordinate reader connections to specific partitions. This way, it is possible for every partition in a consumer group to have only one active reader. Managing the position in the sequence for a reader is an important task that is achieved through checkpointing. This functionality is simplified by using the EventProcessorHost class for .NET clients. EventProcessorHost is an intelligent consumer agent and is described in the following section. 


So based on the event hub documentation, we should only have **ONE** active reader( receiver ) on a partition within the same consumer group.

OK, you know, sometimes *should does not means "must"*. So let's see another documentation from `Azure Stream Analytics`.


[Azure Stream Analytics Preview limitations and known issues](http://blogs.msdn.com/controlpanel/blogs/posteditor.aspx/Azure%20Stream%20Analytics%20Preview%20limitations%20and%20known%20issues):

> Each Stream Analytics job input should be configured to have its own event-hub consumer group. **<font color="red">When a job contains self-join or multiple outputs, some input may be read by more than one reader, which causes the total number of readers in a single consumer group to exceed the event hub’s limit of 5 readers per consumer group.</font>** In this case, the query will need to be broken down into multiple queries and intermediate results routed through additional event hubs. Note that there is also a limit of 20 consumer groups per event hub. For details, see Event Hubs developer guide.

OK, here we see that, in fact, the limitation of total number of readers in a single consumer group on a specific partition is 5.

Let's do some code test:


```csharp
[TestMethod]
public void Concurrent_Readers_On_1Partition_1ConsumerGroup() 
{
    var connectionString = CloudConfigurationManager.GetSetting("ServiceBus.Eventhub.ConnectionString");
    const string eventhubPath = "eventhub";



    var nsm = NamespaceManager.CreateFromConnectionString(connectionString);
    var description = nsm.CreateEventHubIfNotExists(eventhubPath);


    var builder = new ServiceBusConnectionStringBuilder(connectionString) 
    {
        TransportType = TransportType.Amqp
    };
    var factory = MessagingFactory.CreateFromConnectionString(builder.ToString());
    var client = factory.CreateEventHubClient(eventhubPath);

    var partition = description.PartitionIds[0];



    var group = client.GetDefaultConsumerGroup();
    try {
        var receiverList = new List < EventHubReceiver > 
        {
            group.CreateReceiver(partition),
            group.CreateReceiver(partition),
            group.CreateReceiver(partition),
            group.CreateReceiver(partition),
            group.CreateReceiver(partition),
            group.CreateReceiver(partition), // we create more than 5 first and comment this line to pass the test
        };

        var taskFactory = new TaskFactory();

        var task = (
        from r in receiverList
        select taskFactory.StartNew(
        () = > 
        {
            Task.Delay(TimeSpan.FromSeconds(1));
            var msg = r.Receive();
            var body = Encoding.UTF8.GetString(msg.GetBytes());
            Trace.TraceInformation(
            String.Format(
                "Receiver{0}: {1} at offset {2}",
            receiverList.IndexOf(r),
            body,
            msg.Offset));
        })).ToList();

        Task.WaitAll(task.ToArray());

    } catch (Exception e) 
    {
        Trace.TraceInformation(e.Message);
    }
}```

First we create more than 5 reader one a specific partition:

{% qnimg "2015-09-05-Concurrent-Reader-on-a-specific-EventHub-Partition-within-one-consumer-group/exception-more-than-5-reader.png" %}

We will received the above exception.

After comments the 6th reader, and send some data to the event hub, I try to rerun the test again. Then we got the following result:

> vstest.executionengine.x86.exe Information: 0 : Receiver1: { a : 16} at offset 0
vstest.executionengine.x86.exe Information: 0 : Receiver0: { a : 16} at offset 0
vstest.executionengine.x86.exe Information: 0 : Receiver2: { a : 16} at offset 0
vstest.executionengine.x86.exe Information: 0 : Receiver3: { a : 16} at offset 0
vstest.executionengine.x86.exe Information: 0 : Receiver4: { a : 16} at offset 0

So we can see that, the result is obvious, we can see the five reader can work at the same time without competing.


Hope they can give you some more idea when you try to read the data from even hub directly by yourself without using `EventProcessorHost`
