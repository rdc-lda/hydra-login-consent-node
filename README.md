# hydra-user-and-consent-provider-node

This is a reference implementation for the User Login and Consent flow of ORY Hydra version 1.0.x in NodeJS. The
application is bootstrapped using the `express` cli.

---

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Overview](#overview)
- [Running locally](#running-locally)
  - [Using a locally available binary](#using-a-locally-available-binary)
  - [Using Docker](#using-docker)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

---

## Overview

Apart from additions (`./routes/login.js`, `./routes/consent.js`) and their respective templates, only a [CSRF Middleware]
has been added. Everything else is the standard express template.

Also, a simple helper that makes HTTP requests has been added to `./services/hydra.js` which uses the `node-fetch`
library.

To set this example up with ORY Hydra, please refer to the [official documentation](https://www.ory.sh/docs).

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See deployment for notes on how to deploy the project on a live system.

### Prerequisites

In order to develop / test, you need the following tools installed:

* [Docker](https://docs.docker.com/docker-for-mac/install/)
* [GNU Make](http://osxdaily.com/2014/02/12/install-command-line-tools-mac-os-x/) 3.8 or higher

## Built With

* Docker

### Build Docker

~~~bash
# Build the Docker locally
#  -- see output how to run this Docker on your machine
$ make image

# Push Docker to the Docker Registry
$ make push
~~~

## Running locally

To run this example locally, you will first want to start ORY Hydra. Please note, that the set up shown here might not
work in the future as it may become out of date. For now, this should work however. We are also assuming that you are
using ORY Hydra >= 1.0.0.

### Using Docker

If you have ORY Hydra 1.0.0 not installed locally, you can use Docker to run the following commands. First, ensure
that you have the latest version available from Docker Hub:

```
$ docker pull oryd/hydra:v1.0.0-beta.9
```

Then, start the server:

```
$ docker run -it --rm --name login-consent-hydra -p 4444:4444 -p 4445:4445 \
    -e OAUTH2_SHARE_ERROR_DEBUG=1 \
    -e LOG_LEVEL=debug \
    -e OAUTH2_CONSENT_URL=http://localhost:3000/consent \
    -e OAUTH2_LOGIN_URL=http://localhost:3000/login \
    -e OAUTH2_ISSUER_URL=http://localhost:4444 \
    -e DATABASE_URL=memory \
    oryd/hydra:v1.0.0-beta.9 serve all --dangerous-force-http
```

Next, you will need to create a new client that we can use to perform the OAuth 2.0 Authorization Code Flow:

```
$ docker run --link login-consent-hydra:hydra oryd/hydra:v1.0.0-beta.9 clients create \
    --endpoint http://hydra:4445 \
    --id test-client \
    --secret test-secret \
    --response-types code,id_token \
    --grant-types refresh_token,authorization_code \
    --scope openid,offline \
    --callbacks http://127.0.0.1:4446/callback
```

Now, run this project

```
$ npm i
$ HYDRA_ADMIN_URL=http://localhost:4445 npm start
```

And finally, initiate the OAuth 2.0 Authorization Code Flow (you need to manually open the presented URL):

```
$ docker run -p 4446:4446 --link login-consent-hydra:hydra oryd/hydra:v1.0.0-beta.9 token user \
    --token-url http://hydra:4444/oauth2/token \
    --auth-url http://localhost:4444/oauth2/auth \
    --scope openid,offline \
    --client-id test-client \
    --client-secret test-secret
```

### Using a locally available binary

If you have ORY Hydra 1.0.0 installed locally, run the following commands. First, start the server:

```
$ OAUTH2_CONSENT_URL=http://localhost:3000/consent \
    OAUTH2_LOGIN_URL=http://localhost:3000/login \
    OAUTH2_ISSUER_URL=http://localhost:4444 \
    OAUTH2_SHARE_ERROR_DEBUG=1 \
    LOG_LEVEL=debug \
    DATABASE_URL=memory \
    hydra serve all --dangerous-force-http
```

Next, you will need to create a new client that we can use to perform the OAuth 2.0 Authorization Code Flow:

```
$ hydra clients create \
    --endpoint http://localhost:4445 \
    --id test-client \
    --secret test-secret \
    --response-types code,id_token \
    --grant-types refresh_token,authorization_code \
    --scope openid,offline \
    --callbacks http://127.0.0.1:4446/callback
```

Now, run this project

```
$ npm i
$ HYDRA_ADMIN_URL=http://localhost:4445 npm start
```

And finally, initiate the OAuth 2.0 Authorization Code Flow:

```
$ hydra token user \
    --token-url http://localhost:4444/oauth2/token \
    --auth-url localhost:4444/oauth2/auth \
    --scope openid,offline \
    --client-id test-client \
    --client-secret test-secret
```

## FAQ

### TLS Termination

You can mock TLS Termination by setting environment variable `MOCK_TLS_TERMINATION` to any value, for example `MOCK_TLS_TERMINATION=y`.
This will add `X-Forwarded-Proto: https` to each HTTP Request Header.
