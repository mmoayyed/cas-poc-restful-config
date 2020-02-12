CAS POC - REST-based Configuration
=======================

Generic CAS WAR overlay to exercise the latest versions of CAS.

# Versions

- CAS `6.2.x`
- JDK `11`

# Overview

This is a modest POC to demonstrate how a given CAS server can bootstrap itself using a REST API
and auto-construct identity providers for delegated authentication. The goal of the POC is to explore aspects and possibilities
for a CAS server, as a Spring Boot/Spring Cloud application, to dynamically alter its behavior and configuration at runtime
using a configuration store that might be externally managed elsewhere.

# Build

The following modules are turned on in the build:

- Delegated authentication via Pac4j

```groovy
    implementation "org.apereo.cas:cas-server-support-pac4j-webflow:${casServerVersion}"
```

This is to demonsrate the ability to auto-create and auto-configure external identity providers for delegated authntication.

- Spring Cloud Configuration via REST (New in CAS 6.2.x)

```groovy
    implementation "org.apereo.cas:cas-server-support-configuration-cloud-rest:${casServerVersion}"
```

Allow the CAS server to bootstrap and initialize itself with settings fetched from a REST endpoint.

- Configuration Event Management

```groovy
    implementation "org.apereo.cas:cas-server-core-events-configuration:${casServerVersion}"
```

Allow the CAS server to intercept configuration events and reload/refresh itself dynamically when requested via Spring Cloud Actuator endpoints.

# Run

Use:

```
./runpoc.sh
```

If you exmaine the above script, you will find that the CAS server is configured to fetch its settings from a REST endpoint, whose location is taught to CAS as a system property at runtime:

```bash
./gradlew build run -Dcas.spring.cloud.rest.url=https://demo5926981.mockable.io/casproperties
```

This will activate the `RestfulCloudConfigBootstrapConfiguration` in CAS to bootstrap the application context using a property source that is provided by the REST API. 

The API is expected to return a JSON map of relevant CAS settings:

```json
{
    "cas.authn.pac4j.cas[0].loginUrl": "https://casserver.herokuapp.com/cas/login",
    "cas.authn.pac4j.cas[0].protocol": "CAS30",
    "cas.authn.pac4j.cas[0].clientName" : "CAS1",
    "cas.authn.pac4j.cas[0].enabled": true,    
    
    "cas.authn.pac4j.cas[1].loginUrl": "https://casserver.herokuapp.com/cas/login",
    "cas.authn.pac4j.cas[1].protocol": "CAS30",
    "cas.authn.pac4j.cas[1].clientName" : "CAS2",
    "cas.authn.pac4j.cas[1].enabled": true,
    
    "management.endpoints.web.exposure.include": "*",
    "management.endpoints.enabled-by-default": "true",
    "cas.monitor.endpoints.endpoint.defaults.access":"AUTHENTICATED",
    "spring.security.user.name": "casuser",
    "spring.security.user.password" : "Mellon",
    
    "server.port": 8080,
    "server.ssl.enabled": false,
    "cas.server.name": "http://localhost:8080",
    "cas.server.prefix": "http://localhost:8080/cas",
    
    "logging.level.org.springframework.boot": "debug"
}
```

Items to note:

1. We are declaring two external identity providers (CAS servers) for delegated authentication
2. We are exposing all CAS actuator endpoints (powered by Spring Boot) and securing them using basic authN via `casuser:Mellon` (More on this later)
3. Other misc settings for server name, description, and logging config.

When you run the POC, you should expect the following lines in the logs:

```bash
...
INFO [org.springframework.boot.web.embedded.tomcat.TomcatWebServer] - <Tomcat initialized with port(s): 8080 (http)>
...
INFO [org.apereo.cas.support.pac4j.config.support.authentication.Pac4jAuthenticationEventExecutionPlanConfiguration] - <Located and prepared [2] delegated authentication client(s)>
NFO [org.apereo.cas.support.pac4j.config.support.authentication.Pac4jAuthenticationEventExecutionPlanConfiguration] - <Registering delegated authentication clients...>
...
```

...and if you go to `http://localhost:8080/cas/login`, you should see:

![image](https://user-images.githubusercontent.com/1205228/74315845-c772af00-4d91-11ea-9de4-34fb222581fd.png)

# Refresh & Reload

The CAS server is able to dynamically alter itself once it receives a refresh request. Using this strategy, one can modify values and settings in the configuration store (backed by REST), and then
send a notification request to the CAS server to update its state. The configuration management, retrieval and refreshability of components are all managed by Spring Cloud and family.

As an example, you can modify the configuration store to match the following:


```json
{
    "cas.authn.pac4j.cas[0].loginUrl": "https://casserver.herokuapp.com/cas/login",
    "cas.authn.pac4j.cas[0].protocol": "CAS30",
    "cas.authn.pac4j.cas[0].clientName" : "CAS Here",
    "cas.authn.pac4j.cas[0].enabled": true,    
    
    "management.endpoints.web.exposure.include": "*",
    "management.endpoints.enabled-by-default": "true",
    "cas.monitor.endpoints.endpoint.defaults.access":"AUTHENTICATED",
    "spring.security.user.name": "casuser",
    "spring.security.user.password" : "Mellon",
    
    "server.port": 8080,
    "server.ssl.enabled": false,
    "cas.server.name": "http://localhost:8080",
    "cas.server.prefix": "http://localhost:8080/cas",
    
    "logging.level.org.springframework.boot": "debug",
    "logging.level.org.apereo.cas": "debug"
}
```

Changes are:

1. We renamed one identity provider to be `CAS Here`
2. We removed the second identity provider
2. We enabled `DEBUG` logging for `org.apereo.cas`

Once changes are saved, the CAS server can begin to refresh its state via:

```bash
curl -k -u casuser:Mellon http://localhost:8080/cas/actuator/refresh -d {} -H "Content-Type: application/json"
```

The response should outline the collection of settings that were affected and refreshed. 

...and if you go to `http://localhost:8080/cas/login`, you should see:

![image](https://user-images.githubusercontent.com/1205228/74317245-7dd79380-4d94-11ea-924f-b8f59620cea5.png)