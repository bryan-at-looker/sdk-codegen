# Looker SDK

The Looker SDK for Kotlin provides a convenient way to communicate with the Looker API available on your Looker server.

**DISCLAIMER**: This is an _alpha_ version of the Looker SDK, using a completely new code generator developed by Looker. Implementations are still subject to major change. If you run into problems with the SDK, feel free to [report an issue](https://github.com/looker-open-source/sdk-codegen/issues), and please indicate which language SDK you're using in the report.

## Getting started

The Looker SDK can be used in a Kotlin application in 3 steps:

* configure
* install
* use

### Configure the SDK for your Looker server

**Note**: The `.ini` configuration for the Looker SDK is a sample implementation intended to speed up the initial development of Node applications using the Looker API. See this note on [Securing your SDK Credentials](https://github.com/looker-open-source/sdk-codegen/blob/master/README.md#securing-your-sdk-credentials) for warnings about using `.ini` files that contain your API credentials in a source code repository or production environment.

Create a `looker.ini` file with your server URL and API credentials assigned as shown in this example.

```ini
[Looker]
# API version defaults to 3.1. 3.1 and 3.0 are currently supported. 3.1 is highly recommended.
api_version=3.1
# Base URL for API. Do not include /api/* in the url
base_url=https://<your-looker-server>:19999
# API 3 client id
client_id=your_API3_client_id
# API 3 client secret
client_secret=your_API3_client_secret
```

**Note**: If the application using the Looker SDK is going to be committed to a version control system, be sure to
**ignore** the `looker.ini` file so the API credentials aren't unintentionally published.

### Install the Looker SDK for Kotlin

The alpha version of the Looker SDK is not published to a Kotlin Package Manager. It's way too early for that. Currently, the only way to get the source code is by cloning the Looker SDK Codegen repository and use the source code in the `kotlin` folder.

To ensure you have the version of the SDK that matches your Looker version, you can regenerate `methods.kt` and `models.kt` from the root of the repository with the command:

```bash
yarn sdk kotlin
```

If this command fails the first time, read the [instructions for setting up `yarn`](https://github.com/looker-open-source/sdk-codegen/blob/master/README.md#using-the-yarnnode-based-generator)

### Use the SDK in your code

When the SDK is installed and the server location and API credentials are configured in your `looker.ini` file, it's ready to be used.

Verify authentication works and that API calls will succeed with code similar to the following:

```kotlin
val localIni = "./looker.ini"
val settings = ApiSettingsIniFile(localIni, "Looker")
val session = UserSession(settings, Transport(settings))
val sdk = LookerSDK(session)
// Verify minimal SDK call works
val me = sdk.ok<User>(sdk.me())

/// continue making SDK calls
val users = sdk.ok<Array<User>>(sdk.all_users())}
```

## Using AuthSession for automatic authentication

**NOTE**: As we secure the design of the Looker SDK's authentication practices, the authentication behavior described in this section will likely change.

Almost all requests to Looker's API require an access token. This token is established when the `login` endpoint is called with correct API3 credentials for `client_id` and `client_secret`. When `login` is successful, the user whose API3 credentials are provided is considered the active user. For this discussion of `AuthSession`, we'll
call this user the **API User**.

The `settings` provided to the `AuthSession` class include the base URL for the Looker instance, and the API3 credentials. When API requests are made, if the auth session is not yet established, `AuthSession` will automatically authenticate the **API User**. The `AuthSession` also directly support logging in as another user, usually called `sudo as` another user in the Looker browser application.

API users with appropriate permissions can `sudo` as another user by specifying a specific user ID in the `AuthSession.login()` method. Only one user can be impersonated at a time via `AuthSession`. When a `sudo` session is active, all SDK methods will be processed as that user.

## Environment variable configuration

[Environment variables](https://github.com/looker-open-source/sdk-codegen#environment-variable-configuration) can be used to configure access for the Looker SDK.

## THIS IS AN ALPHA VERSION!!!

There are still some issues processing the full return values of some of the more complex structures returned by Looker API endpoints. While these JSON parsing issues are being resolved, restricting the set of fields to be returned by the problematic endpoint is a simple work-around. To save time, some constants for currently supported fields are defined in the SDK.

```kotlin

/// safely get a look via the SDK
val look = sdk.ok<LookWithQuery>(sdk.look(id, fields = Safe.Look))

/// safely get a dashboard
val actual = sdk.ok<Dashboard>(sdk.dashboard(id, fields = Safe.Dashboard))
```

**IMPORTANT**: You'll also want to ensure the <src/main/com/looker/sdk/models.kt> property `parent_id` is optional in the `FolderBase`, `Folder`, `SpaceBase` and `Space` data classes. If it's not, make it look like this:

```kotlin
  var parent_id: String? = null,
```

Enjoy, and thanks for trying out the bleeding edge!
