# jms-loadrunner

## Setting Up of JMS in LoadRunner

JMS (Java Message Service) is a Java-based API that provides a way for applications to communicate with each other asynchronously through a messaging system. It allows applications to send and receive messages between them, without having to know each other's details or location.

JMS is based on a publish-subscribe or point-to-point messaging model, where a sender sends messages to a specific destination (queue or topic) and one or more receivers can consume the messages from the destination. This messaging model enables decoupling of applications, which means that applications can operate independently of each other and communicate only when necessary.

JMS also supports message persistence, which means that messages can be stored in a durable store and retrieved later, ensuring that no messages are lost in case of network failure or system outage. JMS also provides a mechanism for transactions, allowing applications to group multiple JMS operations into a single atomic transaction.

JMS is widely used in enterprise applications for asynchronous messaging, and it is supported by many JMS providers such as Apache ActiveMQ, IBM MQ, and Oracle Advanced Queuing.

> There are two types of messaging domains in JMS.
> - Point-to-Point Messaging Domain
> - Publisher/Subscriber Messaging Domain

> Point-to-Point (PTP) Messaging Domain
> In PTP model, one message is delivered to one receiver only. Here, Queue is used as a message oriented middleware (MOM).
> The Queue is responsible to hold the message until receiver is ready.
> In PTP model, there is no timing dependency between sender and receiver.
 
> Publisher/Subscriber (Pub/Sub) Messaging Domain
> In Pub/Sub model, one message is delivered to all the subscribers. It is like broadcasting. Here, Topic is used as a message oriented middleware that is responsible > to hold and deliver messages.
> In PTP model, there is timing dependency between publisher and subscriber.
 
Before you begin setting up JMS in LoadRunner and Neoload, you will need to ensure that you have the following prerequisites:

- A basic understanding of JMS (Java Messaging Service)

- A working knowledge of LoadRunner

- Access to the JMS provider you want to test (e.g., Apache ActiveMQ, IBM MQ, Oracle Advanced Queuing)

- JMS client libraries for the JMS provider you want to test

- A Java Development Kit (JDK) installed on your machine

- Knowledge of how to configure JMS client libraries in Neoload

## Tools

To set up JMS in LoadRunner , you will need the following tools:

- Loadrunner 12.55

- JMS client libraries for the JMS provider you want to test

- [jboss-client.jar](https://github.developer.allianz.io/ajin-sudhir/jms-neoload/blob/main/jboss-client.jar)

- A Java Development Kit (JDK) installed on your machine

## Process

The process for setting up JMS in LoadRunner  involves the following steps:

-	Download the JMS client libraries for the JMS provider you want to test and add them to your LoadRunner project.

-	Configure the connection factory, destination, and other JMS-related properties.

  
## Steps

-	Create a script in LoadRunner that uses the JMS client libraries to send and receive messages

-	Run the script or scenario to test the JMS provider.

### LoadRunner 12.55 
 
## Webservice Protocol

```shell
Action()
{
   jms_send_message_queue("step1: send message",   lr_eval_string("{NewParam}"),
    "java:/jms/queue/RISK.EU.IN.WRITE");
    jms_receive_message_queue("step2: receive message",  "java:/jms/queue/RISK.EU.JOBS");
    lr_message(lr_eval_string("{JMS_message}"));
    return 0;
}

 ```
## Java Vuser Protocol

```shell

/*
 * LoadRunner Java script. (Build: _build_number_)
 * 
 * Script Description: 
 *                     
 */

import lrapi.lr;
import javax.jms.*;
import javax.naming.*;
import java.util.Hashtable;
import java.nio.file.Paths;
import java.nio.file.Files;
import java.nio.charset.StandardCharsets;



public class Actions
{

    public int init() throws Throwable {
        return 0;
    }//end of init


    public int action() throws Throwable {
        
        String providerUrl ="remote://*******.allianz:*****/";
        String factoryClassName="org.jboss.naming.remote.client.InitialContextFactory";
        String connectionFactoryName="jms/RemoteConnectionFactory";
        String destinationName="java:/jms/queue/*******";
        String destinationName2="java:/jms/queue/*******";
        String username="*******";
        String password="*******";
        
        String xmlFilePath = "C:\\Users\\******\\Documents\\VuGen\\Scripts\\JavaVuser1\\****.xml";
        String xmlContent = new String(Files.readAllBytes(Paths.get(xmlFilePath)),StandardCharsets.UTF_8);
        
        
        Hashtable<String,String>env=new Hashtable<String,String>();
        
        env.put(Context.INITIAL_CONTEXT_FACTORY,factoryClassName);
        env.put(Context.PROVIDER_URL,providerUrl);
        env.put(Context.SECURITY_PRINCIPAL,username);
        env.put(Context.SECURITY_CREDENTIALS,password);
        env.put(Context.URL_PKG_PREFIXES, "org.jboss.ejb.client.naming");
        env.put("jboss.naming.client.connect.timeout","30000");
        
        InitialContext initialContext=new InitialContext(env);
                
    

        ConnectionFactory connectionFactory=(ConnectionFactory)initialContext.lookup(connectionFactoryName);
        
        Queue queue = (Queue)initialContext.lookup(destinationName);
        Queue queue2 = (Queue)initialContext.lookup(destinationName2);
            
        
        Connection connection = connectionFactory.createConnection();
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        MessageProducer producer = session.createProducer(queue);
              
        TextMessage message = session.createTextMessage(xmlContent);
        producer.send(message);
      //  System.out.println("Sent message: " + message.getText());
  
        MessageConsumer messageConsumer = session.createConsumer(queue2);

   
        connection.start();

        // Step 9. Receive the message
        TextMessage messageReceived = (TextMessage)messageConsumer.receive(900000);
        System.out.println(messageReceived);
        System.out.println("Received message: " + messageReceived.getText());

        // Finally, we clean up all the JMS resources
        connection.close();
         
        initialContext.close();        
        return 0;
    }//end of action


    public int end() throws Throwable {
        return 0;
    }//end of end
}


```
 

