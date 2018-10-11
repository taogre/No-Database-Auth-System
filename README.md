# No Database Auth System
This is a mechanism to authenticate a user or even create a small account, which is stored on the client side only. It neither requires a password nor any other "external" input to authenticate a user.


The main part of this method is the usage of the `SHA256` algorithm and the local storage API of the browser (`window.localStorage`).


## How it works

The server has to do the following steps to create an account:

1. It has to create a **unique** string, which gets **salted** and **hashed**. The result of which is the `key`. You could imagine it as something like a username. It is very important that the string you start with is unique. Otherwise somebody might get access to another account. Another very important thing is to have a **secret salt**.

2. In order to prevent somebody from just bruteforceing the key the method requires another parameter, which I call the `authToken`. As you can imagine it is like a password. The way it is created is close to the creation of the `key`. Take the `key`, you've just created in the first step, **salt** it (**Recommended: Use two different salts**) and voilà you've created the `key` and `authToken`.

The client has to receive the `key` and the `authToken` and save it. (By using the local storage API on the browser)


How to verify the account:

1. The client has to send the created `key` and the `authToken` to the server.

2. What the server does to verify it is very simple: Compare the `authToken` to the salted and hashed `key`. If they are equal, the user passes the test.

### Now in code:

Creating an account on the server (in this case NodeJS):
```javascript

//Define the SECRET salts:
const KeySalt = "THIS IS YOUR VERY SECRET SALT FOR THE KEY"
const AuthSalt = "THIS IS YOUR VERY SECRET SALT FOR THE AUTHTOKEN"

//This function returns the final key and authToken
function createAccount(){
  //Create key
  const key = sha256(getUniqueString() + KeySalt)
  //Return and create authToken
  return {
    key,
    authToken: sha256(key + AuthSalt)
  }
}

//The output of this function will ALWAYS change.
//Because of this function the string is truly unique and there won't be any "username collisions"
function getUniqueString(){
  //This returns: "[timeinmilliseconds] - [nanoseconds]"
  return `${(new Date()).getTime()} - ${process.hrtime()[1]}`
}
```

Verifying an account on the server (also NodeJS):
```javascript

//Validation function with key and authToken as parameters (both of type string)
function validateAccount(key, authToken){
  return authToken === sha256(key + AuthSalt)
}

```
