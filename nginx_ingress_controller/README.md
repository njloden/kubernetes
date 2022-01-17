# Kubernetes Nginx Ingress Controller on Minikube 
## Commonly used to broker requests between an application and other resources or the end users. In this example, we will apply this design pattern to implement http request splitting. This can be very useful if you wish to only roll out a new build to a fraction of your user population. Here, we deploy two simple flask web applications, one for prod and one for the new experimental version. Then we spin up an nginx proxy container to act as our ambassador/broker, and split the requests between each web application. 90% of requests should reach the prod web app, while only 10% should hit the new experimental version. This is ultimately handled by the weightdirective in nginx, with a value of 9 set for the production web application. 

![request splitting architecture diagram](ambassador_request_splitting_arch.png)

## Prerequisites:  
Install the docker engine before proceeding.  

Offical Docker install reference:  
https://docs.docker.com/engine/install  

## Build Procedure:
1. Build docker image for flask production web application:  
  ```shell
  sudo docker build --tag web-app-prod:latest ./app_prod  
  ```
  
2. Run the newly built image:  
  ```shell  
  sudo docker run -d -p 5001:5001 --name web-app-prod web-app-prod   
  ```
  
3. Ensure container is running and send a request to the app to make sure it is responding to requests:  
  ```shell  
  sudo docker ps  
  curl http://localhost:5001  
  ```

4. Build docker image for flask experimental web application:  
  ```shell
  sudo docker build --tag web-app-experiment:latest ./app_experiment
  ```

5. Run the newly built image:  
  ```shell
  sudo docker run -d -p 5002:5002 --name web-app-experiment web-app-experiment   
  ```
  
6. Ensure container is running and send a request to the app to make sure it is responding to requests:   
  ```shell
  sudo docker ps  
  curl http://localhost:5002  
  ```

7. Build docker image for nginx reverse proxy which will act as the ambassador and split requests:  
  ```shell
  sudo docker build --tag nginx-proxy:latest ./proxy 
  ```
  
8. Run the newly built image, and link to the other containers so nginx knows where to proxy requests:  
  ```shell
  sudo docker run -d -p 8080:8080 --link web-app-prod:web-app-prod --link web-app-experiment:web-app-experiment --name nginx-proxy nginx-proxy
  ```
  
9. Ensure container is running and send a request to the app to make sure it is responding to requests:  
  ```shell
  sudo docker ps
  curl http://localhost:8080
  ```
  
10. Nginx is configured to send 90% of requests to the prod app, and 10% to the experimental version. Send 10 requests to the nginx reverse proxy/ambassador, and ensure at least one request is being routed to the experimental web application:
  ```shell
  for i in {1..10}; do
    curl http://localhost:8080 && echo ""
  done 
  ```


