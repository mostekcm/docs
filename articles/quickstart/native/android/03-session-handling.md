---
title: Session Handling
description: This tutorial will show you how to login and maintain a session’s connectivity.
seo_alias: android
budicon: 280
---

This tutorial shows you how to let users log in and maintain an active session with Auth0.

<%= include('../../../_includes/_package', {
  org: 'auth0-samples',
  repo: 'auth0-android-sample',
  path: '03-Session-Handling',
  requirements: [
    'Android Studio 2.3',
    'Android SDK 25',
    'Emulator - Nexus 5X - Android 6.0'
  ]
}) %>__

You need the `Credentials` class to handle users' credentials. The class is composed of these elements:

* `accessToken`: Access token used by the Auth0 API. To learn more, see the [access token documentation](/tokens/access-token).
* `idToken`: Identity token that proves the identity of the user. To learn more, see the [ID token documentation](/tokens/id-token).
* `refreshToken`: Refresh token that can be used to request new tokens without signing in again. To learn more, see the [refresh token documentation](/tokens/refresh-token/current).
* `tokenType`: The type of tokens issued by the server.
* `expiresIn`: The number of seconds before the tokens expire.
* `expiresAt`: The date when the tokens expire.
* `scope`: The scope that was granted to a user. This information is shown only if the granted scope is different than the requested one.

Tokens are objects used to prove your identity against the Auth0 APIs. Read more about them in the [tokens documentation](https://auth0.com/docs/tokens).

## Before You Start

::: note
Before you continue with this tutorial, make sure that you have completed the [Login](/quickstart/native/android/00-login) tutorial.
:::

Before you launch the login process, make sure you get a valid refresh token in the response. To do that, ask for the `offline_access` scope. Find the snippet in which you are initializing the `WebAuthProvider` class. To that snippet, add the line `withScope("openid offline_access")`.

```java
// app/src/main/java/com/auth0/samples/LoginActivity.java

Auth0 auth0 = new Auth0(this);
auth0.setOIDCConformant(true);
WebAuthProvider.init(auth0)
                .withScheme("demo")
                .withAudience(String.format("https://%s/userinfo", getString(R.string.com_auth0_domain)))
                .withScope("openid offline_access")
                .start(LoginActivity.this, callback);
```

## Check for Tokens when the Application Starts

::: panel Learn about refresh tokens
Before you go further with this tutorial, read the [refresh token documentation](/refresh-token).
It is important that you remember the following:
* Refresh tokens must be securely saved.
* Even though refresh tokens cannot expire, they can be revoked.
* New tokens will never have a different scope than the scope you requested during the first login.
:::

The library includes a Credentials Manager class that will handle the credentials storage for you. The manager also checks if the stored credentials are still valid, and renew them automatically after they expire. For this reason, it requires an `AuthenticationAPIClient` instance upon instantiation.

Create a new instance of the manager and use it to check if credentials were previously saved:

```java
// app/src/main/java/com/auth0/samples/LoginActivity.java
  Auth0 auth0 = new Auth0(this);
  auth0.setOIDCConformant(true);
  CredentialsManager credentialsManager = new CredentialsManager(new AuthenticationAPIClient(auth0), new SharedPreferencesStorage(this));

  if (credentialsManager.hasValidCredentials()) {
    // Try to make an automatic login
  } else {
    // Prompt Login screen.
  }

```

::: note
For details, check the dedicated [Credentials Manager article](/libraries/auth0-android/save-and-refresh-tokens.md).
:::


## Save the User's Credentials

Save the user's credentials obtained in the login success response. Using the manager, call the `saveCredentials` method.

```java
// app/src/main/java/com/auth0/samples/LoginActivity.java

private final AuthCallback callback = new AuthCallback() {
    @Override
    public void onFailure(@NonNull final Dialog dialog) {
        //show error dialog
    }

    @Override
    public void onFailure(AuthenticationException exception) {
        //show error message
    }

    @Override
    public void onSuccess(@NonNull Credentials credentials) {
        credentialsManager.saveCredentials(credentials);
        //...
    }
};
```

::: note
The Storage implementation given to the Credentials Manager in the seed project uses a SharedPreference file to store the user credentials in [Private mode](https://developer.android.com/reference/android/content/Context.html#MODE_PRIVATE). Change this behavior by implementing a custom Storage.
:::

## Recover the User's Credentials

Retrieving the credentials from the manager is an async process, as credentials may have expired and require to be refreshed. This credentials renewing process is done automatically as long as a valid refresh token is currently stored. A `CredentialsManagerException` exception will raise if the credentials have expired and can't be refreshed.

```java
// app/src/main/java/com/auth0/samples/MainActivity.java

credentialsManager.getCredentials(new BaseCallback<Credentials, CredentialsManagerException>() {
    @Override
    public void onSuccess(Credentials credentials) {
        // use credentials
    }

    @Override
    public void onFailure(CredentialsManagerException error) {
        // Credentials could not be refreshed. Log in again
    }
});
```

## Log the User Out

To log the user out, you must remove their credentials and navigate them to the login screen.

When using a Credentials Manager, you do that by calling `clearCredentials`.

```java
// app/src/main/java/com/auth0/samples/MainActivity.java

private void logout() {
  credentialsManager.clearCredentials();
  startActivity(new Intent(this, MainActivity.class));
  finish();
}
```

::: note
Depending on the way you store users' credentials, you delete them differently.
:::
