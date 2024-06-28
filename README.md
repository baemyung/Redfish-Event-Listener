# Redfish Event Listener

Copyright 2017-2022 DMTF. All rights reserved.

## About

The Redfish Event Listener is a lightweight HTTP(S) server that can be deployed to read and record events from Redfish services.  It can also subscribe to multiple services.

## Prerequisites

The Redfish Event Listener requires Python3 on the user's system.  Additionally, the following Python packages are required:

* redfish
* redfish_utilities

To install the required packages, use either command:

`pip install <package name>`
`pip install -r requirements.txt`

To upgrade already installed packages, use the command:

`pip install --upgrade <package name>`

The target Redfish services must also be configured to send Redfish events.

## Configuration

The file `config.ini` contains a default configuration for the tool.  This can be used as a template for real configurations or edited as needed.  [Sample Configuration with All Options](#sample-configuration-with-all-options) shows an example with all possible configuration options.

The configuration file is broken into different sections with different options.

The `Information` section contains details about the configuration file itself.  The tool does not parse any information in this section, and is used to help users track configuration files.  The following options are found in this section:

* `Updated`: The date the file was updated.
* `Description`: A description of the configuration.

The `SystemInformation` section contains settings for the system acting as the event listener.  All options in this section are required.  The following options are found in this section:

* `ListenerIP`: The interface to use to listen for events.  This is typically `0.0.0.0` to listen on all interfaces for IPv4 or `::` to listen on all interfaces for IPv6.
* `ListenerPort`: The port number to use to listen for events.  This is typically `80` for HTTP or `443` for HTTPS.
* `UseSSL`: `on` to use HTTPS, `off` to use HTTP for incoming event messages.  If HTTPS is to be used, the `CertificateDetails` section will need to be configured.

The `CertificateDetails` section contains the certificate information for the system acting as the event listener.  This section and its options are only required if `UseSSL` in `SystemInformation` is set to `on`.  The following options are found in this section:

* `certfile`: The file containing the certificate for the event listener.  The tool comes with a default self-signed certificate named `cert.pem`.
* `keyfile`: The file containing the certificate private key for the event listener.  The tool comes with a default private key named `server.key`.

The `SubscriptionDetails` section contains subscription information so that the Redfish service can send the desired events to the event listener.  The `Destination` option is required; other options can be omitted or have an empty value.  The following options are found in this section:

* `Destination`: The URI of the event listener.  The Redfish service will perform an HTTP POST on this URI to send events to the event listener.  If provided, the tool will apply the value to the `Destination` property in the `EventDestination` resource.
* `Context`: The context string to be transmitted back by the Redfish service when sending events.  If provided, the tool will apply the value to the `Context` property in the `EventDestination` resource.
* `Format`: The format of the event notifications the event listener will receive.  Possible values are `Event` and `MetricReport`.  If provided, the tool will apply the value to the `EventFormatType` property in the `EventDestination` resource.
* `Expand`: `true` indicates if the `OriginOfCondition` property will contain the expanded resource in the event payload.  `false` indicates no payload expansion.  If provided, the tool will apply the value to the `IncludeOriginOfCondition` property in the `EventDestination` resource.
* `ResourceTypes`: An array of resource types the Redfish service will use to filter events for the event listener.  Some examples include `Chassis`, `Manager`, and `ComputerSystem`.  If provided, the tool will apply the value to the `ResourceTypes` property in the `EventDestination` resource.
* `Registries`: An array of registry names the Redfish service will use to filter events for the event listener.  Some examples include `ResourceEvent` and `TaskEvent`.  If provided, the tool will apply the value to the `RegistryPrefixes` property in the `EventDestination` resource.
* `EventTypes`: An array of classes of events the event listener will receive.  This setting has been deprecated by Redfish in favor of the above settings.  Possible values are `StatusChange`, `ResourceUpdated`, `ResourceAdded`, `ResourceRemoved`, `Alert`, `MetricReport`, and `Other`.  If provided, the tool will apply the value to the `EventTypes` property in the `EventDestination` resource.

The `ServerInformation` section contains information about the Redfish services that will transmit events to the event listener.  All options in this section, except `LoginType`, are required.  The following options are found in this section:

* `ServerIPs`: An array of URIs of the Redfish services for subscribing.
* `UserNames`: An array of Redfish usernames for each of the listed Redfish services.  This must be the same array length as `ServerIPs`.
* `Passwords`: An array of Redfish passwords for each of the listed Redfish services.  This must be the same array length as `ServerIPs`.
* `LoginType`: An array of login types for each of the listed Redfish services.  Possible values are `Basic` and `Session`.  If not specified, `Session` is used.

## Running the Tool

Execute using `python RedfishEventListener_v1.py`

Received event details will be captured on the console and recorded into a file named Events_<Service IP>.txt in the working directory.  Individual files will be generated for each subscribed service.

The tool can be stopped by issuing a keyboard interrupt (CTRL-C).

## Limitations

* The subscription information remains the same for all the subscriptions initiated from the tool.
* The event counter will restart in the event file each time the tool is restarted.

## Sample Configuration with All Options

The following example shows a configuration file with all possible options supported by the tool.  Many Redfish services do not support every possible option, so if a service is returning the HTTP `400 Bad Request` status code, inspect the error message to determine which options are not supported, and modify the configuration file accordingly.

```
[Information]
Updated = February 24, 2023
Description = Redfish Event Listener Tool Full Config

[SystemInformation]
ListenerIP = 0.0.0.0
ListenerPort = 443
UseSSL = on

[CertificateDetails]
certfile = cert.pem
keyfile = server.key

[SubscriptionDetails]
Destination = https://<ListenerIP>/
EventTypes = [
    "Alert",
    "ResourceRemoved",
    "ResourceAdded" ,
    "ResourceUpdated",
    "StatusChange"]
Context = Public
Format = Event
Expand = false
ResourceTypes = ["Chassis"]
Registries = ["ResourceEvent"]

[ServerInformation]
ServerIPs = ["https://<RedfishIP1>","https://<RedfishIP2>"]
UserNames = ["Username1","Username2"]
Passwords = ["Password1","Password2"]
LoginType = ["Session","Basic"]

```

## Release Process

1. Go to the "Actions" page
2. Select the "Release and Publish" workflow
3. Click "Run workflow"
4. Fill out the form
5. Click "Run workflow"
