# Abstract
This will be a complete guide to dealing with API Scraping. I explain how I came up with my [api-tookit](https://github.com/tolicodes/node-api-toolkit) and the [twitter-toolkit](https://github.com/tolicodes/twitter-toolkit) based on it. 

# Outline
## Intro

### Rate Limiting

### Error Handling

### Pagination

### Concurrency

### Logging and Debugging

## The Basics
### Key/Secret Management

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
4. 

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
eyJoaXN0b3J5IjpbLTQ1OTA2NTIzNywxMTkwMjU2MDc0LDE4OT
U2NDMyMzAsLTYwNzYwNDQ2NiwtMTQ3NjQ0NDc4MSw1OTA2MjIz
MTgsMjk4MzM4OTQ0LC0xNDAyNDQwNzk1XX0=
-->