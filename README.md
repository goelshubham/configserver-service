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



