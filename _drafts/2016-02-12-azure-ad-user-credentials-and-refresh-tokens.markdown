---
author: micdemarco
comments: true
date: 2016-02-12 14:19:19+00:00
layout: post
link: https://micdemarco.wordpress.com/2016/02/12/azure-ad-user-credentials-and-refresh-tokens/
published: false
slug: azure-ad-user-credentials-and-refresh-tokens
title: Azure AD Client vs User Credentials and Refresh Tokens
wordpress_id: 75
---

Refresh credentials are great because they can allow a client to access a resource over a period of time without having to re-authenticate.

In this blog post I am going to describe a particular scenario using Azure AD and Web Api.  I will not go into too much detail about setting up things on azure since the [samples](https://github.com/Azure-Samples) do a great job if this.

## Story

I have a specific scenario where:
	
  1. I would like to secure my Web API with Azure AD

	
  2. I would like my SPA Client to use OAuth2 access tokens in order to access the Web API

	
  3. I do not want an interactive login


I have 2 options:

	
  1. Include a secret key to my SPA in order for it to be able to generate tokens

	
  2. Supply my SPA Client with a refresh token during the initial page load, so that it could generate access tokens


I would prefer the second method mainly because

	
  1. I could generate as many refresh tokens as I want without supplying my secret.

	
  2. Refresh tokens will eventually expire as described [here](http://www.cloudidentity.com/blog/2015/03/20/azure-ad-token-lifetime/)


## First Method - Client Credentials - No Refresh Token


I first tried to generate a refresh token using the client credentials where you supply a client Id and a client Secret as described [here](https://msdn.microsoft.com/en-us/library/azure/dn645543.aspx).

However this method did not work for me since client credentials only returned an access token, and **no refresh token**.

Side Note - While searching for a solution I came across this interesting blog post describing [how to create secret keys with extended expiry dates](http://www.silver-it.com/node/203).


## Second Method - User Credentials - Refresh Token Available


After some digging around, I found the [native headless sample](https://github.com/Azure-Samples/active-directory-dotnet-native-headless) which demonstrates how to obtain an access token and a refresh token with a user's credentials in a non interactive way for a native client.

This was great.  Using a saved username and password, I could obtain a refresh token and access token from Azure AD by passing a UserCredentials object to the AcquireToken method in my C# code.

`
var userCredential = new UserCredential(clientUser, clientPassword);
var authenticationResult = authenticationContext.AcquireToken(apiResourceId, clientId,userCredential);`



In the above code example, the authenticationResult will contain both AccessToken and RefreshToken.

By sending these tokens to my SPA I could now generate new access tokens using the refresh token from the Azure AD token endpoint:

`
$.ajax({
url: 'https://login.microsoftonline.com/contoso.onmicrosoft.com/oauth2/token',
type: 'POST',
contentType: 'application/x-www-form-urlencoded',
data: $.param({
grant_type: 'refresh_token',
resource: 'https://contoso.onmicrosoft.com/contosoapi',
refresh_token: '{refresh-token-value}'
}),
success: function(tokenResponse) {
...
},
error: function(tokenResponse) {
...
}});`



During the development process, I realised that it would be handy if I could perform the same AcquireToken method from my javascript code.  In practice it would be wrong for the SPA to store the credentials, however during development it would allow me to not rely on my C# Application to generate a refresh token.

By using fiddler to intercept the call to Azure AD I found that I could call the token endpoint as and specify 'password' as the grant_type as follows.
**Note - I could not find any documentation about this feature**
`
$.ajax({
url: 'https://login.microsoftonline.com/contoso.onmicrosoft.com/oauth2/token',
type: 'POST',
contentType: 'application/x-www-form-urlencoded',
data: $.param({
**grant_type: 'password',**
resource: 'https://contoso.onmicrosoft.com/contosoapi',
client_id: '{client-id-guid e.g 3c4d7df1-79ab-4853-b75d-0136aca5ce55}',
**username: 'contosouser@contoso.onmicrosoft.com',**
** password: '{password-value}'**
}),
success: function(tokenResponse) {
...
},error: function(tokenResponse) {
...
}});`


## Conclusion


This method does not provide any real security to the API since anyone with access to the SPA will obtain a refresh token that they can use to access it.

What it does provide is a way of providing authorization to a Client for a fixed period of time to access a Service through the use of OAuth2 access tokens and Azure AD without providing the Client with a Static Key or Secret.
