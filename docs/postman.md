# Using Postman to test RingCX APIs

RingCentral provides a Postman Collection generated from the public RingCX OpenAPI specification. The collection includes the public API operations, JWT-based authentication requests, account identifier discovery, reusable environment variables, and available response examples.

The files are available here:

* [Postman Collection v2.1](https://raw.githubusercontent.com/ringcentral/engage-voice-api-docs/main/specs/engage-voice_postman2.json)
* [Postman Environment template](https://raw.githubusercontent.com/ringcentral/engage-voice-api-docs/main/specs/ringcx_postman_environment.json)
* [OpenAPI 3.0 Specification](https://raw.githubusercontent.com/ringcentral/engage-voice-api-docs/main/specs/engage-voice_openapi3.json)

## Prerequisites

Before using the collection, you need:

* A production RingCentral account linked to your RingCX account.
* A RingCentral application configured for JWT authentication.
* The application's client ID and client secret.
* A JWT credential for a user who has access to the required RingCX accounts and APIs.

See [Authenticating with RingCentral](authentication/auth-ringcentral.md) for application setup and authentication details.

!!! primary "Legacy authentication"
    The collection's setup workflow uses the current RingCentral JWT and RingCX token exchange flow. The APIs under **Authentication > Legacy Auth** remain available for integrations that require a supported legacy authentication method.

## Import the collection and environment

1. In Postman, select **Import**.
2. Import the [Postman Collection v2.1](https://raw.githubusercontent.com/ringcentral/engage-voice-api-docs/main/specs/engage-voice_postman2.json) from its URL.
3. Import the [Postman Environment template](https://raw.githubusercontent.com/ringcentral/engage-voice-api-docs/main/specs/ringcx_postman_environment.json) from its URL.
4. Select **RingCX Voice API Environment** as the active environment.

The environment template does not contain credentials, tokens, or account identifiers.

## Configure the environment

Set the following environment values:

| Variable | Description |
| --- | --- |
| `RINGCENTRAL_CLIENT_ID` | Client ID for the RingCentral application. |
| `RINGCENTRAL_CLIENT_SECRET` | Client secret for the RingCentral application. |
| `RINGCENTRAL_JWT` | JWT credential for the RingCentral user. |

The base URLs are preconfigured. The authentication and setup requests populate these values:

| Variable | Description |
| --- | --- |
| `rco_access_token` | RingCentral access token used for the RingCX token exchange and RingCentral account lookup. |
| `ringcx_bearer_token` | RingCX access token used by the API requests. |
| `rcx_sub_account_id` | RingCX sub-account ID. |
| `rcx_main_account_id` | RingCX main account ID. |
| `rc_account_uid` | RingCentral account UID used by CX integration APIs. |

## Authenticate and discover account IDs

Open **Auth and setup** and run the requests in order:

1. **Get RingCentral access token** authenticates with the configured JWT.
2. **Exchange for RingCX access token** obtains the RingCX token used by the collection.
3. **Get RingCX account IDs** stores the first available RingCX sub-account and main account IDs.
4. **Get RingCentral account UID** stores the RingCentral account UID used by CX integration APIs.

The RingCX token exchange is limited to five requests per minute. RingCX access tokens are valid for five minutes and are not refreshed automatically. If an API request returns `401 Unauthorized`, run the first two authentication requests again and retry the API request.

If the accounts response contains multiple RingCX sub-accounts, review the response and set `rcx_sub_account_id` and `rcx_main_account_id` to the account your integration will use.

## Send an API request

API requests are grouped by the same product areas and API tags used in the API Reference. Select an individual request, provide any remaining path variables or request-body values, and select **Send**.

!!! warning
    Do not use Postman's **Run collection** command against a production account. The collection includes operations that create, update, log out, and delete RingCX resources.

Optional query parameters are included but disabled by default. Enable only the parameters required by your request.

## Feedback

If you have feedback about the collection, [open an issue in the RingCX documentation repository](https://github.com/ringcentral/engage-voice-api-docs/issues).
