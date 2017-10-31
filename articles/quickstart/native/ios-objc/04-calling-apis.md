---
title: Calling APIs
description: This tutorial will show you how to manage tokens to make authenticated API calls, using NSURLSession.
budicon: 546
---

You may want to restrict access to your API resources, so that only authenticated users with sufficient privileges can access them. Auth0 lets you manage access to these resources using [API Authorization](/api-auth).

<%= include('../../../_includes/_package', {
  org: 'auth0-samples',
  repo: 'auth0-ios-objc-sample',
  path: '04-Calling-APIs',
  requirements: [
    'CocoaPods 1.2.1',
    'Version 8.3.2 (8E2002)',
    'iPhone 7 - iOS 10.3 (14E269)'
  ]
}) %>

The reason for implementing authentication, in the first place, is to protect information. In this case, your information is a resource served from a server of any sort. Auth0 provides a squad of tools to assist you with end-to-end authentication in an application. We recommend that you conform to RFC standards by sending valid authentication tokens through an authorization header.

In this tutorial, you'll learn how to get a token, attach it to a request (using the authorization header), and call any API you need to authenticate with.

Before you continue with this tutorial, make sure that you have completed the previous tutorials. This tutorial assumes that:
* You have completed the [Session Handling](/quickstart/native/ios-objc/03-user-sessions) tutorial and you know how to handle the `Credentials` object.
* You have set up a backend application as API. To learn how to do it, follow one of the [backend tutorials](/quickstart/backend).

<%= include('_includes/_calling_api_create_api') %>

<%= include('_includes/_calling_api_create_scope') %>__

## Get the User's Access Token

To retrieve an access token that is authorized to access your API, you need to specify the **API Identifier** value you created in the [Auth0 APIs Dashboard](https://manage.auth0.com/#/apis).

Present the Hosted Login Page:

```objc
// HomeViewController.m

HybridAuth *auth = [[HybridAuth alloc] init];
[auth showLoginWithScope:@"openid profile" connection:nil audience:"API_IDENTIFIER" callback:^(NSError * _Nullable error, A0Credentials * _Nullable credentials) {
    dispatch_async(dispatch_get_main_queue(), ^{
        if (error) {
            NSLog(@"Error: %@", error);
        } else if (credentials) {
          // Do something with credentials e.g.: save them.
          // Auth0 will dismiss itself automatically by default.
        }
    });
}];
```

## Attach the Access Token

To give the authenticated user access to secured resources in your API, include the user's access token in the requests you send to the API.

```objc
// ProfileViewController.m

NSString* token = ... // The accessToken you stored after authentication
NSString *url = @"https://localhost/api"; // Set to your Protected API URL
NSMutableURLRequest *request = [[NSMutableURLRequest alloc] initWithURL:[NSURL URLWithString:url]];
// Configure your request here (method, body, etc)

[request addValue:[NSString stringWithFormat:@"Bearer %@", token] forHTTPHeaderField:@"Authorization"];
[[[NSURLSession sharedSession] dataTaskWithRequest:request completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
    // Parse the response
}] resume];
```

Notice that how you configure your authorization header should match the standards that you're using in your API. This is just an example of what it could look like.


### Sample Project Configuration

When testing the sample project, make sure you configure your URL request in the `ProfileViewController.swift` file:

```objc
// ProfileViewController.m

NSString *url = @"https://localhost/api"; // Change to your API
NSMutableURLRequest *request = [[NSMutableURLRequest alloc] initWithURL:[NSURL URLWithString:url]];
// Configure your request here (method, body, etc)
```

Once you send a request and your API returns a response, its status code is going to be displayed in an alert view.

::: note
For further information on authentication API on the server-side, check [the official documentation](/api/authentication).
:::
