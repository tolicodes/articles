

# Intro

Colin has a brilliant idea for building a dynamic message broker for our services infrastructure. Multiple HOVER engineers, including Elise, Dave, Max, and Ethan have described versions of this architecture. But what’s different about Colin’s idea is that it gives a simple, graphical, extendable, and dynamic way to manage how our services are interconnected. It solves many of the largest problems that a microservice architecture presents including service discovery, schema management, message pollution, logging, and error handling.

# Challenges in a Micro Server World

## Message Brokerage

Let’s start with the most common model of how microservices work -- RESTful APIs. One server calls another server’s REST endpoint. The other server accepts it and interprets it Easy...until you have hundreds of services all interconnected, and suddenly you’re maintaining potentially REST calls in each application with nobody in the company really understanding which services talk to each other and where that code lies. Services go down, change APIs, and suddenly your perfect microservice world comes to a halt.

### PubSub - Managing API Endpoints 
PubSub (via services like SNS or Kafka) alleviates some issues. The concept is extremely simple. There are only two things you can do:

**Publish**
You publish a message to a topic with a payload. 

    pubsub.publish('image-processing-complete', { image_url: 'http://aws.com/image.jpg' });

**Subscribe**
You subscribe to topics you're interested in with a callback

    pubsub.subscribe('image-processing-complete', (payload) => { 
      console.log(payload.image);
    });
 
Simple right? Now Service A doesn't need to know about Service B. It just publishes a message, and whatever service is interested subscribes to it. No REST calls needed. No need t

### Message Queueing 

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTI4MjI5MDI4NV19
-->