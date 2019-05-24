

# Intro

Colin has a brilliant idea for building a dynamic message broker for our services infrastructure. Multiple HOVER engineers, including Elise, Dave, Max, and Ethan have described versions of this architecture. But what’s different about Colin’s idea is that it gives a simple, graphical, extendable, and dynamic way to manage how our services are interconnected. It solves many of the largest problems that a microservice architecture presents including service discovery, schema management, message pollution, logging, and error handling.

This really is a combination of existing solutions such as SNS/Kafka, SQS, Kibana, etc and a creation of a new service called a Message Broker.

# Existing Solutions to Problems

## Message Brokerage

Let’s start with the most common model of how microservices work -- RESTful APIs. One server calls another server’s REST endpoint. The other server accepts it and interprets it Easy...until you have hundreds of services all interconnected, and suddenly you’re maintaining potentially REST calls in each application with nobody in the company really understanding which services talk to each other and where that code lies. Services go down, change APIs, and suddenly your perfect microservice world comes to a halt.

Elise started describing pubsub [here]([https://hoverinc.atlassian.net/wiki/spaces/EN/pages/846790813/PubSub+Proposal+1](https://hoverinc.atlassian.net/wiki/spaces/EN/pages/846790813/PubSub+Proposal+1)). It took me a while to understand how they work, but now that I do I'll dumb it down. 

### PubSub - Managing API Endpoints 
PubSub (via services like Amazon SNS or Kafka) alleviates some issues. The concept is extremely simple. There are only two things you can do:

**Publish**
You publish a message to a topic with a payload. 

    pubsub.publish('image-processing-complete', { image_url: 'http://aws.com/image.jpg' });

**Subscribe**
You subscribe to topics you're interested in with a callback

    pubsub.subscribe('image-processing-complete', (payload) => { 
      console.log(payload.image);
    });
 
Simple right? Now Service A doesn't need to know about Service B. It just publishes a message, and whatever service is interested subscribes to it. No REST calls needed. No need to set up an express server to listen on those endpoints

### Message Queueing - Services that Are Down, Async Messaging
What if a service is down? Retry logic is hard to build. Not every engineer has the time to write fibonacci backoff sequences. There's an elegant solution called a message queue. An example is Amazon SQS. The simplest way I've heard it described in an email inbox. I don't have to be constantly looking at my inbox. I just check my email when I have time, and mark messages as read. Your PubSub can write to a message queue, and whatever service consumes it can delete it when it's been processed. If your recieving service is down, no problem, the messages are waiting for it when the service restarts (automatically or manually) and the workflow continues as normal. 

Essentially all I have to do is:

     queue.process('image-processing-complete', (newMessage) => {
        // do some processing
        queue.delete(newMessage);
     });

So now we've solved async messaging and services going down.

## Logging
There are many solutions to logging, but the best one I've seen is funneling your SNS to something like [Kibana](https://www.elastic.co/products/kibana), a log analyzer. Now you can observe and visualize all of the inputs and outputs to your services, search through them, query them, you name it. Check out their website, it's f***ing cool.

# Problems DevOpsBroker Solves
Now for Colin's idea. We still have a few problems to solve

## Documenting Service Interaction
Let's take a fake ManoWar Flow (I don't know what actually happens). It would be nice if we could make a  directed flow diagram. The nodes represent services. They accept some input, do something (either automatically or via a human being interaction, and at some point output something that's recieved by another node. In some cases we might even have one process send an output to 2+ processes (ex: normalized image is sent to modeler and ML).

It would be nice if we had an up to date process diagram of every service, it's input, and output, and how they are connnected to each other. It would look something like this:

![Process Flow](https://drive.google.com/uc?export=view&id=17ekCf1mhVNvk6bF9bpZF-XboC7DreUsq)

Of course we have a lot more processes and a lot more flows (ex: what happens in design pro, vray, etc). There's no document that shows all of this. We could of course draw it in LucidCharts, but likely it will go out of date. 

So let's start with a simple service that lets you represent all of our services and how they're connected. All we need is a simple database:

**services**
| id | name             | input_types 	   | output_types    				 |
|--  |--                |--                |--                               |
| 1  | Image Uploader   | null      	   | image          				 |
| 2  | Image Normalizer | image      	   | normalized_image 				 |
| 3  | ML Stuff			| normalized_image | labels           				 |
| 4  | Modeler			| normalized_image | 3d_model	 				     |
| 5 .| Combiner        	| labels,3d_model  | 3d_model_json, completed_jpg .  |

**relationships**
| id | publisher_service_id | subscriber_service_id |
| -- | --                   | --                    |
| 1  | 1    				| 2						|
| 2  | 2					| 3
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTQ0OTk4MzU1NSwxNzQ3MDI1OTMwLC03ND
kzODUxMDEsMTUzNjkzMTgzN119
-->