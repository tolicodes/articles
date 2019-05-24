

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
| 2  | 2					| 3						|
| 3  | 2					| 4						|
| 4  | 3					| 5						|
| 5  | 4					| 5						|

And then we just dump it into D3's [Graph](https://bl.ocks.org/cjrd/6863459) and we've got ourselves a visual representation. It even has a graph builder so that you can generate and rearrange graphs and save it to the DB. 

Now we have documentation of all of our services. Yay!

### Discovery of Services
We can make this even easier once we move to K8S by using its [Discovery API](https://medium.com/technology-matters/service-discovery-with-kubernetes-a503b16e71a0) which will list all the services available. Then you just use the tool to drag and drop them in the right order.

## Source of Truth for Messaging - The DevOps Message Broker
**OK. Here is where it gets interesting!!!**
Why document...when this service can become THE source of truth for messaging! 

All we have to do is make a little tweak on top of SNS/SQS...by adding a Broker. Usually in SNS posts a specific topic like `image_processing_done` and any service can subscribe to it. But then every service that subsribes to it needs to know the name of the topic. What if I create a service which doesn't really care where the image comes from? Perhaps we want to arbitrarily add and remove services at different parts of the process and quickly enable/disable/replace them. We can do that if we have a message broker.

All we have to do is add these two lines to each service:

    pubsub.publish('output:<THE SERVICES ID>:<output type>', payload);
    queue.process('input:<THE SERVICES ID>:<input type>', () => { });

And now the message broker, which dynamically associates all the nodes together based on the database essentially does for each `relationship` in the db:

    queue.proccess('output:<publisher_service_id>:<input_type>', () => {
      pubsub.publish('input:<subscriber_service_id>:<input_type>
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE4ODI4NzYyMjEsMTc0NzAyNTkzMCwtNz
Q5Mzg1MTAxLDE1MzY5MzE4MzddfQ==
-->