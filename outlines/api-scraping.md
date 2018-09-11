# Abstract
This will be a complete guide to dealing with API Scraping. I explain how I came up with my [api-tookit](https://github.com/tolicodes/node-api-toolkit) and the [twitter-toolkit](https://github.com/tolicodes/twitter-toolkit) based on it. 

# Outline
## Intro
I’ve done a few projects now that involve API scraping of some sort - Twitter, AWS, Google, Medium, JIRA, you name it. It’s a fairly common task when you’re a freelance developer. Throughout these implementations I’ve used a few libraries, including bottleneck, promise-queue, and just rolling my own. However none of the existing solutions covered every aspect of scraping. So, I created my own solution, api-toolkit, which I believe solves 90% of the challenges you will encounter in scraping your own APIs. 

I feel it is important to understand the Fundementals behind some of these concepts, so I decided to write this article for CodeMentor. 

API scraping has many challenges, but we will be focusing on what I believe are the major ones in this article. 

### Rate Limiting
Just about API you will be hitting (public or private) will come wit

### Error Handling

### Pagination
 
### Concurrency

### Logging and Debugging

## The Basics
### Key/Secret Management
Storing in .env file vs environmental variables
  
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
eyJoaXN0b3J5IjpbNDM2OTMwNzk4LC02MDc2MDQ0NjYsLTE0Nz
Y0NDQ3ODEsNTkwNjIyMzE4LDI5ODMzODk0NCwtMTQwMjQ0MDc5
NV19
-->