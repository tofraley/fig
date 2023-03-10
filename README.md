## Introduction

Fig is a language that you write configs in. It has a minimal syntax, and tries to make it easy to manage configs for multiple environments, share configs, and navigate between shared configs.

The Fig compiler then takes your configs and generates equivalent json files to be used by your programs. A different file will be generated for each environment.

### Sample

```python
@import
  from "./DefaultSettings.json"
    LogSettings.*
    SecurityService.*
    HealthCheckSettings.*
    SomeUrl

  from "./BoardingFeatures.json"
    AwesomeFeature
    ExperimentFeature

  from "./Utils.fig"
    env_specific_url

@environments from "./Env.fig"

LogSettings.Level = "Information"

LogSettings.variables
  appName = "UnderwritingResultsAPI"
  useEnrichers = true

SecurityService
  AccessKey = "${accessKey}"
  SecurityEdgeService.SigningKey = "${signingKey}"

ExperimentFeature =
  @dev=true
  @qa=true

HierarchyBaseUri = @env_specific_url("https://boarding.clearent.net")

HealthCheckSettings.HealthCheckTasks = [
  {
    Type = SQLDatabase
    Name = "ClearentDatabaseConnection"
    ConnectionString = "${ConnectionStrings:ClearentConnectionString}"
  },
  {
    Type = SQLDatabase
    Name = "MerchantVerificationDatabaseConnection"
    ConnectionString = "${ConnectionStrings:MerchantVerification}"
  },
]
```

## Features

🧪 - Experimental, check back frequently

### Importing

Import settings from another file. This could be fig, json, or toml.

```js
@import
  from "./DefaultSettings.json"
    LogSettings.*
    SecurityService.*
    HealthCheckSettings.*
    SomeUrl
```

Use a feature flag created in another file. When you're ready to turn it on, you just have to change it in one place!

```js
@import NewAwesomeFeature from "./BoardingFeatures.json"
```

But of course you can set it in this file just like any other imported setting. See the section on [Settings](#settings).

### Environments 🧪

Fig will generate a json file for each environment. Configure your environments like this.

```js
@environments from "./Env.fig"
```

### Settings

Set imported values using dot notation.

```python
LogSettings.Level = "Information"
```

Or indent them for better readability.

```python
LogSettings
  Level = "Information"
  WriteTo = "File"
```

### Variables

Variables are used for various purposes.
For example, in **DefaultSettings.json**, a variable called appName might be used like this:

```json
"Serilog": {
 "WriteTo": {
   "Name": "File",
   "Args": { "path": "./logs/${appName}.json"}
 }
}
```

We can set this from our Fig file with the variables object.

```python
Serilog.variables
  appName = "UnderwritingResultsAPI"
  useEnrichers = true
```

If there are no variables, or that variable does not exist, you will get an error!

Provide different values for each environment. Available environments are defined in Env.fig.

```python
Features
  UseNewFeature =
    @dev=true
    @qa=true
```

By default, if you don't provide a value for every environment, it will not be included. But you can configure this behavior.

Or inject env names like a variable. Here, `$env` becomes the environment name ("dev", "qa", etc)

```python
$env_modifier = "-" + $env
HierarchyBaseUri = "https://boarding${env_modifier}.clearent.net"
```

Or define your own functions and use them. Here, `env_specific_url()` appends the env name to the subdomain.

```python
HierarchyBaseUri = @env_specific_url("https://boarding.clearent.net")

```

### Arrays 🧪

```python
HealthCheckSettings.HealthCheckTasks = [
  {
    Type = SQLDatabase
    Name = "ClearentDatabaseConnection"
    ConnectionString = "${ConnectionStrings:ClearentConnectionString}"
  },
  {
    Type = SQLDatabase
    Name = "MerchantVerificationDatabaseConnection"
    ConnectionString = "${ConnectionStrings:MerchantVerification}"
  },
  {
    Type = CustomCode
    Name = "CryptoApi"
    IntervalBetweenRunInSeconds = 120
    Description = "Custom Health Check for Crypto API"
    Settings = {
      ShouldReturnFullStatusReportToInfo = true,
      CryptoApiUrl = "${SettlementCryptoBaseUri}"
    }
  }
]
```

## Future features

### Enums 🧪

It would be nice to have some type safe options instead of strings.

```python
LogSettings.WriteTo = LogLocations.File
```

### Validated settings 🧪

What if we could define validators to make sure imported settings are used correctly.

For example, what if we want to make sure urls use one of a list of domains.

So this would work:

```
MyUrl = "https://compass.clearent.net"
```

But this would error:

```
MyUrl = "https://copmass.clearent.net"
```
