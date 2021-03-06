# Webauthn Passwordless js library

This library allows you to fast & without complexity add passwordless sign in (using fido2/webauthn) to your web application.


## Overview

This is what you need to do:

1. **You add our client side library** and call the function `passwordless.register` or `passwordless.signin`
2. **You add two very simple endpoints on your backend** that integrates to your existing user system (*set cookie, sessions, etc*) (and communicates secrets with our API).
3. You make a request between your clientside code and the verification endpoints on your backend to verify the registration or sign in.
 
## Get coding
To get started, add the library to your website (either as ES6 module or global):

Normal script tag:
```html
<script src="https://cdn.passwordless.dev/dist/latest/passwordlessclient.min.js"></script>
```

ES6 module:
```html
<script src="https://cdn.passwordless.dev/dist/latest/passwordlessclient.min.mjs" type="module"></script>
```

UMD module:
```
https://cdn.passwordless.dev/dist/latest/passwordlessclient.umd.min.js
```

NPM package coming soon.

## Get your API Keys

To create a free account, please visit [Create Account](https://beta.passwordless.dev/create-acount) or perform this http call:

```http
POST https://api.passwordless.dev/account/create?accountName=YOUR_ACCOUNT&adminEmail=YOUR_EMAIL@EXAMPLE.COM
```

It will return two keys, one public and one secret. Copy these keys to a secure location as they are only displayed once.

## Register a webauthn credential to user

```html
<script type="module">
    import PasswordlessClient from "https://cdn.jsdelivr.net/gh/passwordless/passwordless-client-js@1.0.1/passwordlessClient.js";
    async function RegisterPasswordless(e) {
        e.preventDefault();

        var p = new PasswordlessClient({
            apiKey: "demo:public:xxx"
        });

        var myToken = await fetch("/example-backend/passwordless/token").then(r => r.text());

        try {
            await p.register(myToken);
        } catch (e) {
            console.error("Things went really bad: ", e);
        }
    }
</script>
```

Notice the `/example-backend/passwordless/token` call?
You need to add one backend endpoint to verify that the user is allowed to register a credential.

On your backend, perform this call:

```http
POST https://api.passwordless.dev/register/token
ApiSecret: demo:secret:yyy
Content-Type: application/json

{ "username": "anders@user.com", "displayName": "Anders" } 
```
It will return the token.

If `await p.register(myToken)` returns sucessfully, the credential has been registered.

## Sign in using webauthn

```html
<script type="module">
    import PasswordlessClient from "https://cdn.jsdelivr.net/gh/passwordless/passwordless-client-js@1.0.1/passwordlessClient.js";
    async function handleSignInSubmit(e) {
        e.preventDefault();

        var p = new PasswordlessClient({
            apiKey: "demo:public:xxx"
        });

        var username = ""; // get username from form

        try {
            var token = await p.signin(username);
            var verifiedUser = await fetch("/example-backend/signin?token=" + token).then(r => r.json());
            console.log("User", verifiedUser);
        } catch (e) {
            console.error("Things went really bad: ", e);
        }
    }
</script>
```
Notice the `/example-backend/passwordless/token/verify` call?
You need to add one backend endpoint to verify the result from the api and set auth cookies.

On your backend, verify the token from signin with this api call:

```http
POST httsp://api.passwordless.dev/signin/verify
ApiSecret: demo:secret:yyy
Content-Type: application/json

{ "token": "zzz" }
```
... where zzz is the token you received from `p.signin(username)`.

The API call will return information about the user sign in:

```json
{
   "success":true,
   "username":"anders@user.com",
   "timestamp":"2020-08-21T16:42:48.5061807Z",
   "rpid":"example.com",
   "origin":"https://example.com"
}
```

## Backend API endpoints

When you've implemented register & sign in, you might want to manage keys or your account which is also done by API.

## List Credentials for user

List all credentials for a certain username

```http
GET /credentials/list?username=USERNAME
ApiSecret: demo:secret:yyy
```

Response 204 No Content or 200 ok:
```
[
    {
        "aaGuid": "00000000-0000-0000-0000-000000000000",
        "credType": "none",
        "descriptor": {
            "id": "rhrZguMM_yiMA-LVquCkUdGELeCoweSdwI_PaU9cq7U",
            "type": "public-key"
        },
        "publicKey": "pAEDAzkBACBZAQDEB/aDgUQ1uHMXNmYYqJAxPJYOx9uS3eC5U1B4zE3PUoED1Z5k2PdFr5huW/KruuwZCY9FYmJf5xUc/z0WUF6ENZL0rzM3aQ+OeYW13lVR0t7tyzLd4ZDOLu4jSdxgqkbxA333lbR4SCiqNQrw5KkB88mqumodWsF/J+1IyY523UR4iR7J/4jLhNTEcmsO8FFc82konW+7U5LpujqMgQBkM+WreclCrm4L5QtIqMabW9KD31FLKwm5OryAmTBWd+XP1nsIae2X6wqVg9HVOGM0hkcu5WphA4/6VZTZM90JWavNPpZHmnnG62UkiXBR45Ncmx1HEKdptT3GwXwwiY9hIUMBAAE=",
        "signatureCounter": 1,
        "userHandle": "aWlp",
        "userId": null,
        "createdAt": "0001-01-01T00:00:00",
        "lastUsedAt": "0001-01-01T00:00:00",
        "RPID": "example.com",
        "Origin": "https://example.com"
    }
]
```

### Delete credentials for user

Delete a certain credential for a user

```http
POST /credentials/delete
ApiSecret: demo:secret:yyy
Content-Type: application/json

{    
    "CredentialId":"vUZudaBNjzJWf-POrGxsso6ztQ3HQ5C6Aef_T-qnKwzEhfqYXQfvuPuPvC09_6IcSfQYdE130s4rU1zqXVlOkw"
}
```

Returns 200 OK

### Delete your account at passwordless.dev

If you want to delete your account and all data stored.

**Please note: This will not delete your data immediately.**
All admin emails connected to the account will receive a warning email with a link to abort the deletion process.
After 24 hours your API keys will be frozen.
After 14 days your data will be permanently deleted.

```
POST /account/delete
ApiSecret: demo:secret:yyy
```


# Build this library

Run `rollup -c`