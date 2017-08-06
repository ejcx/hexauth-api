# hexwatch
Welcome to Hexwatch! Get familiar with all things hexwatch.

# Table of Contents
1. [protocol](#protocol)
2. [api](#api)


## handshake
The hexauth handhsake is very similar to the oauth handshake.
### You send a `send` request
You send a request to hexauth, containing a customer email address that you wish to authenticate and a callback URL that will receive the callback code.

### You receive a callback.
You receive an HTTP request when the user successfully authenticates with hexauth. The HTTP request contains a one time use callback code as a get parameter.

### You validate the callback code.
Your service validates the callback code it received. Upon validation, the email address that successfully authenticated is returned to your service.

## api
### authentication
Anyone can fetch an API key from https://hexauth.com. All requests sent to the hexauth api must be authenticated.

All clients must authenticate using [http basic authentication.](https://www.ietf.org/rfc/rfc2617.txt).

### `POST /send`
Send an authentication request to a customer's email address.

```
$ curl 'https://hexauth.com/send' -XPOST \
    -u your@email.com:d34db33fd34db33fd34db \
    -d \
"{
    "ToEmail":"evan@example.com",
    "CallbackURL":"https://example.com/callback",
    "CompanyName":"Hexauth",
}"
```

### `POST /validate/:callback_code`
Validate a customer provided code that you receive from your callback. An `HTTP 200 OK` response is returned if the customer callback code is okay. 
```
$ curl 'https://hexauth.com/validate/d34db3defefa' \
    -u your@email.com:d34db33fd34db33fd34db

{
  "ToEmail": "e@ejj.io",
  "CompanyName": "example company",
  "Verified": true,
  "SentTime": "2017-08-06T18:06:42.615388317Z",
  "VerifiedTime": "2017-08-06T18:13:19.875740558Z"
}
```

If an invalid hexauth callback secret is passed to the validate endpoint, or a callback secret that is for a different customer, an `HTTP 400 Bad Request` is returned.

```
$ curl 'https://hexauth.com/validate/aaaaaaaaa' \
    -u your@email.com:d34db33fd34db33fd34db

{
  "Error": "Invalid magic link",
  "Success": false,
  "Data": null
}
```

### Callback
The callback URL that you pass to the Send request, receives the customer callback secret.

The callback urls hit an unauthenticated API endpoint, that will perform an `HTTP 303 See Other` to your callback url, with a `key` GET parameter set. The `key` parameter contains the  callback secret.
```
$ curl https://hexauth.com/verify/aaaaaaaaaaa -v

<a href="https://example.com/callback?key=08f8f9a0df613ed70bc0431dc7a06f8f3c1669dfbf9eff8f">See Other</a>.
```