# Quick Start

Axon provides a _**Quick Start Toolkit**_ to familarize yourself with the setup required for Axon Framework and Axon Server SE \(Standard Edition\).

The only pre-requisite to run the Quick Start is to have a Java 8+ JRE in your system.

## Quick Start Toolkit Download

The Quick Start Toolkit package is available for download at the following location -&gt; [https://axoniq.io/download](https://axoniq.io/download).

This package contains,

* Axon Framework Binaries
* Axon Server Standard Edition
* Gift card sample application - Demo application designed to show various aspects of the platform
* Getting started guide

## Running the Quick Start \(Java 9+\)

1. Unzip `AxonQuickStart-VERSION.zip`
2. Run the [Axon Server](../axon-server-introduction.md): `$ java -jar AxonServer/axonserver-VERSION.jar`
3. Axon Server web dashboard should be available here [`http://localhost:8024/`](http://localhost:8024/)
4. Run the demo application: `$ cd giftcard-demo && ./mvnw spring-boot:run`
5. Demo application should be available here [`http://localhost:8080/`](http://localhost:8080/)
6. Explore the `README.md`

## Running the Quick Start on Java 8

The Quick Start Demo app can also run on Java 8 with a small change in the `pom.xml` file:

1. From the extracted folder, open `giftcard-demo\pom.xml` with your favourite editor
2. Replace `<java.version>11</java.version>` occurrency with `<java.version>1.8</java.version>`
3. Find the `maven-compiler-plugin` plugin block
4. Replace `<release>${java.version}</release>` with `<source>${java.version}</source> <target>${java.version}</target>`
5. Run the demo application: `$ cd giftcard-demo && ./mvnw spring-boot:run`

## Resources

### Axon Framework - Video Tutorials

The following 5-part video tutorial offers a quick start guided path to understand Axon Framework.

| Tutorial Name | Purpose |
| :--- | :--- |
| [Part - 1](https://www.youtube.com/watch?v=tqn9p8Duy54) | Structure of an Axon Framework Application |
| [Part - 2](https://www.youtube.com/watch?v=vnCxjWZrrk0) | Core API development |
| [Part - 3](https://www.youtube.com/watch?v=7oy4w5THFEU) | Command Model of the Application |
| [Part - 4](https://www.youtube.com/watch?v=jS1vfc5EohM&t=297s) | Query Model of the Application |
| [Part - 5](https://www.youtube.com/watch?v=lxonQnu1txQ&t=413s) | Connecting the UI |

### Axon Server

This blog is a good starting point to get familiarized with Axon Server - [https://axoniq.io/blog-overview/running-axon-server\#0](https://axoniq.io/blog-overview/running-axon-server#0)

### Fast Lane Training

AxonIQ provides a free to view [fast lane online training](https://lp.axoniq.io/fast-lane-axon-framework-online-training) which will bring you up to speed on topics such as DDD, CQRS, Event Sourcing and other concepts related to the Axon stack.

### Full Training

AxonIQ provides a [fully online Axon training program](https://axoniq.io/support-overview/axon-training) which provides a more hands-on approach complete with instructor-led content and labs. This program is intended to deepen your knowledge concerning the concepts of event-driven microservices, DDD, CQRS and how Axon supports this journey.

