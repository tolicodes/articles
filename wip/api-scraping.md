		
# Abstract
This will be a complete guide to dealing with API Scraping. I explain how I came up with my [api-tookit](https://github.com/tolicodes/node-api-toolkit) and the [twitter-toolkit](https://github.com/tolicodes/twitter-toolkit) based on it. 

# Outline
## Intro
I’ve done a few projects now that involve API scraping of some sort - Twitter, AWS, Google, Medium, JIRA, you name it. It’s a fairly common task when you’re a freelance developer. Throughout these implementations I’ve used a few libraries, including bottleneck, promise-queue, and just rolling my own. However none of the existing solutions covered every aspect of scraping. 

So, I created my own solution, [api-tookit](https://github.com/tolicodes/node-api-toolkit) and the [twitter-toolkit](https://github.com/tolicodes/twitter-toolkit) based on it, which I believe solves 90% of the challenges you will encounter in scraping your own APIs. 

I feel it is important to understand the Fundementals behind some of these concepts, so I decided to write this article for CodeMentor. 

API scraping has many challenges, but we will be focusing on what I believe are the major ones in this article. 

### Rate Limiting
Just about API you will be hitting (public or private) will come with 2 types of rate limiting

- DDOS protection: almost every production API, if you start hitting it with 1000 requests per second will block your IP. This means your server will be prohibited from accessing the API, potentially indefinitely. This is meant to prevent DDOS attacks (or denial of service). Unfortunately, it’s quite easy to inadvertently trigger these protection if you’re not careful, especially if you are using multiple servers (clustering). 
- Standard Rate Limiting: most APIs will limit either your IP or your API key to a certain amount of requests during a certain timeframe (ex: 180 per 15m). These limits may be different for different for different endpoints for a single service. 

### Error Handling
Errors happen. A lot. Common errors include
- Rate limiting: even if you are careful sometimes a rate limiting error still occurs. You need a strategy to retry
- not found: different APIs handle not found errors in different ways. Some throw a 404. Your application might not care if something is not found. 
- Other errors: you may want to report every error that happens without crashing your app 

### Pagination
This is a common issue with very long sets of results. Generally there are two approaches

- cursor: a pointer to the next record returned by the last record 
- page: standard pagination, You keep passing page numbers sequentially until there are no more results 
 
### Concurrency
Especially if the results are quite large (images, files), you probably want to have some sort of concurrency. That is multiple requests happening at the same time. But also, you don’t want too many concurrent requests to prevent rate limiting/DOS protection. 

### Logging and Debugging
There is so much that can go wrong when scraping an API. For this you need an effective logging and debugging strategy. I have created a progress bar with some messaging indicating what’s going on at any point in a visual way. 

## The Basics
### Key/Secret Management
Almost every private API will have some sort of Private key system (essentially a password that’s easily revocable). The implemenatations vary greatly, but generally they require you to store one or more pieces of “secret” text somewhere. 

Never put secrets in your repository. Even if the repo is private, it is so easy for your secrets to get leaked accidentally. If this happens your account will be hijacked and you will be responsible for anything that happens on it. This includes posts made on Behalf of your company, stolen user information, and any billing that may occur from use of the API. 

Instead, there are a few options:

#### .env file
An .env file is simple file that is excluded from the git repository and is manually copied to your computer and sever. An environment file is great for your dev computer or servers that you configure manually. 

To do this create a file called `.env` in the root of your project. Then add your secrets:
```
API_KEY = asdfhsd834hsd
API_SECRET = ioshfa94widoj2ws
```

Then install [ `dotenv`](https://www.npmjs.com/package/dotenv)

```
yarn add dotenv
```

At the top of your node file add:
```
require('dotenv').config();;

const {
   API_KEY,
   API_SECRET
} = process.env;
```

Finally make sure that you exclude the file from your .GIT repository (and manually copy it anywhere that needs to use the secrets)

`.gitignore`
```
.env
```

#### enviornmental variables
Environmental variables are easily manageable in most environments that run Docker Containers (ex: AWS). They are one of the more secure and simple way to go.

Esseniallly, all you have to do is manually set variables in your shell environment. Or in your container provider's config, such as [AWS ECS](https://stackoverflow.com/questions/43330278/how-to-provide-environment-variables-to-aws-ecs-task-definition) (to persist the variable across restarts).

To get this working on your local machine,

type in your bash environment:
```
export API_KEY=sn89ds2ju93sdnljos
```

or add that line to `~/.bashrc`

And then in your node file, to access the variable just do 

```
const {
    API_KEY
} = process.env;
```
  
### Libraries vs Roll Your Own 
Generally, when you are going to scraping a well known API, such as Twitter, there will be [multiple](https://www.npmjs.com/package/twitter) [node](https://www.npmjs.com/package/node-twitter-api) [packages](https://github.com/ttezel/twit) to choose from. I strongly suggest using those packages, at least in the beginning because they take care of:
* authentication
* the http request headers and body formatting
* api specific peculiarities 

Once you start using the more advanced parts of the API you may want to fork the repository to make it fit your needs, or even write your own.

If you are using a lesser supported API, you may be forced to roll your own implementation. 

In this case you have an option of using ES6's `fetch` function or an exisiting library such as [axios](https://github.com/axios/axios). I like axios because it supports Promises and has a clean syntax. 

Once you have decided on your HTTP request library you may want to create a wrapper for creating requests, so that you do not have to type the same info over and over. Axios has an ability to create an "instance" that serves the same purpose.

For example:

```
const instance = axios.create({
	 baseURL: 'https://twitter.com/api/',
	 headers: {
		'X-Bearer-Token: API_TOKEN
	 }
});

// later using the instance
instance.get('/users');
```

## Setting up our example app - Twitter scraper
For this tutorial we are going to be using the Twitter API with the [`twit`](https://github.com/ttezel/twit) client.

Set up your app as follows.

1. Set up a [developer account](https://developer.twitter.com/en/account/get-started) and get your account secrets on the Twitter website
2. Add your twitter credentials to your `.env` file or environmental variables
3. Run 
	```
	yarn add twit
	```
4. Add an object where we will keep our code
	```
	const twitterScraper = {
	   init: async () => {	
	
	   }
	}
	twitterScraper.init();
	```
5. Configure the Twit client referencing your .env variables
	```
	init: async () => {	
		this.client  =  new  Twit({
			consumer_key:  CONSUMER_KEY,
			consumer_secret:  CONSUMER_SECRET,
			access_token:  ACCESS_TOKEN,
			access_token_secret:  ACCESS_TOKEN_SECRET,
		});
	}
	```

## API Scraping Concepts
Now we are going to start building our scraper one concept at a time.

###  Building a simple queue
We are going to create a simple Queue with 4 states:
* `queued`: waiting to execute
* `pending`: currently executing
* `complete`: successfully executed
* `failed`: failed to execute

We are going to use Promises in this queue. Since we cannot "pause" a Promise, we are going to have to wrap the function to be executed in a second Promise. It will be resolved at the same time that the internal function promise will be executed. But it will be pending before we start executing the internal function.

So:
1. External Promise is created
2. Extenal Promise is pending
3. Queue starts processing and Internal Function is queued up
4. Internal Function is executed
5. Internal Promise Resolves
6. External Promise Resolves

What happens in the `add` function is we sign

```
class Queue {
  constructor() {
    Object.assign(this, {
      // lists of promises
      queued: [],
	  pending: [],
	  complete: [],
      failed: [],
	
	  // unlike queued this is a list of functions, not promises
	  queuedFuncs: [],
	  
	  stopped:  true,
    });
  }

  // there will be a promise added to queued immediately after adding
  // but func will ojnly start executing when `process` is called
  // the wrapper promise will actually be the one that is passed around everywhere
  // the queue will start executing if it's currently stopped if autoStart
  // is enabled;

  add(func) {
	 let  resolve;
     let  reject;

	 const  wrapperPromise = new Promise((res, rej) => {
		resolve = res;
		reject = rej;
	 });
	 
     this.queued.push(wrapperPromise);

     this.queuedFuncs.push(() => {
       func().then(resolve).catch(reject);
       return wrapperPromise;
     });

     setTimeout(() => {
       if (this.stopped  &&  this.autoStart) {
	     this.process();
	   }
     });

     return wrapperPromise;
  }
}
```
 
Now we actually need to write the `process` function that starts off the process. Essentially we have an async `while` loop, that processes one item at a time. For now, there is nothing asynchronous happening...but we will put some blocks in to stop processing if there are too many requests.

All our function is doing is:
1. executing the next function in the `queuedFuncs` list (`this.queuedFuncs.shift()`)
2. moving the Wrapper promise from the `queued` list to the `pending` list.
3. After all the requests have been added to `pending` list, the function will wait for all the requests in the `pending` list to resolve and return their results 

Notice we wrapper the execution in a `try catch` block. But we aren't actually doing anything in the `catch`. This will let all the errors that happen pass through for now.

```
moveLists(item, from, to) {
  this[to].push(item);
  this[from].splice(this[from].indexOf(item), 1);
}

async processNextItem() {
   if (!this.queued.length) { return  false; }

   this.moveLists(this.queued[0], 'queued', 'pending');
   
   let promise;

   try {
	 promise = this.queuedFuncs.shift()();
	 await promise;
	 this.moveLists(promise, 'pending', 'complete');
   } catch (e) {}
}

async process() {
  while (this.queuedFuncs.length) {
    await this.processNextItem();
  }
  
  return Promise.all(this.pending);
}
```
 
### Adding logging
Logging is extremely important. After all we need to see what's going on for long running queues...We want to see how many items are done processing, how many are still left. We will set up an advanced logger later, but for now, let's just set up the ability to see each action as it happens.

For this we set up a simple event listener/trigger system. We will trigger events as they happen and use `.on` to listen to them. Then we can log the output using `console.log` or make more advanced UIs.

```
const  ALL_EVENTS  = [
  'queued',
  'complete',
];

class Queue {	
	constructor() {
	 // ... other code
	 this.initializeEvents();
	}

	initializeEvents() {
	  this.eventListeners = ALL_EVENTS.reduce((listeners, event) => {
	    listeners[event] = [];
	    return  listeners;
	  }, {});
	}

	triggerEvent(event, promise) {
	  if (!this.eventListeners[event]) return;
	  this.eventListeners[event].forEach((cb) => {
	    cb(promise);
	  });
	}

	on(event, cb) {
	  if (event  ===  'all') {
	    ALL_EVENTS.forEach((e) => {
	      this.eventListeners[e].push((...args) =>  cb(e, ...args));
	    });
	  } else {
	    this.eventListeners[event].push(cb);
	  }
	  return  this;
	}

	add() {
	  // ... other code
	  this.queued.push(wrapperPromise);
	  
	  // ADD THIS LINE
	  this.triggerEvent('queued', wrapperPromise);
	}

	processNextItem() {
	  // ... other code
	  this.moveLists(promise, 'pending', 'complete');
	  
	  // ADD THIS LINE
	  this.triggerEvent('complete', promise);
	}
}

const queue = new Queue();

queue.on('all', (event, promise) => {
   console.log(event, promise);
});
```

### Adding wait time between request 
So we have our queue...but at this rate, if we have 1000 requests queued up, it will try to hit the server with 1000 requests at once. Not good...

Let's wait at least 1 second in between requests:

```
const WAIT_BETWEEN_REQUESTS = 1000;

wait (ms) {
  return new Promise(resolve  =>  setTimeout(() => resolve(), ms));
}

async processNextItem() {
   await this.wait(WAIT_BETWEEN_REQUESTS);
   // ...other code
}
```

Awesome. Now at least we will wait a second between bombarding the server

### Adding Concurrency 

Now let's add some concurrency support. Perhaps we only want 3 requests going on at one time.

For this we can wrapper all the code in `processNextItem` below our initial await with an `if` statement

```
const MAX_CONCURRENT = 3;

if (this.pending.length < MAX_CONCURRENT) {
  if (!this.queued.length) { return  false; }
  // ... OTHER CODE
  // this.moveLists(this.queued[0], 'queued', 'pending');
}

return Promise.race(this.pending);
```

### Having multiple queues at the same time
Now that we have a queue of 1 endpoint, we can replicate this for multiple endpoints. We'll create a hash map to store all of our queues.

```
const queues = {};

function createQueue(url) {
  queues[url] = new Queue();
}
```


###  Adding rate limiting support
This is one of the most important parts of API Scraping. Most APIs will give you an endpoint for checking how many requests we have left.

So we will set up a function `getRateLimits` to fetch every few seconds and find out the remaining limits on the endpoints. Then we use `blockQueue` 

```
// fetch every 10 seconds
const RATE_LIMIT_AUTO_FETCH_INTERVAL = 10000;

let rateFetchTimeout;
let rateLimits;

// has our list of queues from previous step
const queues;

const setRateLimitOnQueue = (url, reset) => {
  const queue = queues[url];
  if (!queue) return;
  
  const unblockIn  =  moment.unix(reset).diff(moment());
  queue.blockQueue(unblockIn);
};

async function getRateLimits() {
  const result = await fetch(RATE_LIMIT_ENDPOINT);
  
  // format limits so that it it in the format:
  
  const limits = {
     endpoint_name: ISO_DATE_WHEN_RESET,
     endpoint_name2: ISO_DATE_WHEN_RESET,
  }

  Object.entries(limits).forEach((ep, reset) => { 
    setRateLimitOnQueue(ep, reset);
  });
}

function initRateLimitsAutoFetch() {
  rateFetchTimeout = setInterval(async () => {
    rateLimits = await getRateLimits();
  }, RATE_LIMIT_AUTO_FETCH_INTERVAL);
}
```
 
###  Adding error handling
There are some situations that APIs throw as errors but actually need to be handled in other ways. We want to handle them by wrapping our API in a `try catch`

Two common scenarios we want to handle are a Not Found error, in which case we want to return a `null`. Another scenario may be when we are rate limited, in which case we want to call our previous rate limit function to wait a few seconds.

```
async request(method, url, params } = {}) {
  return this.queues[url].add(async () => {
    try {
      return (await  this.client[method](url, params)).data;
    } catch (e) {
      // not found
      if (e.code  ===  34) return null;
      
      // rate limited
      if (e.code  ===  88) {
        // waits 20 seconds so that the rate limit

        // auto request can find something
        this.setRateLimitOnQueue(url, moment().add(20 * 1000, 'milliseconds').unix());
        return;
      }
    
      throw  e;
    }
  });
}
```

#### Retrying
A lot of times, even when we handle Rate limiting correctly, the server may fail some of our requests. We want to have an effective retry strategy.

For this we have to slightly edit our `process` function. It will still run a `while` loop. It will be in charge of calling `processNextItem` which queues up the next function, which passes it off to `runFunction`.

`runFunction` waits for blocks to be cleared on the queue, then tries to run the function. If the function succeeds, then the item is moved to a `completed` state. Otherwise we move on to our retry logic.

If the function fails, we call `runFunction` again (recursion), while incrementing the `tryNumber`. If the `tryNumber` reaches the maximum tries (3), we move the item to failed state.

```
async runFunction(func, promise, tryNumber  =  0, { retry = true }) {
  if (this.waitBetweenRequests) {
    await wait(this.waitBetweenRequests);
  }

  if (this.blocked) {
    await this.block;
  }
  
  try {
    await func();
    this.moveLists(promise, 'pending', 'complete');

    this.triggerEvent('complete', promise);

  } catch (e) {
    if (this.retry) {
      if (tryNumber < 3) {
        const res = await this.runFunction(func, promise, tryNumber  +  1);

        if (res) return  true;
      }
    }
    
    this.moveLists(promise, 'pending', 'failed');

    this.triggerEvent('failed', promise);
  }
  
  return  true;
}

async processNextItem() {
  if (this.pending.length  <  this.maxConcurrent) {
    if (!this.queued.length) { return  false; }
    
    const  promise = this.queued[0];

    this.moveLists(promise, 'queued', 'pending');

    await  this.runFunction(this.queuedFuncs.shift(), promise);
    
    // process next
    return  true;
  }

  // wait for something to succeed or fail
  return  Promise.race(this.pending);
}

async process() {
  while (this.queuedFuncs.length) {
	await this.processNextItem();
  }

  return  Promise.all(this.pending);
}
```

### Improving our logging - making it easy to debug
It can be quite difficult to debug what's going on without logging, especially if the function is running for hours at a time. I've found that making a progress bar is the most effeective solution to check on what's going on.

#### Progress Bar
I wrote a progress bar that handles multiple queues with the structure we created. Instead of copy and pasting it here, you can look at the [github file](https://github.com/tolicodes/node-api-toolkit/blob/master/components/bar.js)

The key is hooking on the `on` function from the queues. Then we use `terminal-kit` to help us color in the screen according to our current progress and status. 
  
### Using chrome inspector to debug
I strongly recommend using the Chrome Debugger.

Instead of regularly running our application
```
node .`
```
we add an extra parameter 
```
node --inpect .
```

Then we navigate to: [chrome://inspect](chrome://inspect) in the Chrome browser, and are able to see our script in the list. 

This is my preferred method of debugging, because not only can we see a much more user friendly output, but we can execute commands in the node environment as needed.

### Adding pagination support
Most apis have some sort of pagination support for long lists. Generally this is done by passing a "cursor" for the next result, which is provider by the previous result

```
const paginate = async ({

results  = [],

cursor,

func,

params,

page  =  0,

maxPages  =  1,

}) => {

if (page  <  maxPages  &&  cursor) {

const {

data,

nextCursor,

} =  await  func({

params,

cursor,

});

  

const  newResults  =  await  paginate({

results: [

...results,

...data,

],

cursor:  nextCursor,

func,

params,

page:  page  +  1,

}) || [];

  

return [

...results,

...newResults,

];

}

  

return  results;

};

  
  

module.exports  =  paginate;
```

### Pausing/Resuming
All we have to do to pause/resume our queues is call `.block()` to pause, and then `.unblock()` to unpause. Our current strucure will support the rest.
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTQ1MTc0OTM4LC02MjU0NTk2MTQsLTEyND
Q1NDU4NTksLTExNDA0MjkwNDUsLTExODAwMzAxNDksOTM2Nzgx
MTk3LC0xMjA1NzI5ODkxLC0zMjE5Nzk5NjUsMzA4Njk3OTI5LC
0xMTgyNTU1NTA0LC0xMzIyMTcwMDY1XX0=
-->