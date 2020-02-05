# env-map-buildpack

Allows for customised flattening of env vars in Cloud Foundry `VCAP_SERVICES` based on `jq` selectors. Requires a stack with `jq` already installed.

> NB: This is intended only for situations where high levels of customisation are needed e.g. 3rd party software. For most situations, the [pancake-buildpack](https://github.com/starkandwayne/pancake-buildpack) is likely a better option.

## Background

When using Cloud Foundry, services such as databases are provided by service bindings using the Open Service Broker API. The credentials, URLs etc. of these services are placed into a JSON formatted environment variable called `VCAP_SERVICES`. To consume these credentials applications typically either include code to parse the JSON or use custom entry scripts.

This buildpack utilizes the `profile.d` directory to supply a script that will export environment variables based on a selectors using the popular `jq` binary. Any scripts present in the `profile.d` directory will be `source`d when the application starts up.

## Usage

To use this buildpack, include [https://github.com/andy-paine/env-map-buildpack](https://github.com/andy-paine/env-map-buildpack) anywhere but the last element in the `buildpacks` field in your Cloud Foundry manifest:
```yaml
buildpacks:
  - https://github.com/andy-paine/env-map-buildpack
  - python_buildpack
```
Include an `env-map.yml` file in the directory which you `cf push` - see [examples](examples/) directory for full details.

Any selector which works when performing `echo "$VCAP_SERVICES" | jq '<selector>'` will work with this buildpack.

This buildpack does not attempt to resolve situations such as multiple results returned by selector or invalid selectors.

> To use an alternative file name/location, set `ENV_MAP_BP_CONFIG` to the path of your config file relative to the root of the app which is pushed

### JSON/offline mode
To make configuration easier, YAML is the default configuration format. To parse this format, the buildpack downloads and uses [yq](https://github.com/mikefarah/yq). For users who cannot download external binaries or for users who prefer JSON for configuration, you can provide a mapping file in JSON format with a `.json` extension. This will skip downloading `yq` and so should work on security concious/air-gapped environments.

To instruct the buildpack to use a JSON formatted file, use the `ENV_MAP_BP_CONFIG` environment variable detailed above.

### Java Buildpack support

The java buildpack ([cloudfoundry/java-buildpack](https://github.com/cloudfoundry/java-buildpack)) does not support
sourcing environment variables from `deps/$IDX/profile.d` (which is part of the
[core buildpack communication contract](https://docs.cloudfoundry.org/buildpacks/custom.html#-core-buildpack-communication-contract)).

The env-map-buildpack relies on being able to use the `.profile.d` directory to set environment variables just before
the application starts.

To work around the java buildpack's lack of support for this, instead of writing to the `deps/$IDX/profile.d` directory
env-map-buildpack can write directly to the app's `app/.profile.d` directory, which will get sourced, even in the java buildpack.

To enable this behaviour, set the `ENV_MAP_BP_USE_APP_PROFILE_DIR` environment variable to `"true"`.

