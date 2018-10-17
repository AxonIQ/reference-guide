# Introduction

Axon Framework is a lightweight Java framework that helps developers build scalable and extensible applications by addressing these concerns directly in the architecture. This reference guide explains what the Axon Framework is, how it can help you and how you can use it.

If you want to know more about Axon and its background, continue reading in [Axon Framework Background](introduction.md#axon-framework-background). If you're eager to get started building your own application using Axon, go quickly to [Getting Started](getting-started.md). If you're interested in helping out building the Axon Framework, [Contributing](introduction.md#contributing-to-axon-framework) will contain the information you require.

## Axon Framework Background

### A brief history

The demands on software projects increase rapidly as time progresses. Companies want their \(web\)applications to evolve together with their business. That means that not only projects and code bases become more complex, it also means that functionality is constantly added, changed and \(unfortunately not enough\) removed. It can be frustrating to find out that a seemingly easy-to-implement feature can require development teams to take apart an entire application. Furthermore, today's web applications target the audience of potentially billions of people, making scalability an indisputable requirement.

Although there are many applications and frameworks around that deal with scalability issues, such as GigaSpaces and Terracotta, they share one fundamental flaw. These stacks try to solve the scalability issues while letting developers develop applications using the layered architecture they are used to. In some cases, they even prevent or severely limit the use of a real domain model, forcing all domain logic into services. Although that is faster to start building an application, eventually this approach will cause complexity to increase and development to slow down.

The **C**ommand **Q**uery **R**esponsibility **S**egregation \(CQRS\) pattern addresses these issues by drastically changing the way applications are architected. Instead of separating logic into separate layers, logic is separated based on whether it is changing an application's state or querying it. That means that executing commands \(actions that potentially change an application's state\) are executed by different components than those that query for the application's state. The most important reason for this separation is the fact that there are different technical and non-technical requirements for each of them. When commands are executed, the query components are \(a\)synchronously updated using events. This mechanism of updates through events, is what makes this architecture so extensible, scalable and ultimately more maintainable.

> **Note**
>
> A full explanation of CQRS is not within the scope of this document. If you would like to have more background information about CQRS, visit the [AxonIQ ](https://axoniq.io/resources/concepts)website.

Since CQRS is fundamentally different than the layered-architecture, it is not uncommon for developers to walk into a few traps while trying to find their way around this architecture. That's why the Axon Framework was conceived: to help developers implement CQRS applications while focusing on the business logic.

### What is Axon?

Axon Framework helps build scalable, extensible and maintainable applications by supporting developers apply the Command Query Responsibility Segregation \(CQRS\) architectural pattern. It does so by providing implementations of the most important building blocks, such as aggregates, repositories and event buses \(the dispatching mechanism for events\). Furthermore, Axon provides annotation support, which allows you to build aggregates and event listeners without tying your code to Axon specific logic. This allows you to focus on your business logic, instead of the plumbing, and helps you to make your code easier to test in isolation.

Axon does not, in any way, try to hide the CQRS architecture or any of its components from developers. Therefore, depending on team size, it is still advisable to have one or more developers with a thorough understanding of CQRS on each team. However, Axon does help when it comes to guaranteeing delivering events to the right event listeners and processing them concurrently and in the correct order. These multi-threading concerns are typically hard to deal with, leading to hard-to-trace bugs and sometimes complete application failure. When you have a tight deadline, you probably don't even want to care about these concerns. Axon's code is thoroughly tested to prevent these types of bugs.

The Axon Framework consists of a number of modules that provide the tools and components to build a scalable infrastructure. The Axon Core module provides the basic APIs for the different components, and simple implementations that provide solutions for single-JVM applications. The other modules address scalability or high-performance issues, by providing specialized building blocks.

### When to use Axon?

There is a wide variety of applications that do benefit from Axon. Of course, not every application will benefit from Axon. Simple CRUD \(Create, Read, Update, Delete\) applications which are not expected to scale will probably not benefit from CQRS or Axon.

Applications that will likely benefit from CQRS and Axon are those that show one or more of the following characteristics:

* The application is likely to be extended with new functionality during a long period of time - For example, an online store might start off with a system that tracks progress of orders. At a later stage, this could be extended with Inventory information, to make sure stocks are updated when items are sold. Even later, accounting can require financial statistics of sales to be recorded, etc. Although it is hard to predict how software projects will evolve in the future, the majority of this type of application is clearly presented as such.
* The application has a high read-to-write ratio - That means data is only written a few times, and read many times more. Since data sources for queries are different to those that are used for command validation, it is possible to optimize these data sources for fast querying. Duplicate data is no longer an issue, since events are published when data changes.
* The application presents data in many different formats - Many applications nowadays do not stop when showing information on a web page. Some applications, for example, send monthly emails to notify users of changes that occurred that might be relevant to them. Search engines are another example. They use the same data your application does, but in a way that is optimized for quick searching. Reporting tools aggregate information into reports that show data evolution over time. This, again, is a different format of the same data. Using Axon, each data source can be updated independently of each other on a real-time or scheduled basis.
* The application has clearly separated components with different audiences - An example of such an application is an online store. Employees will update product information and availability on the website, while customers place orders and query for their order status. With Axon, these components can be deployed on separate machines and scaled using different policies. They are kept up-to-date using the events, which Axon will dispatch to all subscribed components, regardless of the machine they are deployed on.
* Integration with other applications can be cumbersome work. The strict definition of an application's API using commands and events makes it easier to integrate with external applications. Any application can send commands or listen to events generated by the application.

## Contributing to Axon Framework

Development on the Axon Framework is never finished. There will always be more features that we like to include in our framework to continue making development of scalable and extensible applications easier. This means we are constantly looking for help in developing our framework.

There are a number of ways in which you can contribute to the Axon Framework:

* You can report any bugs, feature requests or ideas for improvements on our [GitHub](https://github.com/AxonFramework/AxonFramework/issues) issues page. All ideas are welcome. Please be as exact as possible when reporting bugs. This will help us reproduce and thus solve the problem faster.
* If you have created a component for your own application that you think might be useful to include in the Axon Framework, send us a patch or a zip containing the source code. We will evaluate it and try to fit it in the framework. Please make sure code is properly documented using JavaDoc. This helps us to understand what is going on.
* If you know of any other way you think you can help us, do not hesitate to send a message to the [Axon Framework mailing list](mailto:axonframework@googlegroups.com).

## Commercial Support

Axon Framework is open source and freely available for anyone to use. However, if you have specific requirements, or just want to be assured of someone to be standby to help you in case of trouble, AxonIQ provides several commercial support services for Axon Framework. These services include training, consultancy and operational support.

For more information about AxonIQ and its services, visit [axoniq.io](https://axoniq.io) or [axoniq.io/services](https://axoniq.io/services).

## License information

The Axon Framework and its documentation are licensed under the Apache License, Version 2.0. You may obtain a copy of the License at [http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0).

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the [License](http://www.apache.org/licenses/LICENSE-2.0) for the specific language governing permissions and limitations under the License.

