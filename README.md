# Cybus Learn - Azure IoT

This repository describes how to connect Azure IoT Hub or Azure IoT Edge.

The purpose is to give Cybus Connectware users a guideline 
how to integrate Connectware with an Azure IoT Hub endpoint directly 
or with an Azure IoT Edge device acting as a transparent gateway to Azure IoT Hub.

## Prerequisites

Besides the Cybus Connectware and an Azure IoT Connector service 
you need a respective Azure IoT endpoint.

This could be either a direct cloud connection to Azure IoT Hub
or a connection to a Azure IoT Edge device, which is meant to run
a Azure IoT Edge runtimee near the network environment of the 
devices to be integrated.

This guide shows two examples:
- Using the Azure IoT Hub directly
- Using an Azure IoT Edge device configured as a transparent gateway to Azure Iot Hub

## Configure the Connectware with a Azure IoT Connector service

Follow these general settings in your Azure IoT Conector service:

- Get a [Cybus Connectware](https://www.cybus.io/) (Version 1.0.13 higher) and some license key.
- Prepare a service commissioning file
- Configure the required `Azure_Iot_Connection_String`
- Configure a log-level of you choice (telemetry data is logged, if set to `trace`)
- Configure the machine topic to publish events for (placeholder `machineTopic`)
- Configure the full PEM formatted chain of CA certificates, if accessing an edge devicee

The placeholder `Cybus::MqttRoot` is replaced with the MQTT root topic of the service,
which is defined as `services/<serviceId>` with serviceId derived from the metadata name.
This is for example in case of `Azure IoT Hub Test` finally `services/azureiothubtest`.

To be able to access the configured topic `test/AzureIoT`, the permissions of the default
role of this service are enhanced (regularly this role has just access to the service topics).

- Publish to the MQTT topic with some payload from your device (or a node-red flow):
see the [Node-Red flow](node-red-flow-test-event.json) for use with the Connectware workbench.
In the example we choose a source topic `${Cybus::MqttRoot}/test/${machineTopic}` to demonstrate a transformation rule within the Cybus::Mapping resource.

If the log level is set to `trace`, the respective container logs show successful data transfer
with a `Telemetry Data Dump` message.

An event produced in the NodeRED flow:
```
{
  "DeviceData": {
    "Temperature": 78.567,
    "Position": {
      "X": 35.02,
      "Y": 12.62,
      "Z": 3.45
    },
    "Status": "operational"
  }
}
```

is sent to the respective endpoint as (in this example):
```
{
  "deviceId": "TestDevice",
  "payload": {
    "Temperature": 78.567,
    "Position": {
      "X": 35.02,
      "Y": 12.62,
      "Z": 3.45
    },
    "Status": "operational"
  }
}
```

### Connect to Azure IoT Hub

For IoT Hub connections create a device in your Azure IoT Hub to get a connection string.

- Prepare a service comissioning file: 
see the [Azure Iot Hub Service Example](azure-iot-hub-connectware-service.yml)

- Set the `Azure_Iot_Connection_String` device connection string for an IoT Hub in the format: 
  `HostName=<your-iothub-name>.azure-devices.net;DeviceId=<your-device-id>;SharedAccessKey=<access-key-for-the-device>`

There is nothing more to configure for the IoT Hub, use the NodeRED flow or any MQTT
client to produce device events.

### Connect to Azure IoT Edge

For IoT Edge connections create and deploy an Azure IoT Edge device onto a machine capable
of running an Edge Runtime and configure proper certificates. 
Then obtain the respective Azure IoT Hub connection string.

- Prepare a service comissioning file: 
see the [Azure Iot Edge Service Example](azure-iot-edge-connectware-service.yml)

- Set the `Azure_Iot_Connection_String` device connection string for an IoT Edge device in the format: 
  `HostName=<your-iothub-name>.azure-devices.net;DeviceId=<your-device-id>;SharedAccessKey=<access-key-for-the-device>;GatewayHostName=<hostname-of-the-edge-device>`

- Set the `Azure_Iot_Hub_CaCertChain` parameter containing the full PEM formatted CA certificate chain

The certificate chain for the CA root and the intermediate CA certificates MUST be valid and complete
in order to successfully connect to the edge device. The generated default certificates during setup
of an Azure IoT Edge runtime environment could also be replaced by own a/o self-signed certificates.

Please note that all host names (`HostName`, `GatewayHostName`) must properly resolve to a 
reachable IP address, and the `GatewayHostName` must match the common name in the server certificate-

This can be verified e.g. with openssl:
```
# retrieve server certificate from a broker
openssl s_client -connect mqtt.fluux.io:8883

# find the subject and compare to the given  hostname
# ...
# subject=/CN=mqtt.fluux.io
# issuer=/C=US/O=Let's Encrypt/CN=Let's Encrypt Authority X3
```

## Verify successful integration

To see a successful integration you may use the Azure CLI and follow the 
[Cybus guide about monitoring](https://github.com/cybusio/example-how-to-monitor-azure-iot-hub-events)

## References

- [Install Microsoft Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)
- [Microsoft Azure IoT Homepage](https://azure.microsoft.com/services/iot-hub)
- [Microsoft Azure IoT Edge](https://azure.microsoft.com/services/iot-edge)
- [How to install IoT Edge](https://docs.microsoft.com/de-de/azure/iot-edge/how-to-install-iot-edge-linux)
- [Cybus Homepage](https://www.cybus.io/)
- [Cybus Connectware documentation](https://docs.cybus.io)
- [Cybus How to monitor Azure IoT Events](https://github.com/cybusio/example-how-to-monitor-azure-iot-hub-events)
