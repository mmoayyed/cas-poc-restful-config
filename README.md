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
