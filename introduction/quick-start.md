# Quick Start

This page shows you how to start building your own application using Axon platform.
In order to run this quick start you will need Java 11 installed on your machine.

 1. Get everything you need by downloading [QuickStart archive](https://axoniq.io/download). It contains a simple demo application, designed to show various aspects of the platform.
 2. Unzip `AxonQuickStart.zip`
 3. Move to the extracted `cd axonquickstart-VERSION`
 4. Run the [Axon Server](axon-server.md): `$ java -jar AxonServer/axonserver-VERSION.jar`
 5. Axon Server web dashboard should be available here [`http://localhost:8024/`](http://localhost:8024/) : moving in the overview page you will see your running instance of the server.
 6. Run the demo application: `$ cd giftcard-demo && ./mvnw spring-boot:run`
 7. Demo application frontend should be available here [`http://localhost:8080/`](http://localhost:8080/)
 8. Going to Axon Server web dashboard overview [`http://localhost:8024/#overview`](http://localhost:8024/#overview) you will see your current demo application instance succesfully connected to the server.
 9. Explore the `README.md`
 10. [Axon Coding Tutorial #1: - The Structure of an Axon Application](https://youtu.be/tqn9p8Duy54)

Quick Start Demo app can run on Java 8 with a small change in the `pom.xml` file
 1. From the extracted folder, open `giftcard-demo\pom.xml` with your favourite editor
 2. Replace `<java.version>11</java.version>` occurrency with `<java.version>1.8</java.version>`
 2. Find the `maven-compiler-plugin` plugin block
 3. Replace `<release>${java.version}</release>` with `<source>${java.version}</source> <target>${java.version}</target>`
 4. Run the demo application: `$ cd giftcard-demo && ./mvnw spring-boot:run`
