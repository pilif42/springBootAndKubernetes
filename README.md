# Instructions I followed:
- https://spring.io/guides/gs/spring-boot-kubernetes/


# To build
./gradlew clean build


# To run
./gradlew bootRun


# To test
curl http://localhost:8080/greeting/ -v -X GET
200 {"id":1,"content":"Hello, World!"}


# To containerize the Application which is a simple HelloWorld Spring Boot app:
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
        - I had installed Minikube locally and started it following https://github.com/pilif42/cheatsheets/blob/master/Containerization/Kubernetes/minikube
- kubectl create deployment spring-boot-and-kubernetes --image=brossierp/spring-boot-and-kubernetes
        - deployment.apps/spring-boot-and-kubernetes created
- kubectl get deployments
        - NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
        - spring-boot-and-kubernetes   0/1     1            0           13s
- kubectl get pods
        - NAME                                          READY   STATUS    RESTARTS   AGE
        - spring-boot-and-kubernetes-6c9b54db54-x9zn4   1/1     Running   0          25s
- kubectl expose deployment spring-boot-and-kubernetes --type=NodePort --port=8080
        - service/spring-boot-and-kubernetes exposed
- minikube service spring-boot-and-kubernetes --url
        - http://192.168.39.206:31404
- test with curl http://192.168.39.206:31404/greeting/ -v -X GET
        - 200 {"id":2,"content":"Hello, World!"}
- to clean up:
        - Note that if you stop/start Minikube, the deployment remains active.
        - kubectl delete service spring-boot-and-kubernetes 
        - kubectl delete deployment spring-boot-and-kubernetes
        
        
# TODOs
Try to follow the section 'Deploy the Application to Kubernetes' at https://spring.io/guides/gs/spring-boot-kubernetes/ 
to replicate the above successful deployment.

kubectl create deployment spring-boot-and-kubernetes --image=brossierp/spring-boot-and-kubernetes --dry-run=client -o=yaml > deployment.yaml
kubectl create service clusterip spring-boot-and-kubernetes --tcp=8080:8080 --dry-run=client -o=yaml >> deployment.yaml
kubectl apply -f deployment.yaml

- apply the created deployment file:
        - kubectl apply -f deployment.yaml
- check that the application is running:
            - kubectl get all
                NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
                service/kubernetes                   ClusterIP   xxx             <none>        443/TCP    21d
                service/spring-boot-and-kubernetes   ClusterIP   xxx             <none>        8080/TCP   53s
- create an SSH tunnel to be able to connect to the application, which you have exposed as a Service in Kubernetes:
        - kubectl port-forward svc/spring-boot-and-kubernetes 8080:8080
        
