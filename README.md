<p align="center">
  <a href="https://helix.eleclink.co.uk">
    <img src='public/logo-dark.svg' width="250" alt="Helix">
  </a>
</p>

<p align="center">
  <i>
    OpenAPI specification for the <strong>Platform API</strong> of <strong>Helix</strong>, the nomination platform of ElecLink
  </i>
</p>

<p align="center">
  <a href="https://github.com/eleclink-helix/platform-api/tags"><img alt="GitHub - Version" src="https://img.shields.io/github/v/tag/eleclink-helix/platform-api?logo=github&label=version"></a>
  <a href="https://pypi.org/project/helix-platform-api-public-client/"><img alt="PyPI - Version" src="https://img.shields.io/pypi/v/helix-platform-api-public-client?logo=pypi"></a>
</p>

---

The **Platform API** is a REST-style HTTP API operating mostly with JSON payloads. The [OpenAPI](https://www.openapis.org/) specification contains every endpoint that is available for Participants to consume. Endpoints are identified and referred to by their _Operation ID_ e.g. `getNominations` throughout the documentation, the terms "endpoint" and "operation" used interchangeably.

**Helix** is being developed in an _API-first approach_. The [semantically versioned](https://semver.org/) specification is used internally to generate code in order to make sure that the API contract is matched with the backend implementation. Consumers of the API are also encouraged to utilise code generation to achieve consistent communication using each version of the contract.

**Resouces**:

* [🧭 **API Navigator**](https://eleclink-helix.github.io/platform-api) - the recommended way of exploring the API
* [📚 **openapi.yaml**](openapi.yaml) - the latest specification in _OpenAPI 3.0.3_ format
* [👩‍🏫 **Wiki**](https://github.com/eleclink-helix/platform-api/wiki) - demonstration of various API use-cases to ease the integration
  - with supporting business logic including [Nominations](https://github.com/eleclink-helix/platform-api/wiki/Nominations), etc

## Table of Contents

* [🏃 Getting Started](#-getting-started)
* [🔌 Environments](#-environments)
* [🎓 General Concepts](#-general-concepts)
  - [❓ What is _Participant ID_](#-what-is-participant-id)
  - [👥 Roles and Permissions](#-roles-and-permissions)
  - [⛔ Errors and Validations](#-errors-and-validations)
  - [📊 Data Formats](#-data-formats)
* [🐍 Python Client SDK](#-python-client-sdk)
* [📖 Changelog](#-changelog)

## 🏃 Getting Started

In order to programmatically use the **Platform API** Participants must have a valid user account and need to obtain an _API Key_. Follow these steps to generate one:

1. First visit an available [environment](#-environments) of **Helix** through its _Web Frontend URL_
2. Click **Login** and use your your email address, password and 2FA token to authenticate
3. When logged in click on **your name** in the top-right corner 
4. Select **API Keys** from the pop-up menu
5. Click on the **New API Key** button
6. Choose a unique name to your _API Key_ and click the **Generate API Key** button
8. Copy and save your _API Key_, the string starting with `hel_`

As described in the `AuthToken` [security scheme](https://swagger.io/docs/specification/authentication/), the _API Key_-based authentication is using the `X-API-Key` request header.

In order to test the newly generated _API Key_ you can try to execute the `getProfile` operation by replacing `<API_BASE_URL>` and `<API_KEY>` with the respective values and should receive a `200 OK` HTTP response.

```shell
curl -X 'GET' \
  '<API_BASE_URL>/v1/profile' \
  -H 'accept: application/json' \
  -H 'X-API-Key: <API_KEY>'
```

> ⚠️ Please always make sure you test your  _API Key_ on the environment that you generated your key on as _API Keys_ are **NOT** shared between the different [environments](#-environments)!

> 💡 Users that are not registered with EleclLink are able to use the endpoints prefixed with `/v1/public/...` for fetching publicly available data from **Helix**. See the [**API Navigator**](https://eleclink-helix.github.io/platform-api) for more information. Please also refer to the relevant [Public Data](https://github.com/eleclink-helix/platform-api/wiki/Public-Data) page on the Wiki for more information.

## 🔌 Environments

**Helix** offers two environments for the Participants to use:

| Environment    | Web Frontend URL                      | API Base URL                                |
| -------------- | ------------------------------------- | ------------------------------------------- |
| **PRODUCTION** | https://helix.eleclink.co.uk          | `https://api.helix.eleclink.co.uk`          |
| **TRAINING**   | https://training.helix.eleclink.co.uk | `https://api.training.helix.eleclink.co.uk` |

The **PRODUCTION** environment means the all-time _live deployment_ of **Helix**. Interacting with it through the API will result in submitting "real" nominations, etc with all their consequences.

The **TRAINING** environment however can be used to onboard new members of your Organisations and test your API integrations in a testing environment. Changes in the **Platform API** (bug fixes, new features, breaking changes) are also made available on the **TRAINING** environment in a pre-determined schedule.

> 💡 You can use the [🧭 **API Navigator**](https://eleclink-helix.github.io/platform-api) to determine which version of the **Platform API** is currently deployed on each environment. Just open the version selector dropdown in the top-right corner of the page.

## 🎓 General Concepts

### ❓ What is _Participant ID_

Many endpoints require submitting a so-called _Participant ID_ in its parameters, for example `getParticipantOverview`, `submitNominations`, `getDefeaultNominations` etc.

**Helix** handles different types of _Organisations_ in the system, and one of these types are _Participants_. **A _Participant ID_ is essentially the ID of a Participant-type Organisation**. To determine your _Participant ID_ you can use the `getProfile` operation:

```bash
curl -X 'GET' \
  '<API_BASE_URL>/v1/profile' \
  -H 'accept: application/json' \
  -H 'X-API-Key: <API_KEY>'
```

which will return the following data structure:

```json
{
  "user": {
    "id": "64f8c88a-6fbd-4fda-8bda-d579f87b13d3",
    "name": "Example User",
    "email": "email@example.com",
    "role": "PART_ADMIN",
    "organisation": {
      "id": "5c6a088a-93b6-433c-8538-b7488207df39",     <<< Participant ID
      "name": "Test Participant",
      "authMethod": "PLATFORM"
    }
  },
  "permissions": [
    "MANAGE_OWN_DEFAULT_NOMINATIONS",
    "MANAGE_OWN_MANUAL_FILE_UPLOAD",
    "MANAGE_OWN_NOMINATIONS",
    ...
  ],
  "appearance": "DEFAULT",
  "impersonatedBy": null
}
```

As highlighted in the JSON response, the _Participant ID_ is the value in the `user.organisation.id` field - the ID of the _Organisation_ that the owner of the _API Key_ belongs to.

The _Partcipant ID_ can be treated as a **constant** in your integrations, it won't ever change in **Helix**.

> 💡 To ease the testing process **_Participant IDs_ are kept in sync between [environments](#-environments)**, which is generally **NOT** true for other IDs (e.g. User IDs, Default Nomination IDs, etc), those are all unique and different on each environment.

### 👥 Roles and Permissions

In **Helix** each individual User is assigned a _Role_ which encapsulates a set of _Permissions_. This information can be retrieved using the `getProfile` operation as seen previously in the [What is _Participant ID_](#-what-is-participant-id) section (fields `user.role` and `permissions[]`).

In the API each endpoint is annotated with a list of _Permissions_ documented in the `x-permissions` field, the values being validated against the `#/components/schemas/Permission` enum:

```yaml
  /v1/attachments:
    post:
      operationId: uploadAttachment
      description: |
        ...
      tags:
        - attachment
      security:
        - ApiKey: []
        - AuthToken: []
      x-permissions:
        - MANAGE_CRISIS_ACTIONS
        - INVITE_OWN_USERS
        - INVITE_ANY_USERS
        - MANAGE_ANY_MANUAL_FILE_UPLOAD
        - MANAGE_OWN_MANUAL_FILE_UPLOAD
      requestBody:
        ...
```

The first line of authorization is implemented based on these attached _Permissions_. In the example above the `uploadAttachment` operation can be executed if **at least one** of the listed five _Permissions_ is granted to the User. If none of the listed _Permissions_ is granted, the endpoint will return a `HTTP 403` status with the `FORBIDDEN` [error code](#-errors-and-validations).

Naturally additional authorization logic can be added to individual operations. Take the `getDefaultBids` operation as an example:

```yaml
  /v1/default-nominations:
    get:
      operationId: getDefaultNominations
      description: |
        ...
      tags:
        - default-nomination
      security:
        - ApiKey: []
        - AuthToken: []
      x-permissions:
        - VIEW_OWN_DEFAULT_NOMINATIONS
        - VIEW_ANY_DEFAULT_NOMINATIONS
      parameters:
        ...
        - $ref: '#/components/parameters/ParticipantId'
        ...
      responses:
        ...
```

In this case the `getDefaultNominations` operation can be called with either `VIEW_OWN_DEFAULT_NOMINATIONS` or `VIEW_ANY_DEFAULT_NOMINATIONS`. However if the User has the `VIEW_OWN_DEFAULT_NOMINATIONS` _Permission_, they  **MUST** send their own [_Participant ID_](#-what-is-participant-id) in the request. Failing to do so will also result in a `403 FORBIDDEN` error response from the API.

### ⛔ Errors and Validations

Each endpoint in the **Platform API** can be a subject of a set of _Validations_, like validating User authentication or authorization, or applying different complex business on validations on requests.

When any of these validations happens to fail, **Helix** will respond with a semantically correct _HTTP Status_ (e.g `HTTP 404` for non-existing entities in the system) and an _Error Code_ to be able to tell apart the different errors that could have happened during execution.

> 💡 With `PUT/POST/DELETE`-type operations (e.g Nomination submission, etc) when no errors were detected and there is no point in returning any response body, **Helix** generally returns the _"204 No Content"_ response status to act as a **positive acknowledgement**.

Errors are documented for the different status codes under the `x-errors` field. The values of `x-errors[*].code` is validated against the `#/components/schemas/ErrorCode` schema.

Below is a full example taken from the `submitNominations` opreation:

```yaml
  /v1/nominations:
      operationId: submitNominations
      ...
      responses:
        '204':
          $ref: '#/components/responses/NoContent'
        '400':
          ...
          x-errors:
            - code: INVALID_PARTICIPANT_ID
              description: specified Participant not found in the system
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '403':
          $ref: '#/components/responses/Forbidden'
        '422':
          ...
          x-errors:
            - code: MTU_LIST_MISALIGNED
              description: |
                - for the values to be sent
                - one or more MTU start times are not aligned to MTU borders (considering MTU size)
                - the number of MTUs defined does not match the required number (pay special attention to daylight saving changes!)
            - code: MTU_NOT_EDITABLE
              description: |
                - only MTUs that are in `mtuStatus = EDITABLE` can contain not-empty values
                - an MTU can be non-editable because:
                  - the MTU nomination window for the selected timescale is closed
                  - has already FIRM nominations
            - code: INVALID_NOMINATION_VALUE
              description: |
                - nomination values can be empty for any MTU, where not empty has to be (0 <= value <= Total TRs)
            ...
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
```

There are several standard _HTTP Status_ and _Error Code_ combinations which are always used in the following pairs:

| HTTP Status        | Error Code     | Description                                                                     |
|--------------------|----------------|---------------------------------------------------------------------------------|
| _401 Unauthorized_ | `UNAUTHORIZED` | failed to authenticate, e.g problem with [_API Key_](#-getting-started)         |
| _403 Forbidden_    | `FORBIDDEN`    | authorization failed, e.g insufficient [_Permissions_](#-roles-and-permissions) |
| _404 Not Found_    | `NOT_FOUND`    | entity with the given ID is not found in the system                             |

On top of the above **Helix** generally utilises three other _HTTP Statuses_ to handle custom business validations:

| HTTP Status                   | Error Code   | Description | Example |
|-------------------------------|--------------|-------------|---------|
| _400 Bad Request_             | vary         | request is "conceptually" wrong | tried to "suspend" a user already in `SUSPENDED` state in the `suspendUser` operation |
| _422 Unprocessable Entity_    | vary         | some value in request failed validations | the User want to nominate more Capacity than the amount of TRs they have in `submitNominations` |

### 📊 Data Formats

**Date/Time handling**

The _System Time_ of **Helix** is `Europe/Paris`. The "delivery days" are interpreted in this time, following the regular [DST](https://en.wikipedia.org/wiki/Daylight_saving_time) changes between the CET and CEST timezones.

Absolute points in time are represented with the `#/components/schemas/DateTime` schema in the specification. These values are always returned and required to be sent in [UTC](https://en.wikipedia.org/wiki/Coordinated_Universal_Time) (a.k.a "Zulu time") in [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) format e.g `2023‐08‐24T15:17:54.123Z`.

**Capacity values**

Capacity values throughout the **Platform API** are represented with the `#/components/schemas/Capacity` schema. It defines that capacity values are to be sent and will be received as **integers** interpreted in **kilowatt (kW)** units.

**Currency values**

Currency values (as seen in `#/components/schemas/Currency`) are defined as **decimal numbers** with a precision of maximum 2 decimal places, and interpreted in **Euros (€)**.

## 🐍 Python Client SDK
​
A _Python Client SDK_ is automatically generated from the OpenAPI specification and published to [PyPi](https://pypi.org/) under the [`helix-platform-api-public-client`](https://pypi.org/project/helix-platform-api-public-client/) project.

In order use the _Python Client SDK_ with the authenticated API endpoints you will need to generate an _API Key_ in **Helix**, see more in the [Getting Started](#-getting-started) section.
​
After installing the package you will be able to use the client by first instantiating a `Configuration` class and creating an `ApiClient` instance:
​
```python
configuration = Configuration(
    api_key={"ApiKey": "<your helix api key>"},
    host="https://api.training.helix.eleclink.co.uk"
)
client = ApiClient(configuration=configuration)
```

The `ApiClient` instance then can be used to construct individual clients for the different API categories.
​
> ⚠️ Please make sure you use the right _API Base URL_ of the [environment](#-environments) you wish to use in the `host` parameter!

An example for retrieving the Profile of a the current User (who the _API Key_ belongs to):
​
```python
profile_api = ProfileApi(api_client=client)
profile = profile_api.get_profile()
print(profile)
```

> ❗ When dealing with `datetime` objects, please do NOT use "naive" objects, always stick to ["aware"](https://docs.python.org/3/library/datetime.html#aware-and-naive-objects) ones, which include timezone information, otherwise your request will fail.

Example of correct `datetime` handling:
 
```python
from datetime import datetime, timezone

beginning_of_year = datetime(2023, 1, 1, 0, 0, 0, 0, timezone.utc)
beginning_of_year.isoformat()

now = datetime.now(timezone.utc)
now.isoformat()

# notice the correct `+00:00` timezone suffixes
```

## 📖 Changelog

Changes in particular versions of the API Specification (in the [📚 **openapi.yaml**](openapi.yaml) file) are automatically tracked in [CHANGELOG.md](CHANGELOG.md).

- 2024-10-22 Initial API documentation
