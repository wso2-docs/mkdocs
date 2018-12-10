## __<font color="#455A64">Architecture and Process Flow</font>__
WSO2 IS is a product built on top of WSO2 Carbon. Based on the OSGi specification, it enables easy customization and extension through its componentized architecture. 

Watch the following video for a quick overview of the process flow of the Identity Server architecture and how the various components interact with each other.

<iframe width="640" height="360" src="https://www.youtube.com/embed/ZnWnDZJ_c4o" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## __<font color="#455A64">Authentication Framework</font>__
The following are the authenticator types in the authentication framework. 

### __Inbound Authenticators__
The responsibility of inbound authenticators is to identify and parse all the incoming authentication requests and then build the corresponding response. A given inbound authenticator has two parts.

1. Request Processor
2. Response Builder 

For each protocol supported by WSO2 Identity Server, there should be an inbound authenticator. This architecture component includes inbound authenticators for [Security Assertion Markup Language (SAML) 2.0](http://saml.xml.org/saml-specifications), [OpenID Connect](http://openid.net/connect/), [OAuth 2.0](https://oauth.net/2/), and [WS-Federation (passive)](http://docs.oasis-open.org/wsfed/federation/v1.2/ws-federation.html). In other words, the responsibility of the SAML 2.0 request processor is to accept a SAML request from a service provider, validate the SAML request and then build a common object model understood by the authentication framework and handover the request to it. The responsibility of the SAML response builder is to accept a common object model from the authentication framework and build a SAML response out of it. Both the request processors and the response builders are protocol aware, while the authentication framework is not coupled to any protocol.

### __Local Authenticators__
The responsibility of the local authenticators is to authenticate the user with locally available credentials. This can be either user name/password or even IWA (Integrated Windows Authentication). Local authenticators are decoupled from the Inbound Authenticators. Once the initial request is handed over to the authentication framework from an inbound authenticator, the authentication framework talks to the service provider configuration component to find the set of local authenticators registered with the service provider corresponding to the current authentication request.

Once the local authentication is successfully completed, the local authenticator will notify the framework. The framework will now decide no more authentication is needed and hand over the control to the corresponding response builder of the inbound authenticator.

You can develop your own local authenticators and plug them into the Identity Server.

### __Outbound/Federated Authenticators__
The responsibility of the federated authenticators is to authenticate the user with an external system. This can be with Facebook, Google, Yahoo, LinkedIn, Twitter, Salesforce or any other identity provider. Federated authenticators are decoupled from the Inbound Authenticators. Once the initial request is handed over to the authentication framework from an inbound authenticator, the authentication framework talks to the service provider configuration component to find the set of federated authenticators registered with the service provider corresponding to the current authentication request.

A federated authenticator has no value unless it is associated with an identity provider. The Identity Server out-of-the-box supports Security Assertion Markup Language (SAML) 2.0, OpenID Connect, OAuth 2.0, and WS-Federation (passive). The SAML 2 .0 federated authenticator itself has no value. It has to be associated with an Identity Provider. Google Apps can be an identity provider - with the SAML 2.0 federated authenticator. This federated authenticator knows how to generate a SAML request to the Google Apps and process a SAML response from it.

There are two parts in a federated authenticator.

1. Request Builder
2. Response Processor

Once the federation authentication is successfully completed, the federated authenticator will notify the authentication framework. The framework will now decide no more authentication is needed and hand over the control to the corresponding response builder of the inbound authenticator.

Both the request builder and the response processor are protocol aware while the authentication framework is not coupled to any protocol.

You can develop your own federated authenticators and plug them into the Identity Server. 

### __Multi-Option Authenticators__
The service provider can define how to authenticate users at the Identity Server, for authentication requests initiated by it. While doing that, each service provider can pick more than one authenticator to allow end users to get multiple login options. This can be a combination of local authenticators and federated authenticators.

### __Multi-Factor Authenticators__
The service provider can define how to authenticate users at the Identity Server, for authentication requests initiated by it. While doing that, each service provider can define multiple steps and for each step it can pick more than one authenticator. The authentication framework tracks all the authenticators in each step and proceeds to the next step only if the user authenticates successfully in the current step. It is an AND between steps, while it is an OR between the authenticators in a given step.

## __<font color="#455A64">Provisioning Framework</font>__
The following are the provisioning components available in the provisioning framework.


### __Inbound provisioning__
Inbound provisioning focuses on how to provision users to the Identity Server. Out-of-the-box, the Identity Server supports inbound provisioning via a Simple Object Access Protocol (SOAP) based API as well as the System for Cross-domain Identity Management (SCIM) 1.1 API. Both the APIs support HTTP Basic Authentication. If you invoke the provisioning API with Basic Authentication credentials, then where to provision the user (to which user store) will be decided based on the inbound provisioning configuration of the resident service provider.

The SCIM API also supports OAuth 2.0. If the user authenticates to the SCIM API with OAuth credentials, then the system will load the configuration corresponding to the service provider who owns the OAuth client id. If you plan to invoke the SCIM API via a web application or a mobile application, we would highly recommend you to use OAuth instead of Basic Authentication. You simply need to register your application as a service provider in Identity Server and then generate OAuth keys.

### __Just-in-time provisioning__
Just-in-time (JIT) provisioning talks about how to provision users to the Identity Server at the time of federated authentication. A service provider initiates the authentication request, the user gets redirected to the Identity Server and then Identity Server redirects the user to an external identity provider for authentication. Just-in-time provisioning gets triggered in such a scenario when the Identity Server receives a positive authentication response from the external identity provider. The Identity Server will provision the user to its internal user store with the user claims from the authentication response.

You configure JIT provisioning against an identity provider - not against service providers. Whenever you associate an identity provider with a service provider for outbound authentication, if the JIT provisioning is enabled for that particular identity provider, then the users from the external identity provider will be provisioned into the Identity Server's internal user store. In the JIT provisioning configuration you can also pick the provisioning user store.

JIT provisioning happens while in the middle of an authentication flow. The provisioning can happen in a blocking mode or in a non-blocking mode. In the blocking mode, the authentication flow will be blocked till the provisioning finishes - while in the non-blocking mode, provisioning happens in a different thread.

### __Outbound provisioning__
Outbound provisioning talks about provisioning users to external systems. This can be initiated by any of the following.

Inbound provisioning request (initiated by a service provider or the resident service provider)
JIT provisioning (initiated by a service provider)
Adding a user via the management console (initiated by the resident service provider)
Assigning a user to a provisioning role (initiated by the resident service provider)
WSO2 Identity Server supports outbound provisioning with the following connectors. You need to configure one or more outbound provisioning connectors with a given identity provider, and associate the identity provider with a service provider. All the provisioning requests must be initiated by a service provider - and will be provisioned to all the identity providers configured in the outbound provisioning configuration of the corresponding service provider.

* SCIM
* SPML
* SOAP
* Google Apps provisioning API
* Salesforce provisioning API

## __<font color="#455A64">Components</font>__
Following are the components pertaining to the architecture of the WSO2 Identity Server, which are depicted in the above figure and video.

### __Service Providers__

__Description__
</br>
A Service Provider (SP) is an entity that provides Web services. A service provider relies on a trusted Identity Provider (IdP) for authentication and authorization. In this case, the Identity Server acts as the IdP and does the task of authenticating and authorizing the user of the service provider.

Salesforce and Google Apps are examples of service providers and are used as such in this case.

!!! info "Related Links"
	For more informatino on how to add an SP to WSO2 IS, see [Adding and Configuring a Service Provider](https://docs.wso2.com/display/IS570/Adding+and+Configuring+a+Service+Provider).

__Process Flow__
</br>
A user of the SP attempts to log into the SPs application. The service provider sends an authentication request to the Identity Server. This request is met by the Inbound Authentication component of the Identity Server and comes in one of the following forms.

* SAML SSO
* OAuth/OpenID Connect
* Passive STS

![screenshot](images/serviceprovider_inboundauthentication.png)

The service provider receives the authentication confirmation from the Identity Server once it follows all the specified processes required in order to authenticate the SP's user. 

Additionally, if a user registers in the service provider's application, a Simple Object Access Protocol (SOAP) or System for Cross-domain Identity Management (SCIM) request can be sent to the Identity Server. The request is met by the Inbound Provisioning component of the Identity Server.

![screenshot](images/serviceprovider_inboundprovisioning.png)


### __Inbound Authentication__
### __Authentication Framework__
### __Local Authenticators__
### __Federated Authenticators__
### __Identity Providers__
### __Provisioning Framework__
### __Authroization Manager__
### __IDP and SP Configurations__
### __Inbound Provisioning__
### __User Store Manager__
### __Claim Manager__
### __XACML__
### __Auditing__
### __Outbound Provisioning__

!!! info "Related Links"
	For further reading about the architecture in an Identity and Access Management solution, see the following article: [Identity Architect Ground Rules: Ten IAM Design Principles](https://wso2.com/whitepapers/identity-architect-ground-rules-ten-iam-design-principles/).
