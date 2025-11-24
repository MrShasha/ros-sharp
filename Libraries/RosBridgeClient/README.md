# [RosBridgeClient](https://github.com/siemens/ros-sharp/tree/master/RosBridgeClient) #
[.NET](https://www.microsoft.com/net) API to [ROS](http://www.ros.org/) via [rosbridge_suite](http://wiki.ros.org/rosbridge_suite)


## Jiří Šašek fork changes:
### Communicators.cs:
``csharp
    // Subscriber for raw JObject delivery
    internal class JObjectSubscriber : Subscriber
    {
        internal override string Id { get; }
        internal override string Topic { get; }
        internal override Type TopicType { get { return typeof(Newtonsoft.Json.Linq.JObject); } }

        internal Action<Newtonsoft.Json.Linq.JObject> SubscriptionHandler { get; }

        public JObjectSubscriber(string id, string topic, Action<Newtonsoft.Json.Linq.JObject> subscriptionHandler, out Subscription subscription, int throttle_rate = 0, int queue_length = 1, int fragment_size = int.MaxValue, string compression = "none")
        {
            Id = id;
            Topic = topic;
            SubscriptionHandler = subscriptionHandler;
            subscription = new Subscription(id, Topic, "std_msgs/String", throttle_rate, queue_length, fragment_size, compression); // message type will be overridden
        }

        internal override void Receive(string message, ISerializer serializer)
        {
            // Always deliver raw JObject
            var jObj = Newtonsoft.Json.Linq.JObject.Parse(message);
            SubscriptionHandler.Invoke(jObj);
        }
    }
    ```

### RosSocket.cs:
``csharp
        // Subscribe with explicit message type, delivering raw JObject
        public string SubscribeWithMessageTypeJObject(string topic, string messageType, Action<Newtonsoft.Json.Linq.JObject> subscriptionHandler, int throttle_rate = 0, int queue_length = 1, int fragment_size = int.MaxValue, string compression = "none")
        {
            string id;
            lock (SubscriberLock)
            {
                id = GetUnusedCounterID(Subscribers, topic);
                Subscription subscription;
                var subscriber = new JObjectSubscriber(id, topic, subscriptionHandler, out subscription, throttle_rate, queue_length, fragment_size, compression);
                subscription.type = messageType;
                Subscribers.Add(id, subscriber);
                Send(subscription);
            }
            return id;
        }
    ```

__Please see the [Wiki](https://github.com/siemens/ros-sharp/wiki) for further info.__

---

© Siemens AG, 2017-2024
