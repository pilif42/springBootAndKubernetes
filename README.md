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
- verify the deployment was successful:
        - kubectl get deployments
                - NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
                - spring-boot-and-kubernetes   0/1     1            0           13s
        - kubectl get pods
                - NAME                                          READY   STATUS    RESTARTS   AGE
                - spring-boot-and-kubernetes-6c9b54db54-x9zn4   1/1     Running   0          25s
- expose the deployment:
        - kubectl expose deployment spring-boot-and-kubernetes --type=NodePort --port=8080
                - service/spring-boot-and-kubernetes exposed
- get the url to access the service/app:
        - minikube service spring-boot-and-kubernetes --url
                - http://192.168.39.206:31404
- test with curl http://192.168.39.206:31404/greeting/ -v -X GET
        - 200 {"id":2,"content":"Hello, World!"}
- to clean up:
        - Note that if you stop/start Minikube, the deployment remains active.
        - kubectl delete service spring-boot-and-kubernetes 
        - kubectl delete deployment spring-boot-and-kubernetes
        
        
# Same as previous section but .yaml files are created so they can simply be re-applied/re-played
- First run:
        cd /home/philippe/code/springBootAndKubernetes/kubernetes
        kubectl create deployment spring-boot-and-kubernetes --image=brossierp/spring-boot-and-kubernetes --dry-run=client -o=yaml > deployment.yaml
        kubectl apply -f deployment.yaml
        kubectl expose deployment spring-boot-and-kubernetes --type=NodePort --port=8080 --dry-run=client -o=yaml > expose.yaml
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
        
        
# TODOs
Read the Kubernetes docs:
      - https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/
      - https://kubernetes.io/docs/concepts/services-networking/service/
Script the above set of cmds        
