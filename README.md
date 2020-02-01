# Instructions I followed
https://spring.io/guides/gs/spring-boot-kubernetes/


# To build
./gradlew clean build


# To run
./gradlew bootRun


# To test
curl http://localhost:8080/greeting/ -v -X GET
200 {"id":1,"content":"Hello, World!"}


# Kubernetes sandbox
TODO Que choisir:
https://aws.amazon.com/eks/?nc2=type_a
       - start at https://eksworkshop.com/010_introduction/basics/what_is_k8s/
https://console.cloud.google.com/projectselector2/kubernetes/


# Install and Set Up kubectl
I followed https://kubernetes.io/docs/tasks/tools/install-kubectl/.
TODO Need to have chosen a Kube cluster before as its version will impact which version of kubectl we need.
