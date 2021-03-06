---
layout: default
title: CAS - Multifactor Authentication Triggers
---

# Multifactor Authentication Triggers

The following triggers can be used to activate and instruct CAS to navigate to a multifactor authentication flow.
To see the relevant list of CAS properties, please [review this guide](Configuration-Properties.html#multifactor-authentication).

## Global

MFA can be triggered for all applications and users regardless of individual settings.

To see the relevant list of CAS properties, please [review this guide](Configuration-Properties.html#multifactor-authentication).

## Applications

MFA can be triggered for a specific application registered inside the CAS service registry.

```json
{
  "@class" : "org.apereo.cas.services.RegexRegisteredService",
  "serviceId" : "^(https|imaps)://.*",
  "id" : 100,
  "name": "test",
  "multifactorPolicy" : {
    "@class" : "org.apereo.cas.services.DefaultRegisteredServiceMultifactorPolicy",
    "multifactorAuthenticationProviders" : [ "java.util.LinkedHashSet", [ "mfa-duo" ] ]
  }
}
```

## Global Principal Attribute

MFA can be triggered for all users/subjects carrying a specific attribute that matches one of the conditions below.

- Trigger MFA based on a principal attribute(s) whose value(s) matches a regex pattern.
**Note** that this behavior is only applicable if there is only a **single MFA provider** configured, since that would allow CAS
to know what provider to next activate.

- Trigger MFA based on a principal attribute(s) whose value(s) **EXACTLY** matches an MFA provider.
This option is more relevant if you have more than one provider configured or if you have the flexibility of assigning
provider ids to attributes as values.

Needless to say, the attributes need to have been resolved for the principal prior to this step.

## Global Authentication Attribute

MFA can be triggered for all users/subjects whose *authentication event/metadata* has resolved a specific attribute
that matches one of the below conditions:

- Trigger MFA based on a *authentication attribute(s)* whose value(s) matches a regex pattern.
**Note** that this behavior is only applicable if there is only a **single MFA provider** configured, since that would allow CAS
to know what provider to next activate.

- Trigger MFA based on a *authentication attribute(s)* whose value(s) **EXACTLY** matches an MFA provider.
This option is more relevant if you have more than one provider configured or if you have the flexibility of assigning
provider ids to attributes as values.

Needless to say, the attributes need to have been resolved for the authentication event prior to this step. This trigger
is generally useful when the underlying authentication engine signals CAS to perform additional validation of credentials.
This signal may be captured by CAS as an attribute that is part of the authentication event metadata which can then trigger
additional multifactor authentication events.

An example of this scenario would be the "Access Challenge response" produced by RADIUS servers.

## Adaptive

MFA can be triggered based on the specific nature of a request that may be considered outlawed. For instance,
you may want all requests that are submitted from a specific IP pattern, or from a particular geographical location
to be forced to go through MFA. CAS is able to adapt itself to various properties of the incoming request
and will route the flow to execute MFA. See [this guide](Configuring-Adaptive-Authentication.html) for more info.

## Grouper

MFA can be triggered by [Grouper](https://www.internet2.edu/products-services/trust-identity-middleware/grouper/)
groups to which the authenticated principal is assigned.
Groups are collected by CAS and then cross-checked against all available/configured MFA providers.
The group's comparing factor **MUST** be defined in CAS to activate this behavior
and it can be based on the group's name, display name, etc where
a successful match against a provider id shall activate the chosen MFA provider.

```xml
<dependency>
  <groupId>org.apereo.cas</groupId>
  <artifactId>cas-server-support-grouper</artifactId>
  <version>${cas.version}</version>
</dependency>
```

You will also need to ensure `grouper.client.properties` is available on the classpath
with the following configured properties:

```properties
grouperClient.webService.url = http://192.168.99.100:32768/grouper-ws/servicesRest
grouperClient.webService.login = banderson
grouperClient.webService.password = password
```

## REST

MFA can be triggered based on the results of a remote REST endpoint of your design. If the endpoint is configured,
CAS shall issue a `POST`, providing the principal id. The body of the response in the event of a successful `200`
status code is expected to be the MFA provider id which CAS should activate.

## Opt-In Request Parameter

MFA can be triggered for a specific authentication request, provided
the initial request to the CAS `/login` endpoint contains a parameter
that indicates the required MFA authentication flow. The parameter name
is configurable, but its value must match the authentication provider id
of an available MFA provider described above.

```bash
https://.../cas/login?service=...&<PARAMETER_NAME>=<MFA_PROVIDER_ID>
```

## Principal Attribute Per Application

As a hybrid option, MFA can be triggered for a specific application registered inside the CAS service registry, provided
the authenticated principal carries an attribute that matches a configured attribute value. The attribute
value can be an arbitrary regex pattern. See below to learn about how to configure MFA settings.

```json
{
  "@class" : "org.apereo.cas.services.RegexRegisteredService",
  "serviceId" : "^(https|imaps)://.*",
  "id" : 100,
  "name": "test",
  "multifactorPolicy" : {
    "@class" : "org.apereo.cas.services.DefaultRegisteredServiceMultifactorPolicy",
    "multifactorAuthenticationProviders" : [ "java.util.LinkedHashSet", [ "mfa-duo" ] ],
    "principalAttributeNameTrigger" : "memberOf",
    "principalAttributeValueToMatch" : "faculty|allMfaMembers"
  }
}
```

## Entity Id Request Parameter

In situations where authentication is delegated to CAS, most commonly via a [Shibboleth Identity Provider](https://shibboleth.net/products/identity-provider.html),
the entity id may be passed as a request parameter to CAS to be treated as a CAS registered service.
This allows one to [activate multifactor authentication policies](#applications) based on the entity id that is registered
in the CAS service registry. As a side benefit, the entity id can take advantage of all other CAS features
such as access strategies and authorization rules simply because it's just another service definition known to CAS.

To learn more about integration options and to understand how to delegate authentication to CAS 
from a Shibboleth identity provider, please [see this guide](../integration/Shibboleth.html).

Support is enabled by including the following dependency in the WAR overlay:

```xml
<dependency>
  <groupId>org.apereo.cas</groupId>
  <artifactId>cas-server-support-shibboleth</artifactId>
  <version>${cas.version}</version>
</dependency>
```

The `entityId` parameter may be passed as such:

```bash
https://.../cas/login?service=http://idp.example.org&entityId=the-entity-id-passed
```

## Custom

While support for triggers may seem extensive, there is always that edge use case that would have you trigger MFA based on a special set of requirements. To learn how to design your own triggers, [please see this guide](Configuring-Multifactor-Authentication-CustomTriggers.html).