# Kubernetes Nginx Ingress Controller on Minikube 
## Minikube provides a very easy way to standup a local, single node kubernetes development cluster. In this example, we will utilize Minikube and deploy the nginx ingress controller to manage new ingress objects that will provide virtual host routing, ssl termination, and session affinity/stickiness. We will build this application from the ground up, and slowly add new functionality via different kubernetes objects to illustrate their purpose.  

![request splitting architecture diagram](ambassador_request_splitting_arch.png)

## Prerequisites:  
Install minikube before proceeding.  

Offical Minikube install reference:  
https://minikube.sigs.k8s.io/docs/start/ 

## Build Procedure:
1. Start minikube if not already running:  
  ```shell
  minikube start  
  ```
  
2. Optional: Create alias for kubectl to reduce keystrokes. Example for debian based linux distribution:  
  ```shell  
  echo 'alias kubectl="minikube kubectl --"' >> ~/.bashrc   
  ```
  
3. Install the nginx ingress controller in minikube by enabling an addon:  
  ```shell  
  minikube addons enable ingress  
  ```

4. Verify the nginx ingress controller was installed and it's pod is running:  
  ```shell
  kubectl -n ingress-nginx get pods
  minikube service list
  ```

5. Create the namespace for the echo-server application:  
  ```shell
  kubectl apply -f echo-server-namespace.yaml
  kubectl get namespace
  ```
  
6. Create a deployment for this application, which will create 3 echo-server pods, a replicaset, and the deployment object:   
  ```shell
  kubectl apply -f echo-server-deployment.yaml
  kubectl get all -n echo-server
  ```

7. Create a service of type NodePort to expose this application externally from the minikube cluster:  
  ```shell
  kubectl apply -f echo-server-service-nodeport.yaml
  kubectl get service -n echo-server
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


