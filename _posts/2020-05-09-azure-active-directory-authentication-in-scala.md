---
layout: post
title: Using Spring/Azure authentication in Scala
---
# Using Spring/Azure authentication in Scala

One of the big concerns when starting to use Scala in commercial projects is how you can leverage the existing wider JVM
ecosystem of libraries. Scala is a relatively younger and highly productive language, but there are many popular, well 
tested Java libraries and frameworks e.g. for web developments such as Spring. 

For example, you want to implement JWT authentication with Azure Active Directory in Scala, browsing the [Microsoft Authentication Library](https://github.com/AzureAD/microsoft-authentication-library-for-java) 
you'll find that they have a lot of facilities for fetching jwt in different scenarios but nothing related to validating
an incoming request with embedded jwt. This is because the library is tightly coupled with Spring framework in term of 
handling a request with jwt header and when it comes to validating the token, it will simply leverage the Spring 
`security.oauth2.client.*` settings.

However, if you want to validate a raw JWT in Scala, here is how you can do it using the [azure-spring-boot](https://github.com/microsoft/azure-spring-boot)
library, specifically the [UserPrincipalManager](https://github.com/microsoft/azure-spring-boot/blob/master/azure-spring-boot/src/main/java/com/microsoft/azure/spring/autoconfigure/aad/UserPrincipalManager.java)
class similarly to below

```scala
val aadAuthProps: AADAuthenticationProperties = new AADAuthenticationProperties()
aadAuthProps.setAllowTelemetry(false)
aadAuthProps.setSessionStateless(true)
aadAuthProps.setClientId(clientId)
aadAuthProps.setClientSecret(clientSecret)
aadAuthProps.setTenantId(azureTenant)

val serviceEndpoints = new ServiceEndpoints()
serviceEndpoints.setAadGraphApiUri("https://graph.windows.net/")
serviceEndpoints.setAadKeyDiscoveryUri("https://login.microsoftonline.com/common/discovery/keys/")
serviceEndpoints.setAadMembershipRestUri("https://graph.windows.net/me/memberOf?api-version=1.6")
serviceEndpoints.setAadSigninUri("https://login.microsoftonline.com/")
val serviceEndpointsProperties = new ServiceEndpointsProperties()
serviceEndpointsProperties.getEndpoints.put("global", serviceEndpoints)

val userPrincipalManager: UserPrincipalManager = new UserPrincipalManager(
serviceEndpointsProperties,
aadAuthProps,
resourceRetriever,
explicitAudienceCheck
)

def validateToken(token: String): Try[UserPrincipal] = Try(userPrincipalManager.buildUserPrincipal(token))
```

Notice how `aadAuthProps` and `serviceEndpoints` are constructed in a way that is somewhat counterintuitive. 
This is because normally these instance will be automatically instantiated using Spring dependency injections, e.g.
reading the configurations from a `*.properties` file embedded inside the `azure-spring-boot` library. So even though 
you'll be working with Scala, when it comes to using Java libraries it still very doable, but you'll have to mentally 
switch back and forth between doing things the Scala v.s. Java ways.