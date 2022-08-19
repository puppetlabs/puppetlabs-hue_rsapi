
# hue_rsapi

> This module is not supported or maintained by Puppet and does not qualify for Puppet Support plans.
> It's provided without guarantee or warranty and is not intended for public use. It was created as a
> simple example of the new way to create a custom Type and Provider using the
> [Resource API](https://github.com/puppetlabs/puppet-resource_api).
>
> [tier:demo]


#### Table of Contents

1. [Module Description - What the module does and why it is useful](#module-description)
2. [Setup - The basics of getting started with hue_rsapi](#setup)
    * [Setup requirements](#setup-requirements)
    * [Getting started with hue_rsapi](#getting-started-with-hue_rsapi)
3. [Usage - Configuration options and additional functionality](#usage)
4. [Reference - An under-the-hood peek at what the module is doing and how](#reference)
5. [Limitations - OS compatibility, etc.](#limitations)
6. [Development - Guide for contributing to the module](#development)


## Module Description

The hue_rsapi module will connect to a Philips Hue hub and retrieve basic details of the lights that are connected to it.

The module enables the retrieval and modification of the data.

## Setup

To install from source, download the tar file from GitHub and run `puppet module install <file_name>.tar.gz --force`.

### Setup Requirements

The module has a dependency on the `resource_api` - it will be installed when the module is installed. Alternatively, it can be manually installed by running `puppet module install puppetlabs-resource_api`, or by following the setup instructions in the [Resource API README](https://github.com/puppetlabs/puppetlabs-resource_api#resource_api).

### Getting started with hue_rsapi

To get started, create or edit `/etc/puppetlabs/puppet/device.conf`, add a section for the device (this will become the device's `certname`), specify a type of `philips_hue`, and specify a `url` to a credentials file. For example:

```INI
[homehub]
type philips_hue
url file:////etc/puppetlabs/puppet/devices/homehub.conf
```

Next, create a credentials file containing a host and key as shown below, with the port value being optional and defaulting to 80 if not set. For more detailed information on the credentials file please see the [schema](./lib/puppet/transport/schema/philips_hue.rb). See the [HOCON documentation](https://github.com/lightbend/config/blob/master/HOCON.md) for information on quoted/unquoted strings.

```
host: 10.0.10.1
key:  onmdTvd198bMrC6QYyVE9iasfYSeyAbAj3XyQzfL
port: 80
```

To obtain an API key for the device follow the steps on the [HUE Developer Site](http://www.developers.meethue.com/documentation/getting-started).

Test your setup and get the certificate signed:

```puppet device --verbose --target homehub```

This will request a certificate and set up the device for Puppet.

See the [`puppet device` documentation](https://puppet.com/docs/puppet/latest/puppet_device.html) for more options. See [Puppet CA](https://puppet.com/docs/puppet/latest/ssl_certificates.html) for signing the certificate on your puppet master.

## Usage

Now you can manage your `hue_light` resources. See the REFERENCE for

### Puppet Device

To get information from the device, use the `puppet device --resource` command. For example, to retrieve addresses on the device, run the following:

`puppet device --resource --target homehub hue_light`

To make changes to the light, write a manifest. Start by making a file named `manifest.pp` with the following content:

```
hue_light { '1':
  on => true,
  bri => 255,
  hue => 39139,
  sat => 255,
  effect => 'colorloop',
  alert => 'none',
}
```

Execute the following command:

`puppet device  --target homehub --apply manifest.pp`

This will apply the manifest. Puppet will check if light is already configured with these settings (idempotency check) and if it finds that it needs to make changes will make the appropriate API calls.

Note that if you get errors, run the above commands with `--verbose` - this will give you error message output.


## Unit Testing

Unit tests test the parsing and command generation logic, executed locally.

Use the PDK to run the tests:

```
pdk test unit
```

## Local Development

If you do not have a Philips Hue system handy when developing you may find the [Hue-Emulator](https://github.com/SteveyO/Hue-Emulator) tool useful. Use the emulator instead of a real system to test your changes. The tool is written in java. Download the latest version from github and launch the emulator, with the following command:

```
java -jar ./spec/fixtures/HueEmulator-v0.8.jar
```

Set the port to a number available on your system (usually the default `8000` is fine), then press "Start".

Have a look at the conf files in `spec/fixtures` and adjust to your local environment.

Once you have set everything up, use the following commands as examples to get you going:

To list available lights:
```
pdk bundle exec puppet device --verbose --debug --trace --modulepath spec/fixtures/modules --target=emulatorhub --deviceconfig spec/fixtures/device.conf --resource hue_light
```

To manage values of the lights:
```
pdk bundle exec puppet device --verbose --debug --trace --modulepath spec/fixtures/modules --target=emulatorhub --deviceconfig spec/fixtures/device.conf --apply examples/traffic_lights.pp
```
