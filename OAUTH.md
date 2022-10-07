# spi-oauth
Service provider integration OAuth2 microservice.

### About

OAuth2 protocol is the most commonly used way that allows users to authorize applications to communicate with service providers.
`spi-oauth` to use this protocol to obtain service provider’s access tokens without the need for the user to provide us his login credentials.


This OAuth2 microservice would be responsible for:
 - Initial redirection to the service provider
 - Callback from the service provider
 - Persistence of access token that was received from  the service provider into the permanent backend (k8s secrets or Vault)
 - Handling of negative authorization and error codes
 - Creation or update of SPIAccessToken
 - Successful redirection at the end

### How to build
 `make docker-build docker-push`
  Available paramters
  - `SPIS_IMAGE_TAG_BASE` - the name of the image. Example `quay.io/skabashn/service-provider-integration-oauth`.
  - `SPIS_TAG_NAME` - the tag of the image. Example `$(git branch --show-current)'_'$(date '+%Y_%m_%d_%H_%M_%S')`.
### How to run
The easiest way to run the SPI OAuth service is to deploy it together with the SPI
operator.

Check out the [SPI operator repository](https://github.com/redhat-appstudio/service-provider-integration-operator)
and run:
```
make install deploy SPIS_IMG=<...the image of the SPI OAuth service...>
```

replace the `deploy` target above with the specialization required for your target
cluster, e.g. use `deploy_minikube` when deploying to Minikube.

### HTTP API Endpoints

The OAuth service exposes 3 kinds of endpoints:

* `/<service_provider>/authenticate` (e.g. `/github/authenticate`) - the endpoint for initiating the OAuth flow with
  given service provider. This endpoint accepts either `GET` or `POST` request with the following attributes:
  * `k8s_token` - the token used to authenticate with the configured Kubernetes API server. This token
    must represent a user that is able to create `SPIAccessTokenDataUpdate` objects in the namespace for which
    the OAuth flow is being initiated.
  * `state` - the OAuth state as generated by the SPI operator
  
  **Note** that this endpoint sets a session cookie that must be available when the `callback` endpoint is called 
* `/<service_provider>/callback` (e.g. `/github/callback`) - the endpoint to finish the OAuth flow to which
  the service provider redirects back.
* `/token/<namespace>/<spiaccesstoken_name>` - the endpoint using which one can manually upload the token data for given
  `SPIAccessToken` object.
  
  This POST endpoint accepts JSON object with the following structure:
  ```javascript
  {
    "access_token": "string value of the access token",
    "token_type": "the type of the token", // currently ignored
    "refresh_token": "string value of the refresh token", // currently ignored
    "expiry": 42 // the date when the token expires represented as timestamp, currently ignored 
  }
  ```