# To build
./gradlew clean build

# To run
./gradlew bootRun

# To test
curl http://localhost:8080/greeting/ -v -X GET
200 {"id":1,"content":"Hello, World!"}
