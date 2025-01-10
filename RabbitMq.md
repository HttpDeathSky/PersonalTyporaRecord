# RabbitMQ Tutorials

## Introduction

[Official]: https://www.rabbitmq.com/docs	"Official"

最新关于RabbitMq的文档强烈建议查阅官方文档，优于大部分第三方文档，且简单易读。

**Attention!**最新版本RabbitMq:4.0.3对于Centos仅支持企业Centos8、Centos Stream 9，因erlang版本对于过时Centos不支持，详细请翻阅官方文档。

关于rabbitmq : amqp-client的rpc、publisher confirms部门更多内容请翻阅官网或查阅资料。



Rabbitmq由exchange \ queue组成

exchange目前已知：direct/topic/fanout/header 四种

direct/topic 模式下routingkey 生效，两种模式分别代表不同交换方式

## Install

### Local Server

使用Ubuntu或Centos Stream 9更新yum、epel库，安装erlang和socat依赖，再安装RabbitMq。

### Docker

直接拉去RabbitMq镜像，最新参考官方文档。

## WebUi

## Getting Started 

### [com.rabbitmq : amqp-client]

[Official]: https://www.rabbitmq.com/tutorials/tutorial-one-java	"官方教学"

**Dependency**

```xml
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
</dependency>
```

**`basicPublish` 的参数**

`basicPublish` 方法用于将消息发布到交换机。以下是其参数列表和作用：

```java
void basicPublish(String exchange, String routingKey, AMQP.BasicProperties props, byte[] body)
        throws IOException;
```

**参数说明**

| **参数**         | **类型**               | **描述**                                                     |
| ---------------- | ---------------------- | ------------------------------------------------------------ |
| **`exchange`**   | `String`               | 交换机名称： - 如果为空字符串 `""`，表示默认交换机。 - 如果为具体名称，消息会发送到指定交换机。 |
| **`routingKey`** | `String`               | 路由键： - 用于匹配队列绑定规则。 - 在默认交换机中，路由键通常是队列名称。 |
| **`props`**      | `AMQP.BasicProperties` | 消息属性：可以附加元数据（如持久化、过期时间、优先级等）。   |
| **`body`**       | `byte[]`               | 消息内容：必须以字节数组形式传递数据（消息正文）。           |



#### HelloWorld

生产者向queue发送消息，不通过交换机，消费者直接送queue获取内容。

```java
//Producer
package com.example.rabbitmqdemo.helloworld;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

public class Send {
   	//QUeue Name
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] argv) throws Exception {
        //Connection Obj
        ConnectionFactory factory = new ConnectionFactory();
        //set con params
        factory.setHost("1.94.16.94");
        factory.setUsername("admin");
        factory.setPassword("admin");
        try (Connection connection = factory.newConnection(); Channel channel = connection.createChannel()) {
            //Channel obj & set channel params
            channel.queueDeclare(QUEUE_NAME, false, false, false, null);
            String message = "Hello World!";
			//push message
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
            System.out.println(" [x] Sent '" + message + "'");
        }
    }
}
```

```java
//Consumer
package com.example.rabbitmqdemo.helloworld;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

import java.nio.charset.StandardCharsets;

public class Recv {

    private final static String QUEUE_NAME = "hello";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("1.94.16.94");
        factory.setUsername("admin");
        factory.setPassword("admin");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
            System.out.println(" [x] Received '" + message + "'");
        };
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, consumerTag -> {
        });
    }
}
```

#### WorkQueue

关于Ack(手动应答、自动应答)、Durable（持久化）、Qos(Quality of Service)分配

```java
//Durable Consumer
//muti Consumer : try to know rabbit how to assign message to muti consumer
package com.example.rabbitmqdemo.workqueue;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

import java.nio.charset.StandardCharsets;

public class DurableWorker {

    private final static String QUEUE_NAME = "task_queue";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("1.94.16.94");
        factory.setUsername("admin");
        factory.setPassword("admin");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        //consumer max qos & if not response ack rabbit will not send message to this consumer
        channel.basicQos(1);
        //if channel exist & queueDeclare cant change the staut of durable
        boolean durable = true;
        channel.queueDeclare(QUEUE_NAME, durable, false, false, null);
        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

        //callback function handle when message in
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
            System.out.println(" [x] Received '" + message + "'");
            try {
                doWork(message);
            } finally {
                System.out.println(" [x] Done");
                //if not auto response ack,u should response ack manually
                channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
            }
        };
        //is or not auto response ack
        boolean autoAck = false; // acknowledgment is covered below
        channel.basicConsume(QUEUE_NAME, autoAck, deliverCallback, consumerTag -> {
        });
    }

    //simulate processing data(need time)
    private static void doWork(String task) {
        for (char ch : task.toCharArray()) {
            try {
                if (ch == '.') Thread.sleep(1000);
            } catch (InterruptedException _ignored) {
                Thread.currentThread().interrupt();
            }
        }
    }
}
```

```java
//Durable Producer
package com.example.rabbitmqdemo.workqueue;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.MessageProperties;

import java.util.Arrays;
import java.util.List;

public class DurableTask {
    private final static String QUEUE_NAME = "task_queue";

    public static void main(String[] argv) throws Exception {
        List<String> list = Arrays.asList("1 message.", "2 message..", "3 message...", "4 message....", "5 message.....", "6 message......", "7 message.......");

        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("1.94.16.94");
        factory.setUsername("admin");
        factory.setPassword("admin");
        for (String s : list) {
            try (Connection connection = factory.newConnection(); Channel channel = connection.createChannel()) {
                boolean durable = true;
                channel.queueDeclare(QUEUE_NAME, durable, false, false, null);
//            String message = String.join(" ", argv);
                String message = s;
                channel.basicPublish("", QUEUE_NAME, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes());
                System.out.println(" [x] Sent '" + message + "'");
            }
        }
    }
}
```

#### Publish/Subscribe(fanout)

扇出交换机，一Exchange对多queue，不同消费者通过各自的queue同时消费同一生产者发送给的Exchange。

```java
//Provider
package com.example.rabbitmqdemo.publish_subscribe;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.util.Arrays;
import java.util.List;

public class Provider {
    private final static String QUEUE_NAME = "";
    private final static String EXCHANGE_NAME = "logs";

    public static void main(String[] argv) throws Exception {
        List<String> list = Arrays.asList("1 message.", "2 message..", "3 message...", "4 message....", "5 message.....", "6 message......", "7 message.......");

        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("1.94.16.94");
        factory.setUsername("admin");
        factory.setPassword("admin");
        for (String s : list) {
            try (Connection connection = factory.newConnection(); Channel channel = connection.createChannel()) {
                channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
                String message = s;
                channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes());
                System.out.println(" [x] Sent '" + message + "'");
            }
        }
    }
}
```

```java
//Consumer
package com.example.rabbitmqdemo.publish_subscribe;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

import java.nio.charset.StandardCharsets;

public class Consumer {

    private final static String QUEUE_NAME = "";
    private static final String EXCHANGE_NAME = "logs";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("1.94.16.94");
        factory.setUsername("admin");
        factory.setPassword("admin");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
        String queueName = channel.queueDeclare().getQueue();
        channel.queueBind(queueName, EXCHANGE_NAME, "");
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
            System.out.println(" [x] Received '" + message + "'");
        };
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, consumerTag -> {
        });
    }
}
```

#### Routing

根据exchange[direct\fanout\topic\headers]的类型，决定routingKey是否生效，type为direct时：routingKey为完全匹配，为topic时根据配置规则来。

通过queueBind指定queue接受哪个exchange的哪个routingKey的信息。

basicPublish发送message到指定exchange和绑定了routingkey的queue。

```java
//Provider
package com.example.rabbitmqdemo.routing;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.util.Arrays;
import java.util.List;

public class Provider {

    public static void main(String[] argv) throws Exception {
        List<String> list = Arrays.asList("1 message.", "2 message..", "3 message...", "4 message....", "5 message.....", "6 message......", "7 message.......");
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("1.94.16.94");
        factory.setUsername("admin");
        factory.setPassword("admin");
        for (String message : list) {
            try (Connection connection = factory.newConnection(); Channel channel = connection.createChannel()) {
                channel.exchangeDeclare("routing", "direct");
                channel.basicPublish("routing", "warning", null, message.getBytes());
                System.out.println(" [x] Sent '" + message + "'");
            }
        }
    }
}
```

```java
//Consumer 1
package com.example.rabbitmqdemo.routing;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

import java.nio.charset.StandardCharsets;

public class Consumer {

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("1.94.16.94");
        factory.setUsername("admin");
        factory.setPassword("admin");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        channel.exchangeDeclare("routing", "direct");
        String queueName = channel.queueDeclare().getQueue();
        channel.queueBind(queueName, "routing", "error");
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
            System.out.println(" [x] Received '" +
                    delivery.getEnvelope().getRoutingKey() + "':'" + queueName + "':'" + message + "'");
        };
        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> {
        });
    }
}
```

```java
//Consumer 2
package com.example.rabbitmqdemo.routing;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

import java.nio.charset.StandardCharsets;

public class Consumerr {
    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("1.94.16.94");
        factory.setUsername("admin");
        factory.setPassword("admin");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        channel.exchangeDeclare("routing", "direct");
        String queueName = channel.queueDeclare().getQueue();
        channel.queueBind(queueName, "routing", "error");
        channel.queueBind(queueName, "routing", "info");
        channel.queueBind(queueName, "routing", "warning");
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
            System.out.println(" [x] Received '" +
                    delivery.getEnvelope().getRoutingKey() + "':'" + queueName + "':'" + message + "'");
        };
        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> {
        });
    }
}
```

#### Topic

当exchangeDeclare类型为topic时，queueBind的routingKey为配置规则，*可以替换一个单词，#可以替换多个单词，当没有适配的routingKey时，则会丢失消息。

```java
package com.example.rabbitmqdemo.topic;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.util.Arrays;
import java.util.List;

public class Provider {

    private static final String ROUTINGKEY = "topic.test.sadklfj";

    public static void main(String[] argv) throws Exception {
        List<String> list = Arrays.asList("1 message.", "2 message..", "3 message...", "4 message....", "5 message.....", "6 message......", "7 message.......");
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("1.94.16.94");
        factory.setUsername("admin");
        factory.setPassword("admin");
        for (String message : list) {
            try (Connection connection = factory.newConnection(); Channel channel = connection.createChannel()) {
                channel.exchangeDeclare("topic", "topic");
                channel.basicPublish("topic", ROUTINGKEY, null, message.getBytes());
                System.out.println(" [x] Sent '" + "routingKey : " + ROUTINGKEY + "," + message + "'");
            }
        }
    }
}
```

```java
package com.example.rabbitmqdemo.topic;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

import java.nio.charset.StandardCharsets;

public class Consumer {

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("1.94.16.94");
        factory.setUsername("admin");
        factory.setPassword("admin");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        channel.exchangeDeclare("topic", "topic");
        String queueName = channel.queueDeclare().getQueue();
        channel.queueBind(queueName, "topic", "topic.#");
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
            System.out.println(" [x] Received '" +
                    delivery.getEnvelope().getRoutingKey() + "':'" + queueName + "':'" + message + "'");
        };
        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> {
        });
    }
}
```

```java
package com.example.rabbitmqdemo.topic;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

import java.nio.charset.StandardCharsets;

public class Consumerr {
    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("1.94.16.94");
        factory.setUsername("admin");
        factory.setPassword("admin");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        channel.exchangeDeclare("topic", "topic");
        String queueName = channel.queueDeclare().getQueue();
        channel.queueBind(queueName, "topic", "*.*.test");
        channel.queueBind(queueName, "topic", "*.test.#");
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
            System.out.println(" [x] Received '" +
                    delivery.getEnvelope().getRoutingKey() + "':'" + queueName + "':'" + message + "'");
        };
        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> {
        });
    }
}
```

#### RPC

生产者发送消息时与BasicProperties中传递callbackqueue(name)告知消费者当处理完消息后与哪个queue返回结果，同时生成并传递correlationId，在消费者返回时根据生产者提供的queue将参数和带有correlationId的BasicProperties一并发送，在生产者执行回调函数时校验correlationId是否一致，并处理返回结果。

##### Client interface

To illustrate how an RPC service could be used we're going to create a simple client class. It's going to expose a method named `call` which sends an RPC request and blocks until the answer is received:

```java
FibonacciRpcClient fibonacciRpc = new FibonacciRpcClient();
String result = fibonacciRpc.call("4");
System.out.println( "fib(4) is " + result);
```

> **A note on RPC**
>
> Although RPC is a pretty common pattern in computing, it's often criticised. The problems arise when a programmer is not aware whether a function call is local or if it's a slow RPC. Confusions like that result in an unpredictable system and adds unnecessary complexity to debugging. Instead of simplifying software, misused RPC can result in unmaintainable spaghetti code.
>
> Bearing that in mind, consider the following advice:
>
> - Make sure it's obvious which function call is local and which is remote.
> - Document your system. Make the dependencies between components clear.
> - Handle error cases. How should the client react when the RPC server is down for a long time?
>
> When in doubt avoid RPC. If you can, you should use an asynchronous pipeline - instead of RPC-like blocking, results are asynchronously pushed to a next computation stage.

##### Callback queue

In general doing RPC over RabbitMQ is easy. A client sends a request message and a server replies with a response message. In order to receive a response we need to send a 'callback' queue address with the request. We can use the default queue (which is exclusive in the Java client). Let's try it:

We need this new import:

```java
import com.rabbitmq.client.AMQP.BasicProperties;
```

```java
callbackQueueName = channel.queueDeclare().getQueue();
BasicProperties props = new BasicProperties
                            .Builder()
                            .replyTo(callbackQueueName)
                            .build();
channel.basicPublish("", "rpc_queue", props, message.getBytes());
// ... then code to read a response message from the callback_queue ...
```

> **Message properties**
>
> The AMQP 0-9-1 protocol predefines a set of 14 properties that go with a message. Most of the properties are rarely used, with the exception of the following:
>
> - `deliveryMode`: Marks a message as persistent (with a value of `2`) or transient (any other value). You may remember this property from [the second tutorial](https://www.rabbitmq.com/tutorials/tutorial-two-java).
> - `contentType`: Used to describe the mime-type of the encoding. For example for the often used JSON encoding it is a good practice to set this property to: `application/json`.
> - `replyTo`: Commonly used to name a callback queue.
> - `correlationId`: Useful to correlate RPC responses with requests.

##### Correlation Id

In the method presented above we suggest creating a callback queue for every RPC request. That's pretty inefficient, but fortunately there is a better way - let's create a single callback queue per client.

That raises a new issue, having received a response in that queue it's not clear to which request the response belongs. That's when the `correlationId` property is used. We're going to set it to a unique value for every request. Later, when we receive a message in the callback queue we'll look at this property, and based on that we'll be able to match a response with a request. If we see an unknown `correlationId` value, we may safely discard the message - it doesn't belong to our requests.

You may ask, why should we ignore unknown messages in the callback queue, rather than failing with an error? It's due to a possibility of a race condition on the server side. Although unlikely, it is possible that the RPC server will die just after sending us the answer, but before sending an acknowledgment message for the request. If that happens, the restarted RPC server will process the request again. That's why on the client we must handle the duplicate responses gracefully, and the RPC should ideally be idempotent.

##### Summary

- Our RPC will work like this:
  - For an RPC request, the Client sends a message with two properties: `replyTo`, which is set to an anonymous exclusive queue created just for the request, and `correlationId`, which is set to a unique value for every request.
  - The request is sent to an `rpc_queue` queue.
  - The RPC worker (aka: server) is waiting for requests on that queue. When a request appears, it does the job and sends a message with the result back to the Client, using the queue from the `replyTo` field.
  - The client waits for data on the reply queue. When a message appears, it checks the `correlationId` property. If it matches the value from the request it returns the response to the application.

##### Putting it all together

The Fibonacci task:

```java
private static int fib(int n) {
    if (n == 0) return 0;
    if (n == 1) return 1;
    return fib(n-1) + fib(n-2);
}
```

We declare our fibonacci function. It assumes only valid positive integer input. (Don't expect this one to work for big numbers, and it's probably the slowest recursive implementation possible).

The code for our RPC server can be found here: [`RPCServer.java`](https://github.com/rabbitmq/rabbitmq-tutorials/blob/main/java/RPCServer.java).

The server code is rather straightforward:

- As usual we start by establishing the connection, channel and declaring the queue.
- We might want to run more than one server process. In order to spread the load equally over multiple servers we need to set the `prefetchCount` setting in channel.basicQos.
- We use `basicConsume` to access the queue, where we provide a callback in the form of an object (`DeliverCallback`) that will do the work and send the response back.

The code for our RPC client can be found here: [`RPCClient.java`](https://github.com/rabbitmq/rabbitmq-tutorials/blob/main/java/RPCClient.java).

The client code is slightly more involved:

- We establish a connection and channel.
- Our `call` method makes the actual RPC request.
- Here, we first generate a unique `correlationId` number and save it - our consumer callback will use this value to match the appropriate response.
- Then, we create a dedicated exclusive queue for the reply and subscribe to it.
- Next, we publish the request message, with two properties: `replyTo` and `correlationId`.
- At this point we can sit back and wait until the proper response arrives.
- Since our consumer delivery handling is happening in a separate thread, we're going to need something to suspend the `main` thread before the response arrives. Usage of `CompletableFuture` is one possible solution to do so.
- The consumer is doing a very simple job, for every consumed response message it checks if the `correlationId` is the one we're looking for. If so, it completes the `CompletableFuture`.
- At the same time `main` thread is waiting for the `CompletableFuture` to complete.
- Finally, we return the response back to the user.

Now is a good time to take a look at our full example source code (which includes basic exception handling) for [RPCClient.java](https://github.com/rabbitmq/rabbitmq-tutorials/blob/main/java/RPCClient.java) and [RPCServer.java](https://github.com/rabbitmq/rabbitmq-tutorials/blob/main/java/RPCServer.java).

Compile and set up the classpath as usual (see [tutorial one](https://www.rabbitmq.com/tutorials/tutorial-one-java)):

```bash
javac -cp $CP RPCClient.java RPCServer.java
```

Our RPC service is now ready. We can start the server:

```bash
java -cp $CP RPCServer
# => [x] Awaiting RPC requests
```

To request a fibonacci number run the client:

```bash
java -cp $CP RPCClient
# => [x] Requesting fib(30)
```

The design presented here is not the only possible implementation of a RPC service, but it has some important advantages:

- If the RPC server is too slow, you can scale up by just running another one. Try running a second `RPCServer` in a new console.
- On the client side, the RPC requires sending and receiving only one message. No synchronous calls like `queueDeclare` are required. As a result the RPC client needs only one network round trip for a single RPC request.

Our code is still pretty simplistic and doesn't try to solve more complex (but important) problems, like:

- How should the client react if there are no servers running?
- Should a client have some kind of timeout for the RPC?
- If the server malfunctions and raises an exception, should it be forwarded to the client?
- Protecting against invalid incoming messages (eg checking bounds, type) before processing.

> If you want to experiment, you may find the [management UI](https://www.rabbitmq.com/docs/management) useful for viewing the queues.

##### Code Example

```java
import com.rabbitmq.client.AMQP;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.util.UUID;
import java.util.concurrent.*;

public class RPCClient implements AutoCloseable {

    private Connection connection;
    private Channel channel;
    private String requestQueueName = "rpc_queue";

    public RPCClient() throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");

        connection = factory.newConnection();
        channel = connection.createChannel();
    }

    public static void main(String[] argv) {
        try (RPCClient fibonacciRpc = new RPCClient()) {
            for (int i = 0; i < 32; i++) {
                String i_str = Integer.toString(i);
                System.out.println(" [x] Requesting fib(" + i_str + ")");
                String response = fibonacciRpc.call(i_str);
                System.out.println(" [.] Got '" + response + "'");
            }
        } catch (IOException | TimeoutException | InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
    }

    public String call(String message) throws IOException, InterruptedException, ExecutionException {
        final String corrId = UUID.randomUUID().toString();

        String replyQueueName = channel.queueDeclare().getQueue();
        AMQP.BasicProperties props = new AMQP.BasicProperties
                .Builder()
                .correlationId(corrId)
                .replyTo(replyQueueName)
                .build();

        channel.basicPublish("", requestQueueName, props, message.getBytes("UTF-8"));

        final CompletableFuture<String> response = new CompletableFuture<>();

        String ctag = channel.basicConsume(replyQueueName, true, (consumerTag, delivery) -> {
            if (delivery.getProperties().getCorrelationId().equals(corrId)) {
                response.complete(new String(delivery.getBody(), "UTF-8"));
            }
        }, consumerTag -> {
        });

        String result = response.get();
        channel.basicCancel(ctag);
        return result;
    }

    public void close() throws IOException {
        connection.close();
    }
}
```

```java
import com.rabbitmq.client.*;

public class RPCServer {

    private static final String RPC_QUEUE_NAME = "rpc_queue";

    private static int fib(int n) {
        if (n == 0) return 0;
        if (n == 1) return 1;
        return fib(n - 1) + fib(n - 2);
    }

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");

        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        channel.queueDeclare(RPC_QUEUE_NAME, false, false, false, null);
        channel.queuePurge(RPC_QUEUE_NAME);

        channel.basicQos(1);

        System.out.println(" [x] Awaiting RPC requests");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            AMQP.BasicProperties replyProps = new AMQP.BasicProperties
                    .Builder()
                    .correlationId(delivery.getProperties().getCorrelationId())
                    .build();

            String response = "";
            try {
                String message = new String(delivery.getBody(), "UTF-8");
                int n = Integer.parseInt(message);

                System.out.println(" [.] fib(" + message + ")");
                response += fib(n);
            } catch (RuntimeException e) {
                System.out.println(" [.] " + e);
            } finally {
                channel.basicPublish("", delivery.getProperties().getReplyTo(), replyProps, response.getBytes("UTF-8"));
                channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
            }
        };

        channel.basicConsume(RPC_QUEUE_NAME, false, deliverCallback, (consumerTag -> {}));
    }
}
```

```java
/**
 * RPC服务端
 *
 * @author yuanzhihao
 * @since 2020/11/21
 */
public class RPCServer {

    public static void main(String[] args) throws IOException, TimeoutException {
        // 首先还是正常获得connection以及channel对象
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("192.168.1.108");
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/");
        Connection connection = connectionFactory.newConnection();
        Channel channel = connection.createChannel();

        // 定义一个rpc的队列
        String queueName = "test_rpc";
        channel.queueDeclare(queueName, false, false, false, null);

        Object monitor = new Object();
        // 具体的消费代码里面实现
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            // 消费者将请求消息中的correlationId信息再作为响应传回replyTo队列
            AMQP.BasicProperties replyProps = new AMQP.BasicProperties
                    .Builder()
                    .correlationId(delivery.getProperties().getCorrelationId())
                    .build();

            String response = "";
            try {
                // 提供一个大小写转换的方法
                String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
                System.out.println("toUpperCase(" + message + ")");
                response = toUpperCase(message);
            } catch (RuntimeException e) {
                System.out.println(e.toString());
            } finally {
                // 将响应传回replyTo队列
                channel.basicPublish("", delivery.getProperties().getReplyTo(), replyProps, response.getBytes(StandardCharsets.UTF_8));
                // 设置了手动应答 需要手动确认消息
                channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
                // 执行完成会释放主线程的锁
                // RabbitMq consumer worker thread notifies the RPC server owner thread
                synchronized (monitor) {
                    monitor.notify();
                }
            }
        };

        // 监听"test_rpc"队列
        channel.basicConsume(queueName, false, deliverCallback, (consumerTag -> { }));
        // 这个锁对象是确保我们server的调用逻辑执行完成 首先挂起主线程
        // Wait and be prepared to consume the message from RPC client.
        while (true) {
            synchronized (monitor) {
                try {
                    monitor.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    // 提供一个大小写转换的方法
    private static String toUpperCase(String msg) {
        return msg.toUpperCase();
    }
}

```

```java
/**
 * RPC客户端
 *
 * @author yuanzhihao
 * @since 2020/11/21
 */

public class RPCClient {

    public static void main(String[] args) throws IOException, TimeoutException, InterruptedException {
        // 创建connection以及channel对象
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("192.168.1.108");
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/");

        try ( Connection connection = connectionFactory.newConnection();
              Channel channel = connection.createChannel()) {
            // 声明一个队列
            String queueName = "test_rpc";

            // 请求消息中需要带一个唯一标识ID 
            String corrId = UUID.randomUUID().toString();
            // 声明一个回调队列
            String replayQueueName = channel.queueDeclare().getQueue();
            // 将correlationId以及回调队列设置在消息的属性中
            AMQP.BasicProperties properties = new AMQP.BasicProperties
                    .Builder()
                    .correlationId(corrId)
                    .replyTo(replayQueueName)
                    .build();
            // 具体消息内容
            String msg = "hello rpc";
            // 发送请求消息
            channel.basicPublish("",queueName,properties,msg.getBytes());
            // 设置一个阻塞队列  等待服务端的响应
            final BlockingQueue<String> response = new ArrayBlockingQueue<>(1);

            String ctag = channel.basicConsume(replayQueueName, true, (consumerTag, message) -> {
                // 注意 这边根据correlationId进行下判断
                if (message.getProperties().getCorrelationId().equals(corrId)) {
                    response.offer(new String(message.getBody(), StandardCharsets.UTF_8));
                }
            }, consumerTag -> {});

            // 获取响应结果
            String take = response.take();
            System.out.println("rpc result is "+ take);
            channel.basicCancel(ctag);
        }
    }
}
```

#### Publisher Confirms

##### Overview[](https://www.rabbitmq.com/tutorials/tutorial-seven-java#overview) 概述

In this tutorial we're going to use publisher confirms to make sure published messages have safely reached the broker. We will cover several strategies to using publisher confirms and explain their pros and cons.
在本教程中，我们将使用 publisher confirms 来确保发布的消息已安全到达 broker。我们将介绍使用出版商确认的几种策略并解释它们的优缺点。

##### Enabling Publisher Confirms on a Channel[](https://www.rabbitmq.com/tutorials/tutorial-seven-java#enabling-publisher-confirms-on-a-channel) 在频道上启用 Publisher Confirms

Publisher confirms are a RabbitMQ extension to the AMQP 0.9.1 protocol, so they are not enabled by default. Publisher confirms are enabled at the channel level with the `confirmSelect` method:
Publisher 确认是 AMQP 0.9.1 协议的 RabbitMQ 扩展，因此默认情况下不启用它们。发布者确认是使用 `confirmSelect` 方法在渠道级别启用的：

```java
Channel channel = connection.createChannel();
channel.confirmSelect();
```



This method must be called on every channel that you expect to use publisher confirms. Confirms should be enabled just once, not for every message published.
必须在您希望使用 publisher 确认的每个渠道上调用此方法。Confirms 应仅启用一次，而不是针对发布的每条消息启用。

##### Strategy #1: Publishing Messages Individually[](https://www.rabbitmq.com/tutorials/tutorial-seven-java#strategy-1-publishing-messages-individually) 策略 #1：单独发布消息

Let's start with the simplest approach to publishing with confirms, that is, publishing a message and waiting synchronously for its confirmation:
让我们从使用 confirm 发布的最简单方法开始，即发布一条消息并同步等待其确认：

```java
while (thereAreMessagesToPublish()) {
    byte[] body = ...;
    BasicProperties properties = ...;
    channel.basicPublish(exchange, queue, properties, body);
    // uses a 5 second timeout
    channel.waitForConfirmsOrDie(5_000);
}
```



In the previous example we publish a message as usual and wait for its confirmation with the `Channel#waitForConfirmsOrDie(long)` method. The method returns as soon as the message has been confirmed. If the message is not confirmed within the timeout or if it is nack-ed (meaning the broker could not take care of it for some reason), the method will throw an exception. The handling of the exception usually consists in logging an error message and/or retrying to send the message.
在前面的示例中，我们像往常一样发布一条消息，并等待该方法的 `Channel#waitForConfirmsOrDie(long)` 确认。确认消息后，该方法会立即返回。如果消息未在超时内确认，或者如果消息是 nack-ed （意味着代理由于某种原因无法处理它），该方法将引发异常。异常的处理通常包括记录错误消息和/或重试发送消息。

Different client libraries have different ways to synchronously deal with publisher confirms, so make sure to read carefully the documentation of the client you are using.
不同的客户端库有不同的方式来同步处理发布者确认，因此请务必仔细阅读您正在使用的客户端的文档。

This technique is very straightforward but also has a major drawback: it **significantly slows down publishing**, as the confirmation of a message blocks the publishing of all subsequent messages. This approach is not going to deliver throughput of more than a few hundreds of published messages per second. Nevertheless, this can be good enough for some applications.
这种技术非常简单，但也有一个主要缺点：它会**显着减慢发布速度**，因为消息的确认会阻止所有后续消息的发布。这种方法不会提供超过每秒几百条已发布消息的吞吐量。尽管如此，这对于某些应用程序来说已经足够好了。

> #### Are Publisher Confirms Asynchronous?[](https://www.rabbitmq.com/tutorials/tutorial-seven-java#are-publisher-confirms-asynchronous) 发布者确认是否异步？[](https://www.rabbitmq.com/tutorials/tutorial-seven-java#are-publisher-confirms-asynchronous)
>
> We mentioned at the beginning that the broker confirms published messages asynchronously but in the first example the code waits synchronously until the message is confirmed. The client actually receives confirms asynchronously and unblocks the call to `waitForConfirmsOrDie` accordingly. Think of `waitForConfirmsOrDie` as a synchronous helper which relies on asynchronous notifications under the hood.
> 我们在开头提到过，代理异步确认已发布的消息，但在第一个示例中，代码同步等待，直到消息得到确认。客户端实际上以异步方式接收 confirms，并相应地取消阻止对 `waitForConfirmsOrDie` 的调用。将 `waitForConfirmsOrDie` 视为一个同步帮助程序，它依赖于后台的异步通知。

##### Strategy #2: Publishing Messages in Batches[](https://www.rabbitmq.com/tutorials/tutorial-seven-java#strategy-2-publishing-messages-in-batches) 策略 #2：批量发布消息

To improve upon our previous example, we can publish a batch of messages and wait for this whole batch to be confirmed. The following example uses a batch of 100:
为了改进前面的示例，我们可以发布一批消息并等待整个批次得到确认。以下示例使用一批 100：

```java
int batchSize = 100;
int outstandingMessageCount = 0;
while (thereAreMessagesToPublish()) {
    byte[] body = ...;
    BasicProperties properties = ...;
    channel.basicPublish(exchange, queue, properties, body);
    outstandingMessageCount++;
    if (outstandingMessageCount == batchSize) {
        channel.waitForConfirmsOrDie(5_000);
        outstandingMessageCount = 0;
    }
}
if (outstandingMessageCount > 0) {
    channel.waitForConfirmsOrDie(5_000);
}
```



Waiting for a batch of messages to be confirmed improves throughput drastically over waiting for a confirm for individual message (up to 20-30 times with a remote RabbitMQ node). One drawback is that we do not know exactly what went wrong in case of failure, so we may have to keep a whole batch in memory to log something meaningful or to re-publish the messages. And this solution is still synchronous, so it blocks the publishing of messages.
等待一批消息得到确认比等待确认单个消息的吞吐量大大提高（使用远程 RabbitMQ 节点时最多可提高 20-30 次）。一个缺点是，在失败的情况下，我们不知道到底出了什么问题，因此我们可能必须在内存中保留一整批来记录有意义的内容或重新发布消息。并且此解决方案仍然是同步的，因此它会阻止消息的发布。

##### Strategy #3: Handling Publisher Confirms Asynchronously[](https://www.rabbitmq.com/tutorials/tutorial-seven-java#strategy-3-handling-publisher-confirms-asynchronously) 策略 #3：异步处理发布者确认

The broker confirms published messages asynchronously, one just needs to register a callback on the client to be notified of these confirms:
broker 异步确认已发布的消息，只需在客户端上注册一个回调即可收到这些确认的通知：

```java
Channel channel = connection.createChannel();
channel.confirmSelect();
channel.addConfirmListener((sequenceNumber, multiple) -> {
    // code when message is confirmed
}, (sequenceNumber, multiple) -> {
    // code when message is nack-ed
});
```



There are 2 callbacks: one for confirmed messages and one for nack-ed messages (messages that can be considered lost by the broker). Each callback has 2 parameters:
有 2 个回调：一个用于已确认的消息，一个用于 nack-ed 消息（可以被代理视为丢失的消息）。每个回调有 2 个参数：

- sequence number: a number that identifies the confirmed or nack-ed message. We will see shortly how to correlate it with the published message.
  序列号：标识已确认消息或 nack-ed 消息的数字。我们很快就会看到如何将其与发布的消息相关联。
- multiple: this is a boolean value. If false, only one message is confirmed/nack-ed, if true, all messages with a lower or equal sequence number are confirmed/nack-ed.
  multiple：这是一个布尔值。如果为 false，则只确认/nack-ed 一条消息，如果为 true，则确认/nack-ed 序列号较低或相等的所有消息。

The sequence number can be obtained with `Channel#getNextPublishSeqNo()` before publishing:
在发布之前，可以通过 `Channel#getNextPublishSeqNo（）` 获取序列号：

```java
int sequenceNumber = channel.getNextPublishSeqNo());
ch.basicPublish(exchange, queue, properties, body);
```



A simple way to correlate messages with sequence number consists in using a map. Let's assume we want to publish strings because they are easy to turn into an array of bytes for publishing. Here is a code sample that uses a map to correlate the publishing sequence number with the string body of the message:
将消息与 sequence number 相关联的一种简单方法是使用 map。假设我们想要发布字符串，因为它们很容易变成字节数组进行发布。下面是一个代码示例，该代码示例使用 map 将发布序列号与消息的字符串正文相关联：

```java
ConcurrentNavigableMap<Long, String> outstandingConfirms = new ConcurrentSkipListMap<>();
// ... code for confirm callbacks will come later
String body = "...";
outstandingConfirms.put(channel.getNextPublishSeqNo(), body);
channel.basicPublish(exchange, queue, properties, body.getBytes());
```



The publishing code now tracks outbound messages with a map. We need to clean this map when confirms arrive and do something like logging a warning when messages are nack-ed:
发布代码现在使用映射跟踪出站消息。我们需要在确认到达时清理这张地图，并做一些事情，比如在消息被 nack-ed 时记录警告：

```java
ConcurrentNavigableMap<Long, String> outstandingConfirms = new ConcurrentSkipListMap<>();
ConfirmCallback cleanOutstandingConfirms = (sequenceNumber, multiple) -> {
    if (multiple) {
        ConcurrentNavigableMap<Long, String> confirmed = outstandingConfirms.headMap(
          sequenceNumber, true
        );
        confirmed.clear();
    } else {
        outstandingConfirms.remove(sequenceNumber);
    }
};

channel.addConfirmListener(cleanOutstandingConfirms, (sequenceNumber, multiple) -> {
    String body = outstandingConfirms.get(sequenceNumber);
    System.err.format(
      "Message with body %s has been nack-ed. Sequence number: %d, multiple: %b%n",
      body, sequenceNumber, multiple
    );
    cleanOutstandingConfirms.handle(sequenceNumber, multiple);
});
// ... publishing code
```



The previous sample contains a callback that cleans the map when confirms arrive. Note this callback handles both single and multiple confirms. This callback is used when confirms arrive (as the first argument of `Channel#addConfirmListener`). The callback for nack-ed messages retrieves the message body and issues a warning. It then re-uses the previous callback to clean the map of outstanding confirms (whether messages are confirmed or nack-ed, their corresponding entries in the map must be removed.)
前面的示例包含一个回调，用于在 confirms 到达时清理 Map。请注意，此回调同时处理单个和多个确认。该回调在 confirms 到达时使用（作为 `Channel#addConfirmListener` 的第一个参数）。nack-ed 消息的回调检索消息正文并发出警告。然后，它会重用之前的回调来清理未完成的确认映射（无论是已确认的消息还是 nack-ed，都必须删除它们在映射中的相应条目。

> #### How to Track Outstanding Confirms?[](https://www.rabbitmq.com/tutorials/tutorial-seven-java#how-to-track-outstanding-confirms) 如何跟踪未完成的确认？[](https://www.rabbitmq.com/tutorials/tutorial-seven-java#how-to-track-outstanding-confirms)
>
> Our samples use a `ConcurrentNavigableMap` to track outstanding confirms. This data structure is convenient for several reasons. It allows to easily correlate a sequence number with a message (whatever the message data is) and to easily clean the entries up to a given sequence id (to handle multiple confirms/nacks). At last, it supports concurrent access, because confirm callbacks are called in a thread owned by the client library, which should be kept different from the publishing thread.
> 我们的示例使用 `ConcurrentNavigableMap` 来跟踪未完成的确认。这种数据结构很方便，原因有几个。它允许轻松地将序列号与消息相关联（无论消息数据是什么），并轻松清理条目，直到给定的序列 ID（以处理多个确认/nack）。最后，它支持并发访问，因为 confirm 回调是在 client 库拥有的线程中调用的，这应该与发布线程保持不同。
>
> There are other ways to track outstanding confirms than with a sophisticated map implementation, like using a simple concurrent hash map and a variable to track the lower bound of the publishing sequence, but they are usually more involved and do not belong to a tutorial.
> 除了使用复杂的 map 实现之外，还有其他方法可以跟踪未完成的确认，例如使用简单的并发哈希 map 和变量来跟踪发布序列的下限，但它们通常涉及更多，不属于教程。

To sum up, handling publisher confirms asynchronously usually requires the following steps:
综上所述，异步处理 publisher 确认通常需要以下步骤：

- provide a way to correlate the publishing sequence number with a message.
  提供一种将发布序列号与消息相关联的方法。
- register a confirm listener on the channel to be notified when publisher acks/nacks arrive to perform the appropriate actions, like logging or re-publishing a nack-ed message. The sequence-number-to-message correlation mechanism may also require some cleaning during this step.
  在通道上注册一个确认侦听器，以便在发布者 ACK/NACK 到达时收到通知，以执行适当的操作，例如记录或重新发布 NACK 编辑的消息。在此步骤中，序列号与消息关联机制可能还需要进行一些清理。
- track the publishing sequence number before publishing a message.
  在发布消息之前跟踪发布序列号。

> #### Re-publishing nack-ed Messages?[](https://www.rabbitmq.com/tutorials/tutorial-seven-java#re-publishing-nack-ed-messages) 重新发布 nack-ed Messages？[](https://www.rabbitmq.com/tutorials/tutorial-seven-java#re-publishing-nack-ed-messages)
>
> It can be tempting to re-publish a nack-ed message from the corresponding callback but this should be avoided, as confirm callbacks are dispatched in an I/O thread where channels are not supposed to do operations. A better solution consists in enqueuing the message in an in-memory queue which is polled by a publishing thread. A class like `ConcurrentLinkedQueue` would be a good candidate to transmit messages between the confirm callbacks and a publishing thread.
> 从相应的回调重新发布 nack-ed 消息可能很诱人，但应该避免这种情况，因为确认回调是在 I/O 线程中调度的，而通道不应该执行操作。更好的解决方案是将消息排入由发布线程轮询的内存队列中。像 `ConcurrentLinkedQueue` 这样的类非常适合在确认回调和发布线程之间传输消息。

##### Summary[](https://www.rabbitmq.com/tutorials/tutorial-seven-java#summary) 总结

Making sure published messages made it to the broker can be essential in some applications. Publisher confirms are a RabbitMQ feature that helps to meet this requirement. Publisher confirms are asynchronous in nature but it is also possible to handle them synchronously. There is no definitive way to implement publisher confirms, this usually comes down to the constraints in the application and in the overall system. Typical techniques are:
确保发布的消息能够到达 Broker 在某些应用程序中可能至关重要。Publisher 确认是有助于满足此要求的 RabbitMQ 功能。发布者确认本质上是异步的，但也可以同步处理它们。没有明确的方法来实现发布者确认，这通常归结为应用程序和整个系统中的约束。典型的技术是：

- publishing messages individually, waiting for the confirmation synchronously: simple, but very limited throughput.
  单独发布消息，同步等待确认：简单，但吞吐量非常有限。
- publishing messages in batch, waiting for the confirmation synchronously for a batch: simple, reasonable throughput, but hard to reason about when something goes wrong.
  批量发布消息，等待批量同步确认：简单、合理的吞吐量，但很难推断何时出现问题。
- asynchronous handling: best performance and use of resources, good control in case of error, but can be involved to implement correctly.
  异步处理：最佳性能和资源使用，在出现错误时具有良好的控制力，但可以参与正确实施。

##### Putting It All Together[](https://www.rabbitmq.com/tutorials/tutorial-seven-java#putting-it-all-together) 把它们放在一起

The [`PublisherConfirms.java`](https://github.com/rabbitmq/rabbitmq-tutorials/blob/main/java/PublisherConfirms.java) class contains code for the techniques we covered. We can compile it, execute it as-is and see how they each perform:
[`PublisherConfirms.java`](https://github.com/rabbitmq/rabbitmq-tutorials/blob/main/java/PublisherConfirms.java) 类包含我们介绍的技术的代码。我们可以编译它，按原样执行它，看看它们各自的表现如何：

```bash
javac -cp $CP PublisherConfirms.java
java -cp $CP PublisherConfirms
```



The output will look like the following:
输出将如下所示：

```bash
Published 50,000 messages individually in 5,549 ms
Published 50,000 messages in batch in 2,331 ms
Published 50,000 messages and handled confirms asynchronously in 4,054 ms
```



The output on your computer should look similar if the client and the server sit on the same machine. Publishing messages individually performs poorly as expected, but the results for asynchronously handling are a bit disappointing compared to batch publishing.
如果客户端和服务器位于同一台计算机上，则计算机上的输出应看起来相似。单独发布消息的性能不如预期，但与批量发布相比，异步处理的结果有点令人失望。

Publisher confirms are very network-dependent, so we'd better off trying with a remote node, which is more realistic as clients and servers are usually not on the same machine in production. `PublisherConfirms.java` can easily be changed to use a non-local node:
Publisher 确认非常依赖于网络，因此我们最好尝试使用远程节点，这样更现实，因为客户端和服务器在生产中通常不在同一台机器上。`PublisherConfirms.java` 可以很容易地更改为使用非本地节点：

```bash
static Connection createConnection() throws Exception {
    ConnectionFactory cf = new ConnectionFactory();
    cf.setHost("remote-host");
    cf.setUsername("remote-user");
    cf.setPassword("remote-password");
    return cf.newConnection();
}
```



Recompile the class, execute it again, and wait for the results:
重新编译类，再次执行，然后等待结果：

```bash
Published 50,000 messages individually in 231,541 ms
Published 50,000 messages in batch in 7,232 ms
Published 50,000 messages and handled confirms asynchronously in 6,332 ms
```



We see publishing individually now performs terribly. But with the network between the client and the server, batch publishing and asynchronous handling now perform similarly, with a small advantage for asynchronous handling of the publisher confirms.
我们看到单独发布现在表现得很糟糕。但是，在客户端和服务器之间的网络中，批处理发布和异步处理现在的执行方式类似，但对发布者的异步处理有一个小优势。

Remember that batch publishing is simple to implement, but does not make it easy to know which message(s) could not make it to the broker in case of negative publisher acknowledgment. Handling publisher confirms asynchronously is more involved to implement but provide better granularity and better control over actions to perform when published messages are nack-ed.
请记住，批量发布很容易实现，但在发布者确认是否定的情况下，无法轻松知道哪些消息无法到达代理。异步处理发布者确认的实施更复杂，但提供了更好的粒度，并更好地控制了在已发布消息过时要执行的操作。