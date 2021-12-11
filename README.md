# CampGrounds App
<p align="center">
  <img width="1000" height="500" src="https://github.com/Hemendra05/CampgroundsApp/blob/master/screenshots/YelpCamp%20%E2%80%94%20Mozilla%20Firefox%2011-12-2021%2015_21_51.png">
</p>

Yelp Camp is an application for a fictional start up that allows you to add campgrounds for other users to comment and rate, as view campgrounds added by other users. Not much more I can say about it.

Here are some basic functionality of the app:-

Authentication

    Users can sign up or login using username and password.
    User can not submit campgrounds if they are not logged in.

Authorization

    User can only modify campgrounds created by them.

User Profile

    Every registered user has profile where all his submitted campgrounds are shown.
    
Basic Functionality

    Add Name, Image and Description to the campground.
    Create, Update, Delete the Campground.
    Add comments to campgrounds.
    Flash Important messages to warn or greet the users.
    Responsive Web design.
    
## Creating Dockerfile

Dockerfile is a concept of docker, that is a great containerization tool itself, so Dockerfile is a way to create docker image. This docker image is build with all the configuration of required software and dependencies of the application.

```Dockerfile
FROM hemendra05/nodejs

RUN mkdir /campGroundApp

WORKDIR /campGroundApp

RUN mkdir ./cloudinary

RUN mkdir ./controller

RUN mkdir ./public

RUN mkdir ./seed

RUN mkdir ./schemaValidator

RUN mkdir ./models

RUN mkdir ./routes

RUN mkdir ./templates

RUN mkdir ./utils

COPY cloudinary/ ./cloudinary

COPY controller/ ./controller

COPY public/ ./public

COPY seed/ ./seed

COPY schemaValidator/ ./schemaValidator

COPY models/ ./models

COPY routes/ ./routes

COPY templates/ ./templates

COPY utils/ ./utils

COPY app.js .

COPY middleware.js .

COPY package.json .

COPY .env .

EXPOSE 3000

RUN npm install

CMD node app.js

```

This is the Dockerfile we are going to use to create the Docker image.

## Creating ECR repository on AWS

Now we are going to create a Elastic Container Registory, where we'll upload our Docker image after building it.
Now we will retrieve an authentication token and authenticate Docker client to our registry using the AWS CLI by following command:
```
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 237488248364.dkr.ecr.us-east-1.amazonaws.com
```

Now its time to build our Docker image with the Dockerfile that we create earlier. We use following command to build the Docker image:
```
docker build -t yelp-camp .
```

After the build completes, we'll tag our image so we can push the image to this repository:

```
docker tag yelp-camp:latest *************.dkr.ecr.us-east-1.amazonaws.com/yelp-camp:latest
```

After doing all the things, we are good to go and run the following command to push this image to our newly created AWS repository:
```
docker push **************.dkr.ecr.us-east-1.amazonaws.com/yelp-camp:latest
```

Now after the image pushed successfully, we'll cross check it by going to the AWS Console. There we'll see that there is a image in that repository with the image tag 'latest'.
<p align="center">
<img width="1000" height="500" src="https://github.com/Hemendra05/CampgroundsApp/blob/master/screenshots/Search%20results%20-%20hemendrachaudhary2052000%40gmail.com%20-%20Gmail%20%E2%80%94%20Mozilla%20Firefox%2011-12-2021%2015_53_58.png">
</p>

## Creating Task in ECS

Now when we created docker image and push it to AWS ECR successfully, its time to create cluster in Elastic Container Service. First we create a Task Defination in the ECS by fullfilling all requirements.
<p align="center">
<img width="1000" height="500" src="https://github.com/Hemendra05/CampgroundsApp/blob/master/screenshots/Amazon%20ECS%20%E2%80%94%20Mozilla%20Firefox%20(Private%20Browsing)%2011-12-2021%2014_03_24.png">
</p>

Now we'll add the container to this Task Defination. There we'll give the container name and the docker image that we created earlier. We'll use only that image. Then we'll map the port to conatiner that is 3000.
<p align="center">
<img width="1000" height="500" src="https://github.com/Hemendra05/CampgroundsApp/blob/master/screenshots/Amazon%20ECS%20%E2%80%94%20Mozilla%20Firefox%20(Private%20Browsing)%2011-12-2021%2014_04_39.jpg">
</p>

Now after creating Task Defination, we'll create the ECS Cluster. It simple to create the cluster, few steps and cluster is running.
<p align="center">
<img width="1000" height="500" src="https://github.com/Hemendra05/CampgroundsApp/blob/master/screenshots/Amazon%20ECS%20%E2%80%94%20Mozilla%20Firefox%20(Private%20Browsing)%2011-12-2021%2014_05_41.png">
</p>

We used Fargate to create the Task Defination and Cluster. So now we'll run a Task in Fargate with that Task Defination.
<p align="center">
<img width="1000" height="500" src="https://github.com/Hemendra05/CampgroundsApp/blob/master/screenshots/Amazon%20ECS%20%E2%80%94%20Mozilla%20Firefox%20(Private%20Browsing)%2011-12-2021%2014_06_49.png">
</p>

Now after sometime task is running and in healthy state. We can check some details like public IP and public DNS, security groups etc.

## Creating Lambda Function

Now our ECS cluster is up and running, we'll create a lambda function that will start the app. We add an HTTP API here as trigger that trigger the function and our app will start and show the output from the ECS Task. For this we have to write some code in fucntion in Node Js.

Let’s have a look at our Lambda function:<br />
From ./lambda/index.js:
```javascript
'use strict';

var helper = require('./lib/helper');

/**
 * Handles GET requests.
 */
module.exports.handler = (event, context, callback) => {
    callback = (typeof callback === 'function') ? callback : function() {};

    var response = helper.assembleResponse();

    callback(null, response);
};
```

From ./lambda/lib/helper.js:
```javascript
const HTTP_RESPONSE = 301;
const NEW_DOMAIN = "http://ec2-44-192-77-135.compute-1.amazonaws.com:3000/#";


module.exports.assembleResponse = function () {
    return {
        statusCode: HTTP_RESPONSE,
        headers: {
            "Location": NEW_DOMAIN
        },
        body: null
    }
}
```
