# Abstract
This will be a complete guide to dealing with API Scraping. I explain how I came up with my [api-tookit](https://github.com/tolicodes/node-api-toolkit) and the [twitter-toolkit](https://github.com/tolicodes/twitter-toolkit) based on it. 

# Outline
## Intro
I’ve done a few projects now that involve API scraping of some sort - Twitter, AWS, Google, Medium, JIRA, you name it. It’s a fairly common task when you’re a freelance developer. Throughout these implementations I’ve used a few libraries, including bottleneck, promise-queue, and just rolling my own. However none of the existing solutions covered every aspect of scraping. So, I created my own solution, api-toolkit, which I believe solves 90% of the challenges you will encounter in scraping your own APIs. 

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

#### enviornmental variables
Environmental variables are easily manageable in most environments that run Docker Containers (ex: AWS). They are one of the more secure and simple way to go.

Esseniallly, all you have to do is manually set variables in your shell environment. Or in your container provider's config, such as [AWS ECS](https://stackoverflow.com/questions/43330278/how-to-provide-environment-variables-to-aws-ecs-task-definition) (to persist the variable across restarts).

To get this working on your local machine,

type in your bash environment:
```
export API_KEY=sn89ds2ju93sdnljos
```

or add that line to `~/.bashrc`

And then in your node file ic

  
### Libraries vs Roll Your Own 
 Using existing libraries vs using fetch vs axios/request
### Setting up our example app - Twitter scraper

## API Scraping Concepts

###  Building a simple queue
 
### Adding logging

### Adding concurrency 

### Having multiple queues at the same time

###  Adding rate limiting support
 
###  Adding error handling

#### Retrying

#### Exponential Backoff
 
### Request Dependencies

### Improving our logging - making it easy to debug
 
#### Progress Bar
  
#### Status Reporting
  
### Using chrome inspector to debug

### Adding pagination support

### Adding support for batch operations

### Pausing/Resuming

### Streaming Results to a file

## Advanced Topics
### Scraping Cluster

### Multiple Keys
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTg3NTg0MTI5LDE4OTU2NDMyMzAsLTYwNz
YwNDQ2NiwtMTQ3NjQ0NDc4MSw1OTA2MjIzMTgsMjk4MzM4OTQ0
LC0xNDAyNDQwNzk1XX0=
-->