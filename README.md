# configserver-service

This is a basic Spring Cloud config server example. Actual externalized properties are commited in another github repository.

Tutorial to refer:
1. https://sivalabs.in/2017/08/spring-cloud-tutorials-introduction-to-spring-cloud-config-server/
2. https://failedtofunction.com/configuration-with-spring-cloud-config/
3.
4.
5. 

Bring up the config server on localhost and hit the url - http://localhost:8001/product-service/dev

This will show up:
```
{
  name: "product-service",
  profiles: [
    "dev"
  ],
  label: null,
  version: "c49507cf24dd68bdf1e0eca172be6e133d5fd385",
  state: null,
  propertySources: [
    {
      name: "https://github.com/goelshubham/external-properties.git/product-service/product-service-dev.properties",
      source: {
        property_key: "property_value",
        key.com: "shubham"
      }
    }
  ]
}
```
Spring Cloud Config Server exposes the following REST endpoints to get application specific configuration properties:
```
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```
Here {application} refers to value of spring.config.name property, {profile} is an active profile and {label} is an optional git label (defaults to “master”).

### Encrypting properties in the externalized repository
We are saving all the properties of our microservices in a git repo but there might be some important properties which we want to hide or encrypt in order to protect from misuse.

Spring Cloud config server provides two REST apis to encrypt and decrypt such important properties. One feature that I like is the presence of /encrypt and /decrypt endpoints which accept POST requests with a payload to be encrypted. A private key needs to be configured for the server. For test purposes you can add this to the bootstrap.properties file of the config server, but ideally it is stored securely separate from source.

Steps to follow:
1. To use the encryption and decryption features you need the full-strength JCE installed in your JVM (it’s not there by default). You can download the "Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files" from Oracle, and follow instructions for installation (essentially replace the 2 policy files in the JRE lib/security directory with the ones that you downloaded).

2. add bootstrap.properties file in config server and enter a property
```
encrypt.key=secret
```
The phrase 'secret' cam be anything confidential. Now
3. Now go to postman and hit the configserver url (with /encrypt) with POST request and in body section enter the property value which you want to encrypt.

We can also use curl commands...

```
curl -X POST localhost:8001/encrypt -d shubham
2d64749ca1397e19176839a3430aa2566686eb522786a9ca2874655100c17cc1
```

4. Now we encrypted the value 'shubham' to 2d64749ca1397e19176839a3430aa2566686eb522786a9ca2874655100c17cc1. In application property file of application we need to follow {cipher}2d64749ca1397e19176839a3430aa2566686eb522786a9ca2874655100c17cc1 in order to tell config server to decrypt the value before using it.

Now I changed a property in application properties:
```
property_key=property_value
key.com=shubham
encrypted.key={cipher}dd7fba640c89443459aafba14cb72336713dd5897f21210cf0aef985f3010692
```
And when I hit http://localhost:8001/product-service/dev, I get following result where I see decrypted value

```
{
  name: "product-service",
  profiles: [
    "dev"
  ],
  label: null,
  version: "fbb2d3e0278d75b92451cd2f374b9eb64afd391a",
  state: null,
  propertySources: [
    {
      name: "https://github.com/goelshubham/external-properties.git/product-service/product-service-dev.properties",
      source: {
        property_key: "property_value",
        key.com: "shubham",
        encrypted.key: "dbpassword"
      }
    }
  ]
}
```

### Changing application properties with @RefreshScope and without the need to restart application

Annotate your application controller class with @RefreshScope like below:
```
@RefreshScope
@RestController
@RequestMapping(value="/productservice")
public class ProductServiceController {
	
	@Value("${key.com}")
	private String key;
	
	@RequestMapping(value="/products", method=RequestMethod.GET)
	public String getProducts() {
		return "Property from external file is -> " + key;
	}

}
```
As per the tutorial, once we change property in external application property file, we need to hit /refresh endpoint to get the properties reflected but when I tried I didn't need to explicitly refresh them. 

There is a limitation in using only @RefreshScope and /refresh endpoint and the limitation is that we need to do it for all the microservices. If we change a prop of one microservice, we need to do /refresh for that application.

Spring Cloud suggests to use Spring Cloud Bus. Spring Cloud Bus links the nodes of a distributed system with a lightweight message broker. This broker can then be used to broadcast state changes (such as configuration changes) or other management instructions. A key idea is that the bus is like a distributed actuator for a Spring Boot application that is scaled out. However, it can also be used as a communication channel between apps. This project provides starters for either an AMQP broker or Kafka as the transport.

Refer this official documentation here [https://cloud.spring.io/spring-cloud-static/spring-cloud-bus/2.1.0.RELEASE/single/spring-cloud-bus.html]

My changes are as follow:
1. Installed RabbitMQ on local machine.
2. In microservice, product-service, I added following properties in application properties to connect it with RabbitMQ installation.
```
spring.rabbitmq.host=localhost
# by default rabbit mq starts on port 5672. I have verified it on my Mac
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
```
3. bootstrap.yml looks like below:
```
spring:
  cloud:
    config:
     uri: http://localhost:8001
     failFast: true
     name: product-service
---
spring:
  profiles: dev
---
spring:
  profiles:
    active: dev
---
legacy:
  context:
    enabled: true
management:
  endpoint:
    env:
      enabled: true 
    health:
      enabled: true
    info:
      enabled: true
    threaddump:
      enabled: true
    refresh:
      enabled: true
  endpoints:
    web:
      exposure:
        include: env, bus-refresh
```
4. Now change a property in externalized property files. And hit this endpoint - http://localhost:8002/actuator/bus-refresh
This will refresh all properties for all microservices configured to same config server.

Please note all this cloud bus configuration can also be done in config server itself instead of microservices.
1. I added spring-cloud-starter-bus-amqp in pom.xml of config server and added same properties in application.properties and bootstrap.yml file.
2. Now refreshing from cloud config server also refreshes all properties - http://localhost:8001/actuator/bus-refresh


