## Introduction ##
- This 'proof of concept' is designed for the scenario of SAML SSO users where the SAML IdP allows duplicate email addresses. Liferay does not allow duplicate email addresses (or dpplicate screenNames) within a Virtual Instance.
- When a new or existing user attempts to access the Liferay DXP environment via SAML SSO the custom code in SAMLCustomUserResolver.java will check if another user with the same email address exists in the Liferay DXP Virtual Instance:
  - If a Liferay user with the same email address and the same screenName from the claims exists in the Liferay DXP Virtual Instance then no email address change happens as it is assumed to be the same user.
  - If a Liferay user with the same email address but a different screenName exists in the Liferay DXP Virtual Instance then the email address will have the screenName appended to make the email address unique.

## Pre-requisite ##
- The solution must be using the out of the box Liferay SAML Service Provider (SP).
- The Control Panel > SAML Admin > SAML IdP **MUST** be configured to Match Users by screenName.

## Setup ##
- Build the custom OSGi module.
- Start the Liferay cluster and deploy the custom OSGi module to each node of the Liferay DXP cluster.
- Confirm that the custom OSGi module deploys without any errors for example:
```
2025-10-21 14:10:15.951 INFO  ... [BundleStartStopLogger:68] STARTED com.mw.saml.custom.user.resolver_1.0.0 [1500]
2025-10-21 14:10:15.955 INFO  ... [SAMLCustomUserResolver:53] Activating
2025-10-21 14:10:15.955 INFO  ... [SAMLCustomUserResolver:55] Activated
```

## Notes ##
- This is a ‘proof of concept’ that is being provided ‘as is’ without any support coverage or warranty.
- The implementation uses a custom OSGi module meaning it is compatible with Liferay DXP Self-Hosted and Liferay PaaS, but is not compatible with Liferay SaaS.
- The implementation was tested locally using Liferay DXP 2025.Q1.0 LTS configured as a SAML SP with Keycloak configured as a SAML IdP.
- The base DefaultUserResolver class that was customized to create SAMLCustomUserResolver was taken from **2025.Q1.17**.
- JDK 21 is expected for both compile time and runtime.
- The module can be deployed to all nodes in the cluster, but ensure that restrict.access.login.event.enabled is not set to true on each node, otherwise privileged users will be prevented from logging to any of the cluster nodes with SAML SSO.
