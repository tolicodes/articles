# Abstract
This will be a complete guide to dealing with API Scraping. I explain how I came up with my [api-tookit](https://github.com/tolicodes/node-api-toolkit) and the [twitter-toolkit](https://github.com/tolicodes/twitter-toolkit) based on it. 

# Outline
## Intro
I’ve done a few projects now that involve API scraping of some sort - Twitter, AWS, Google, Medium, JIRA, you name it. It’s a fairly common task when you’re a freelance developer. Throughout these implementations I’ve used a few libraries, including bottleneck, promise-queue, and just rolling my own 

### Rate Limiting

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
eyJoaXN0b3J5IjpbMTE4NjU3NjA2LC02MDc2MDQ0NjYsLTE0Nz
Y0NDQ3ODEsNTkwNjIyMzE4LDI5ODMzODk0NCwtMTQwMjQ0MDc5
NV19
-->