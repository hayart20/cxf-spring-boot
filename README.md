Apache CXF is an open source services framework that helps build and develop services using frontend programming APIs, like JAX-WS.

In this tutorial, we will take a look at how we can integrate CXF with Spring Boot in order to build and run a Hello World SOAP service. 
Throughout the example, we will be creating a contract first client and endpoint using Apache CXF, Spring Boot, and Maven.

General Project Setup
Tools used:

Apache CXF 3.2
Spring Boot 1.5
Maven 3.5

The below code is organized in such a way that you can choose to only run the client (consumer) or endpoint (provider) part. 
In the example, we will setup both parts and then make an end-to-end test in which the client calls the endpoint.

For this example, we will start from an existing WSDL file (contract-first) which is shown below. 
The content represents a SOAP service in which a person is sent as input and a greeting is received as a response.

The Hello World service endpoint will be hosted on an embedded Apache Tomcat server that ships directly with Spring Boot. 
To facilitate the management of the different Spring dependencies, 
Spring Boot Starters are used which are a set of convenient dependency descriptors that you can include in your application.

To avoid having to manage the version compatibility of the different Spring dependencies, 
we will inherit the defaults from the spring-boot-starter-parent parent POM.

There is actually a Spring Boot starter specifically for CXF that takes care of importing the needed Spring Boot dependencies. 
In addition it automatically registers CXFServlet with a '/services/' 
URL pattern for serving CXF JAX-WS endpoints and it offers some properties for configuration of the CXFServlet. 
In order to use the starter, we declare a dependency to cxf-spring-boot-starter-jaxws in our Maven POM file.

For Unit testing our Spring Boot application we also include the spring-boot-starter-test dependency.

To take advantage of Spring Boot’s capability to create a single, runnable “uber-jar”, 
we also include the spring-boot-maven-plugin Maven plugin. 
This also allows quickly starting the web service via a Maven command.

CXF includes a Maven cxf-codegen-plugin plugin which can generate java artifacts from a WSDL file. 
In the above plugin configuration we’re running the 'wsdl2java' goal in the 'generate-sources' phase. 
When executing following Maven command, CXF will generate artifacts in the '<sourceRoot>' directory that we have specified.

mvn generate-sources

After running above command you should be able to find back a number of auto-generated classes among which the HelloWorldPortType interface in addition to Person and Greeting

Next, we create a SpringCxfApplication class. It contains a main() method that delegates to Spring Boot’s SpringApplication class by calling run(). 
SpringApplication will bootstrap our application, starting Spring which will, in turn, start the auto-configured Tomcat web server.

The @SpringBootApplication is a convenience annotation that adds all of the following: @Configuration, @EnableAutoConfiguration and @ComponentScan. 
Checkout the Spring Boot getting started guide for more details.

Creating the CXF Endpoint (Provider)
In order for the CXF framework to be able to process incoming SOAP request over HTTP, we need to setup a CXFServlet. 
In our previous CXF SOAP Web Service tutorial we did this by using a deployment descriptor file (web.xml file under the WEB-INF directory) 
or an alternative with Spring is to use a ServletRegistrationBean. In this example, 
there is nothing to be done as the cxf-spring-boot-starter-jaxws automatically register the CXFServlet for us, great!

We want the CXFServlet to listen for incoming requests on the following URI: “/codenotfound/ws”, instead of the default value which is: “/services/*”. 
This can be achieved by setting the 'cxf.path' property in the application.yml file located under the src/main/resources folder.

Next, we create a configuration file that contains the definition of our Endpoint at which our Hello World SOAP service will be exposed. 
The Endpoint gets created by passing the CXF bus, 
which is the backbone of the CXF architecture that manages the respective inbound and outbound message and fault interceptor chains for all client and server endpoints. 
We use the default CXF bus and get a reference to it via Spring’s @Autowired annotation.

In addition to the bus we also specify the HelloWorldImpl class which contains the actual implementation of the service. 
Finally we set the URI on which the endpoint will be exposed to “/helloworld”. 
Together with the 'cxf.path' configuration in the application.yml file this result into following URL that clients will need to call:

The service implementation is specified the HelloWorldImpl POJO that implements the HelloWorldPortType interface that was generated from the WSDL file earlier. 
We simply use the name of the received Person to construct a Greeting that is returned. 

Creating the CXF Client (Consumer)
CXF provides a JaxWsProxyFactoryBean that will create a Web Service client for you which implements a specified service class.

Let’s create a ClientConfig class with the @Configuration annotation which indicates that the class can be used by the Spring IoC container as a source of bean definitions. 
Next we create a JaxWsProxyFactoryBean and set HelloWorldPortType as service class.

The last thing we set is the endpoint at which the Hello World service is available. This endpoint is fetched from the application.yml file so it can easily be changed if needed.

Don’t forget to call the create() method on the JaxWsProxyFactoryBean in order to have the Factory create a JAX-WS proxy that we can then use to make remote invocations.

Below HelloWorldClient provides a convenience sayHello() method that will create a Person object based on a first and last name. 
It then uses the auto-wired helloWorldClientProxyBean to invoke the Web Service. 
The result is a Greeting that is returned as a String.

Annotating our class with the @Component annotation will cause Spring to automatically import this bean into the container if automatic component scanning is enabled. 
This is the case as in the beginning we added the @SpringBootApplication annotation to the main SpringWsApplication class which is equivalent to using @ComponentScan.

Testing the Client & Endpoint
To wrap up the tutorial, we will create a basic test case that will use our Hello World client to send a request to the CXF endpoint 
that is being exposed on Spring Boot’s embedded Tomcat.

The new @RunWith and @SpringBootTest testing annotations, that are available as of Spring Boot 1.4, 
are used to tell JUnit to run using Spring’s testing support and bootstrap with Spring Boot’s support.

By default, the embedded HTTP server will be started on a random port. 
As we have defined the URL which the client needs to call with a specific port number, we need to set the DEFINED_PORT web environment. 
This will cause Spring to use the 'server.port' property from the application.properties file instead of a random one.