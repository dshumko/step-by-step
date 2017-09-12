# step-by-step

An example of a service, which "evolves" step by step.
[Видео](https://www.youtube.com/watch?v=0ndWw1udpsA)

## Step 01. Preparation

It might be helpful to set up git pre-commit hooks before writing the code.
For example, you can use [pre-commit by Yelp](http://pre-commit.com) to manage this kind of hooks.

See `.pre-commit-config.yaml`.

## Step 02. Add the simplest web-server

The simplest way to make a web-server: register a handler function and listen on some TCP network address:
 
    http.HandleFunc("/", home)
    http.ListenAndServe(":8000", nil)

## Step 03. Add tests

`TestHandler` function (see `handlers_test.go`) is the simplest way to test our web-server: check if the handler
returns expected result. Use `go test -v` command to run test.

## Step 04. Add router

We need to be able to process diffrenet HTTP methods and URLs. 
So, let's use [takama/router](https://github.com/takama/router) instead of standard handler registrar.

You can try any other router. 
For example, one of the most popular solutions is [julienschmidt/httprouter](https://github.com/julienschmidt/httprouter).

Don't forget to check if tests are still green (`go test -v`) and change them if it is necessary.

## Step 05. Add logger

Add logger to log requests. [Sirupsen/logrus](https://github.com/Sirupsen/logrus) is my favourite!

## Step 06. Add dependency management

For this workshop we choose [glide](https://glide.sh). 
After installation of glide run `glide init` to initialize dependency management and `glide install` to install
dependencies. Feel free to use `glide up` to check for dependencies updates.

In this example we ignore `vendor` directory and `glide.lock` file. But in real "production" cases it's
better to commit `vendor` or at least `glide.lock` to be able to reach dependencies in an emergency situation.

## Step 07. Get service configuration from ENV

It's very important to store service configuration in the environment.
So, ENV variables is, probably, the simplest way to deal with config.

## Step 08. Run service in Docker

We believe what Dockerfile of any application must be as tiny as it possible.
For simple services like this one, we can use `FROM scratch` construction and provide
very tiny Docker image.

Let's compile our service with special tags: 

    env CGO_ENABLED=0 GOOS=linux go build

and make a container with the binary:

    docker build -t step-by-step -f ./Dockerfile .
    
Run service on Docker:

    docker run -p 8000:8000 step-by-step 

## Step 09. Add information about current build

It might be very helpful to know, what the current version of build is.

## Step 10. Liveness and readiness probes (healthchecks)

Liveness and readiness probes will help us to manage containers.

Liveness probe is more like typical healthcheck.
If liveness probe fails, Kubernetes will destroy and deploy again the failed container.

Readiness probe shows that application is ready to receive connections.
For example, during the initialization readiness probe may show what
the instance of application is not ready for traffic yet, so Kubernetes will not send
requests to this instance.

## Step 11. Graceful shutdown

