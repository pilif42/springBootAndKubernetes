# Instructions I followed:
- https://spring.io/guides/gs/spring-boot-kubernetes/


# To build
./gradlew clean build


# To run
./gradlew bootRun


# To test
curl http://localhost:8080 -v -X GET
200 {"id":1,"content":"Hello, World!"}

curl http://localhost:8080/greeting/ -v -X GET
200 {"id":1,"content":"Hello, World!"}


# To containerize (Docker container) the Application which is a simple HelloWorld Spring Boot app:
- create a Dockerfile at the project root. Note the value given to APPJAR.
- install Docker locally. Verify it is up and running with: 
        - sudo docker --version
- build the container image (brossierp = my https://hub.docker.com/ ID, springBootAndKubernetes = image tag):
        - cd /home/philippe/code/springBootAndKubernetes
        - sudo docker build -t brossierp/spring-boot-and-kubernetes .
- run the container locally:
        - sudo docker run -p 8080:8080 brossierp/spring-boot-and-kubernetes     
- test it is OK. In another cmd window:
        - curl http://localhost:8080/greeting/ -v -X GET
- kill the container and push the image to Dockerhub:
        - sudo docker push brossierp/spring-boot-and-kubernetes
- verify the image can be seen in Dockerhub:
        - sign into https://hub.docker.com/ with brossierp
        - you should see an entry at https://hub.docker.com/repository/docker/brossierp/spring-boot-and-kubernetes
        
        
# Once the app has been containerized, it is time to deploy it to Kubernetes:
- prerequisite:
        - I had installed Minikube (a lightweight Kubernetes implementation that creates a VM on your local machine and 
        deploys a simple cluster containing only one node.) and started it following https://github.com/pilif42/cheatsheets/blob/master/Containerization/Kubernetes/minikube
- create a deployment: fully explained at https://kubernetes.io/docs/tutorials/kubernetes-basics/deploy-app/deploy-intro/
        - kubectl create deployment spring-boot-and-kubernetes --image=brossierp/spring-boot-and-kubernetes
                - deployment.apps/spring-boot-and-kubernetes created
- verify the deployment was successful:
        - kubectl get deployments
                - NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
                - spring-boot-and-kubernetes   0/1     1            0           13s
        - kubectl get pods
                - NAME                                          READY   STATUS    RESTARTS   AGE
                - spring-boot-and-kubernetes-6c9b54db54-x9zn4   1/1     Running   0          25s
- expose the deployment as a service:
        - why would you do this? -> https://kubernetes.io/docs/concepts/services-networking/service/#motivation
        - different options exist:
                - option 1 = kubectl expose deployment spring-boot-and-kubernetes --type=NodePort --port=8080
                        - service/spring-boot-and-kubernetes exposed
                - option 2 (our preferred option) = kubectl expose deployment spring-boot-and-kubernetes --type=LoadBalancer --port=8080
                        - service/spring-boot-and-kubernetes exposed
                        - --type=LoadBalancer flag indicates that you want to expose your Service outside of the cluster.
                                - On cloud providers that support load balancers, an external IP address would be provisioned to access the Service. 
                                - On Minikube, the LoadBalancer type makes the Service accessible through the minikube service command (see option 2 below).
- verify the service creation was successful:
        - kubectl get services
- get the url to access the service/app:
        - minikube service spring-boot-and-kubernetes --url
                - http://192.168.39.206:31165
- to test your service:
        - option 1 = curl http://192.168.39.206:31165/greeting/ -v -X GET
                - 200 {"id":2,"content":"Hello, World!"}
        - option 2 = minikube service spring-boot-and-kubernetes
                - to use if option 2 was selected when creating the service.
                - it opens a browser and does a GET to http://192.168.39.206:31165/
- to clean up:
        - Note that if you stop/start Minikube, the deployment remains active.
        - kubectl delete service spring-boot-and-kubernetes 
        - kubectl delete deployment spring-boot-and-kubernetes
        
        
# Same as previous section but .yaml files are created so they can simply be re-applied/re-played
- First run:
        cd /home/philippe/code/springBootAndKubernetes/kubernetes
        kubectl create deployment spring-boot-and-kubernetes --image=brossierp/spring-boot-and-kubernetes --dry-run=client -o=yaml > deployment.yaml
        kubectl apply -f deployment.yaml
        kubectl expose deployment spring-boot-and-kubernetes --type=LoadBalancer --port=8080 --dry-run=client -o=yaml > expose.yaml
        kubectl apply -f expose.yaml
- Verify:
        - get the url with: minikube service spring-boot-and-kubernetes --url
        - curl http://192.168.39.206:xxx/greeting/ -v -X GET
- Clean up:
        - kubectl delete service spring-boot-and-kubernetes 
        - kubectl delete deployment spring-boot-and-kubernetes
- Subsequent runs:
        - verify that nothing is deployed yet with: kubectl get pods
        - cd /home/philippe/code/springBootAndKubernetes/kubernetes
        - kubectl apply -f deployment.yaml
        - kubectl apply -f expose.yaml
        
        
# Useful kubectl commands
- To find out details about the node minikube (in a prod cluster, you would have several nodes):
        - kubectl describe node minikube
                - in particular: 
                        - InternalIP = 192.168.39.206 -> the IP address of the node that is routable only within the cluster.
                        - no ExternalIP listed.
        
        
# TODOs
Read the Kubernetes docs:
        - https://kubernetes.io/docs/concepts/services-networking/service/
