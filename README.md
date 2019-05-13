# configserver-service

This is a basic Spring Cloud config server example. Actual externalized properties are commited in another github repository.

Tutorial to refer [https://sivalabs.in/2017/08/spring-cloud-tutorials-introduction-to-spring-cloud-config-server/]

Bring up the config server on localhost and hit the url - http://localhost:8001/product-service/dev

This will show up:
'''
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
'''
