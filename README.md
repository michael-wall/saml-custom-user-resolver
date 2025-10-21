## Introduction ##
- This 'proof of concept' is designed for the scenario of SAML SSO users where the SAML IdP allows duplicate email addresses.
- Liferay does not allow duplicate email addresses or duplicate screenNames within a Virtual Instance.
- When a new or existing user attempts to access the Liferay DXP environment via SAML SSO, the custom code in SAMLCustomUserResolver.java will check if another user with the same email address exists in the Liferay DXP Virtual Instance:
  - If a Liferay user with the email address and the screenName from the SAML claims exists in the Liferay DXP Virtual Instance then no email address change happens as it is assumed to be the same user.
  - If a Liferay user with the same email address but a different screenName from the SAML claims exists in the Liferay DXP Virtual Instance then the email address will have the screenName appended to make the email address unique. 

## Pre-requisite ##
- The solution must be using the out of the box Liferay SAML Service Provider (SP).
- The Control Panel > SAML Admin > SAML IdP **MUST** be configured to Match Users by screenName.
- ScreenName is immutable and unique per User in both the Liferay DXP Virtual Instance AND in the SAML IdP system.

## Setup ##
- Build the custom OSGi module.
- Start the Liferay cluster and deploy the custom OSGi module to each node of the Liferay DXP cluster.
- Confirm that the custom OSGi module deploys without any errors for example:
```
2025-10-21 14:10:15.951 INFO  ... [BundleStartStopLogger:68] STARTED com.mw.saml.custom.user.resolver_1.0.0 [1500]
2025-10-21 14:10:15.955 INFO  ... [SAMLCustomUserResolver:53] Activating
2025-10-21 14:10:15.955 INFO  ... [SAMLCustomUserResolver:55] Activated
```

## SAMLCustomUserResolver Notes ##
- The implementation was tested locally using Liferay DXP 2025.Q1.0 LTS configured as a SAML SP with Keycloak configured as a SAML IdP.
  - However Keycloak doesn't allow duplicate email addresses, so as a workaround for local testing oo replicate the scenario of duplicate email addresses in the IdP, the local testing was done by manually changing screenNames in Liferay DXP for an existing user, then trying to login with SAML SSO with an IdP user with the same email address but a different screenName. In theory this **should** replicate the scenario.
- **Ensure all possible scenarios are tested in a non-prod environment with a SAML IdP that allows duplicate email addresses before considering deploying this in a prod environment.**
- Calls to the _resolveByNameId method in DefaultUserResolver.java have been deactivated as they may interfere with the custom email address change logic.
  - The method uses the Liferay SamlPeerBinding database table to match SAML IdP user records to Liferay User records but the Liferay User values aren't kept in sync with changes in the Liferay User_ table.

## Notes ##
- This is a ‘proof of concept’ that is being provided ‘as is’ without any support coverage or warranty.
- The implementation uses a custom OSGi module meaning it is compatible with Liferay DXP Self-Hosted and Liferay PaaS, but is not compatible with Liferay SaaS.
- The implementation was tested locally using Liferay DXP 2025.Q1.0 LTS configured as a SAML SP with Keycloak configured as a SAML IdP.
- The base DefaultUserResolver.java class that was customized to create SAMLCustomUserResolver.java was taken from the Liferay DXP **2025.Q1.17** source code.
- JDK 21 is expected for both compile time and runtime.
