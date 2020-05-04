# Quick Start

This page shows you how to start building your own application using Axon platform.
In order to run this quick start you will need Java 11 installed on your machine.

1. Get everything you need by downloading [QuickStart archive](https://axoniq.io/download). It contains a simple demo application, designed to show various aspects of the platform.
1. Unzip `AxonQuickStart.zip`
1. Move to the extracted `cd axonquickstart-VERSION`
1. Run the [Axon Server](axon-server.md): `$ java -jar AxonServer/axonserver-VERSION.jar`
1. Axon Server web dashboard should be available here [`http://localhost:8024/`](http://localhost:8024/) : moving in the overview page you will see your running instance of the server.
1. Run the demo application: `$ cd giftcard-demo && ./mvnw spring-boot:run`
1. Demo application frontend should be available here [`http://localhost:8080/`](http://localhost:8080/)
1. Going to Axon Server web dashboard overview [`http://localhost:8024/#overview`](http://localhost:8024/#overview) you will see your current demo application instance succesfully connected to the server.
1. [Axon Coding Tutorial #1: - The Structure of an Axon Application](https://youtu.be/tqn9p8Duy54)

Quick Start Demo app can run on Java 8 with a small change in the `pom.xml` file
1. From the extracted folder, open `giftcard-demo\pom.xml` with your favourite editor
1. Replace `<java.version>11</java.version>` occurrency with `<java.version>1.8</java.version>`
1. Find the `maven-compiler-plugin` plugin block
1. Replace `<release>${java.version}</release>` with `<source>${java.version}</source> <target>${java.version}</target>`
1. Run the demo application: `$ cd giftcard-demo && ./mvnw spring-boot:run`
