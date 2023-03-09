### Importing

Import settings from another file. This could be fig, json, or toml.

```js
@import {
  LogSettings.*,
  AuthService.*,
  SecurityService.*,
  HealthCheckSettings.*,
  SendgridUri,
} from "./DefaultSettings.json"
```

### Environments

Fig will create a file for each environment. Environments are defined in Env.fig.

```js
@import { env } from "./Env.fig"
```

### Settings

Set imported values using dot notation.

```python
LogSettings.Level = "Information"
```

Or indent them for better readability.

```python
SecurityService
  AccessKey = "${uwresultsAK}"
  SecurityEdgeService.SigningKey = "${uwresultsSigningKey}"
```

Notice these values are secrets, replaced with variables by the build process as usual.

### Variables

Variables are used for various purposes. For example, in **DefaultSettings.json**, a variable called appName could be used like this:

```json
"Serilog": {
 "WriteTo": {
   "Name": "File",
   "Args": { "path": "./logs/${appName}.json"}
 }
}
```

We can set this from our fig file with the variables object.

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

### Arrays... still workshopping this one 😜

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
