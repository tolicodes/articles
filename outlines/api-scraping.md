# Abstract
This will be a complete guide to dealing with API Scraping. I explain how I came up with my [api-tookit](https://github.com/tolicodes/node-api-toolkit) and the [twitter-toolkit](https://github.com/tolicodes/twitter-toolkit) based on it. 

# Outline
## Intro
Why API scraping is not as easy as it seems

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
- API Scraping Concepts
  - Building a simple queue 
  - Adding logging
  - Adding concurrency 
  - Adding rate limiting support
  - Adding error handling
  - Improving our logging - making it easy to debug
    - Progress Bar
    - Status Reporting
    - Using chrome inspector to debug
  - Adding pagination support
  - Adding support for batch operations
- Advanced Topics
  - Scraping Cluster
  - Multiple Keys
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTQ2ODc3NTM1MSwyOTgzMzg5NDQsLTE0MD
I0NDA3OTVdfQ==
-->