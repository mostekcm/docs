---
title: User Profile
description: This tutorial will show you how to get and modify the user's profile data.
seo_alias: android
budicon: 292
---

This tutorial shows you how to get and modify the user's profile data with Auth0 in your Android apps.

<%= include('../../../_includes/_package', {
  org: 'auth0-samples',
  repo: 'auth0-android-sample',
  path: '04-User-Profile',
  requirements: [
    'Android Studio 2.3',
    'Android SDK 25',
    'Emulator - Nexus 5X - Android 6.0'
  ]
}) %>__

## Before You Start

::: note
Before you continue with this tutorial, make sure that you have completed the [Login](/quickstart/native/android/00-login) and [Session Handling](/quickstart/native/android/03-session-handling) tutorials. To call the API clients, you need a valid access token and ID token.
:::

Before launching the login process, you need to make sure you get a valid profile from the authorization server. To do that, ask for the `openid profile email` scope. Find the snippet in which you initialize the `WebAuthProvider` class. To that snippet, add the line `withScope("openid profile email")`.

```java
Auth0 auth0 = new Auth0(this);
auth0.setOIDCConformant(true);
WebAuthProvider.init(auth0)
                .withScheme("demo")
                .withScope("openid profile email")
                .start(this, callback);
```

## Request User Data


To get the user's information:
1. Use the user's access token to call the `userInfo` method in the `AuthenticationAPIClient` client instance.
The profile obtained this way is OIDC-conformant. Depending on the [scopes](/scopes/current) you requested when logging in, the profile contains different information. The result will never contain fields outside the OIDC specification.
2. Get the user's full profile using the [Management API](/api/management/v2#!/Users). This step is explained next:


Create an instance of the Users API client using the Id token obtained in the log in step. You will use this client to request the users' profile data.

```java
// app/src/main/java/com/auth0/samples/activities/MainActivity.java

Auth0 auth0 = new Auth0(this);
auth0.setOIDCConformant(true);

CredentialsManager credentialsManager = new CredentialsManager(new AuthenticationAPIClient(auth0), new SharedPreferencesStorage(this));
credentialsManager.getCredentials(new BaseCallback<Credentials, CredentialsManagerException>() {

    @Override
    public void onSuccess(Credentials credentials) {
        String idToken = credentials.getIdToken();
        UsersAPIClient usersClient = new UsersAPIClient(auth0, idToken);
        //...
    }

    @Override
    public void onFailure(CredentialsManagerException error) {
        //Credentials expired. Log in again
    }
});
```

::: note
Do not hardcode the Auth0 `domain` and `clientId` values when creating the Auth0 instance. We recommend you add them to the `strings.xml` file.
:::


Decode the Id Token using a [JWT library](https://github.com/auth0/JWTDecode.Android) and obtain the Subject value, which represents the id of the user to whom the token was issued.

```java
// app/src/main/java/com/auth0/samples/activities/MainActivity.java

private String getUserId(String idToken) {
    return new JWT(idToken).getSubject();
}
```

Use the user id to call `getProfile` on the Users API client in order to obtain the full user profile.

```java
// app/src/main/java/com/auth0/samples/activities/MainActivity.java

usersClient.getProfile(getUserId(idToken))
  .start(new BaseCallback<UserProfile, ManagementException>() {
      @Override
      public void onSuccess(UserProfile profile) {
          userProfile = profile;
          runOnUiThread(new Runnable() {
              public void run() {
                  refreshScreenInformation();
              }
          });
      }

      @Override
      public void onFailure(ManagementException error) {
          runOnUiThread(new Runnable() {
              public void run() {
                  Toast.makeText(MainActivity.this, "User Profile Request Failed", Toast.LENGTH_SHORT).show();
              }
          });
      }
  });
```


When you get the `sub` value, call the [Management API](https://auth0.com/docs/api/management/v2#!/Users).

Use the `UsersAPIClient` client and the user's ID to get the full user profile.

```java
// app/src/main/java/com/auth0/samples/activities/MainActivity.java

usersClient.getProfile(userId)
        .start(new BaseCallback<UserProfile, ManagementException>() {
            @Override
            public void onSuccess(UserProfile profile) {
                // Display the user profile
            }

            @Override
            public void onFailure(ManagementException error) {
                //show error
            }
        });
```

## Access the Data Inside the Received Profile

### Default information

At this point, you already have access to the `UserProfile` instance.
You can use this data wherever you need it.

Some examples are:

```java
profile.getName();
profile.getEmail();
profile.getPictureURL();
```

::: panel Modifying the UI
You cannot modify the UI inside the `onSuccess()` method because the method works in a second thread. To solve this issue, you can choose between three options:
* Persist the data
* Create a task in the UI thread
* Create a handler to receive the information
:::

### Additional information

You can handle additional information within any of the following maps:

#### A. User metadata

The `userMetadata` map contains user profile fields that can be added from the client-side (e.g. when a user edits their profile). You can access this information in the following way:

```java
String country = (String) profile.getUserMetadata().get("country");
```

::: note
You can choose the key names and value types for subscripting the `user_metadata` map.
:::

#### B. App metadata

The `appMetadata` map contains fields that are usually added with a [Rule](/rule) or a [Hook](/hooks). For native platforms, this information is read-only.

::: note
To learn more about metadata, see the [metadata documentation](/metadata).
:::

#### C. Extra information

The `extraInfo` map contains additional values that are stored in Auth0 but not mapped to a `UserProfile` getter method. For native platforms, this information is read-only.

## Update the User's Profile

You can only update the user metadata. To do this, create a `Map<String, Object>` object and add the new metadata:

```java
Map<String, Object> userMetadata = new HashMap<>();
userMetadata.put("country", "USA");
```

Update the information with the User API client:

```java
// app/src/main/java/com/auth0/samples/activities/MainActivity.java

String idToken = CredentialsManager.getCredentials(this).getIdToken();
UsersAPIClient usersClient = new UsersAPIClient(auth0, idToken);
usersClient.updateMetadata(userInfo.getId(), userMetadata).start(new BaseCallback<UserProfile, ManagementException>() {
  @Override
  public void onSuccess(final UserProfile profile) {
    // As receive the updated profile here
    // You can react to this, and show the information to the user.
  }

  @Override
  public void onFailure(ManagementException error) {
    //show error
  }
});
```
