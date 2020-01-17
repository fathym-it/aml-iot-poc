# aml-iot-poc

## Subscription : Fathym R&D
## Resource Group : develop


**Operational Steps**
Setting up resources and deploying a test module to an IoT Edge device on a Linux VM
References: https://docs.microsoft.com/en-us/azure/iot-edge/quickstart-linux#deploy-a-module

#### 1. Create iot hub - standard tier :

```
az iot hub create --resource-group 'develop' --name aml-iot-poc-hub --sku S1 --partition-count 2
```

#### 2. Create ACR AMLIotPoCAcr 

```
az storage account create \
    --name AMLIotPoCAcr \
    --resource-group develop \
    --location 'West US 2, West Central US' \
    --sku Standard_ZRS \
    --encryption blob \
	
export AZURE_STORAGE_ACCOUNT="<account-name>" \
export AZURE_STORAGE_KEY="<account-key>"
```

#### 3. Create repo (aml-iot-poc)

```
git init
git add .gitignore
git add README.md
git commit
git remote add origin https://github.com/fathym-it/aml-iot-poc.git
git push -u origin master
```

#### 4. Create blob storage (amliotpocstorage) Cool storage


#### 5. Create Linux VM (TestIoTEdgeVM)
```
az vm image terms accept --urn microsoft_iot_edge:iot_edge_vm_ubuntu:ubuntu_1604_edgeruntimeonly:latest
az vm create --resource-group develop --name TestIoTEdgeVM --image microsoft_iot_edge:iot_edge_vm_ubuntu:ubuntu_1604_edgeruntimeonly:latest --admin-username azureuser --generate-ssh-keys
```

Result: 

```
{
  "fqdns": "",
  "id": "/subscriptions/32271d36-d77b-485b-b7b2-0c990d2abf56/resourceGroups/develop/providers/Microsoft.Compute/virtualMachines/TestIoTEdgeVM",
  "location": "westus2",
  "macAddress": "00-0D-3A-F9-05-2D",
  "publicIpAddress": "40.65.115.8",
  "resourceGroup": "develop",
  "zones": ""
}
```

#### 6.  Create IoT Edge device identity
```
az extension add --name azure-cli-iot-ext
az iot hub device-identity create --hub-name aml-iot-poc-hub --device-id TestIoTEdgeDevice --edge-enabled
```

Result:

```
{
  "authentication": {
    "symmetricKey": {
      "primaryKey": "nwGR7TzVXAbv+XTAPeziwJQPTLRkkUkqc9zamHLOVhk=",
      "secondaryKey": "aLYZIP2mnpwkek28+n6lBPDVeIzeFFLa7amYA7Klcnc="
	},
    "type": "sas",
    "x509Thumbprint": {
      "primaryThumbprint": null,
      "secondaryThumbprint": null
    }
  },
  "capabilities": {
    "iotEdge": true
  },
  "cloudToDeviceMessageCount": 0,
  "connectionStateUpdatedTime": "0001-01-01T00:00:00",
  "deviceId": "TestIoTEdgeDevice",
  "deviceScope": "ms-azure-iot-edge://TestIoTEdgeDevice-637146731770177879",
  "etag": "NDc5NjE2NzAw",
  "generationId": "637146731770177879",
  "lastActivityTime": "0001-01-01T00:00:00",
  "status": "enabled",
  "statusReason": null,
  "statusUpdatedTime": "0001-01-01T00:00:00"
}
```

```
az iot hub device-identity show-connection-string --device-id TestIoTEdgeDevice --hub-name aml-iot-poc-hub
```

Result:

```
{
  "connectionString": "HostName=aml-iot-poc-hub.azure-devices.net;DeviceId=TestIoTEdgeDevice;SharedAccessKey=nwGR7TzVXAbv+XTAPeziwJQPTLRkkUkqc9zamHLOVhk="
}
```

```
az vm run-command invoke -g develop -n TestIoTEdgeVM --command-id RunShellScript --script "/etc/iotedge/configedge.sh 'HostName=aml-iot-poc-hub.azure-devices.net;DeviceId=TestIoTEdgeDevice;SharedAccessKey=nwGR7TzVXAbv+XTAPeziwJQPTLRkkUkqc9zamHLOVhk='"
```

Result:

```
{
  "value": [
    {
      "code": "ProvisioningState/succeeded",
      "displayStatus": "Provisioning succeeded",
      "level": "Info",
      "message": "Enable succeeded: \n[stdout]\n Wed Jan 15 08:33:52 UTC 2020 Connection string set to HostName=aml-iot-poc-hub.azure-devices.net;DeviceId=TestIoTEdgeDevice;SharedAccessKey=nwGR7TzVXAbv+XTAPeziwJQPTLRkkUkqc9zamHLOVhk=\n\n[stderr]\n",
      "time": null
    }
  ]
}
```


#### 7.  Verify device availability

This must be done from Azure Cloud Powershell

```
ssh azureuser@40.65.115.8
```

Once in the shell:

```
sudo systemctl status iotedge
```

Result:

```
Jan 15 08:36:58 TestIoTEdgeVM iotedged[5697]: 2020-01-15T08:36:58Z [INFO] - [mgmt] - - - [2020-01-15 08:36:58.324717631 UTC] "GET /modules?api-ve
Jan 15 08:37:03 TestIoTEdgeVM iotedged[5697]: 2020-01-15T08:37:03Z [INFO] - [mgmt] - - - [2020-01-15 08:37:03.325574172 UTC] "GET /modules?api-ve
Jan 15 08:37:08 TestIoTEdgeVM iotedged[5697]: 2020-01-15T08:37:08Z [INFO] - [mgmt] - - - [2020-01-15 08:37:08.326306501 UTC] "GET /modules?api-ve
Jan 15 08:37:13 TestIoTEdgeVM iotedged[5697]: 2020-01-15T08:37:13Z [INFO] - [mgmt] - - - [2020-01-15 08:37:13.326921717 UTC] "GET /modules?api-ve
Jan 15 08:37:18 TestIoTEdgeVM iotedged[5697]: 2020-01-15T08:37:18Z [INFO] - [mgmt] - - - [2020-01-15 08:37:18.331780872 UTC] "GET /modules?api-ve
Jan 15 08:37:23 TestIoTEdgeVM iotedged[5697]: 2020-01-15T08:37:23Z [INFO] - [mgmt] - - - [2020-01-15 08:37:23.336880207 UTC] "GET /modules?api-ve
Jan 15 08:37:28 TestIoTEdgeVM iotedged[5697]: 2020-01-15T08:37:28Z [INFO] - [mgmt] - - - [2020-01-15 08:37:28.341468927 UTC] "GET /modules?api-ve
Jan 15 08:37:33 TestIoTEdgeVM iotedged[5697]: 2020-01-15T08:37:33Z [INFO] - [mgmt] - - - [2020-01-15 08:37:33.342230865 UTC] "GET /modules?api-ve
Jan 15 08:37:38 TestIoTEdgeVM iotedged[5697]: 2020-01-15T08:37:38Z [INFO] - [mgmt] - - - [2020-01-15 08:37:38.342870681 UTC] "GET /modules?api-ve
Jan 15 08:37:43 TestIoTEdgeVM iotedged[5697]: 2020-01-15T08:37:43Z [INFO] - [mgmt] - - - [2020-01-15 08:37:43.347125537 UTC] "GET /modules?api-ve
```

```
sudo iotedge list
```

Result:
```
NAME             STATUS           DESCRIPTION      CONFIG
edgeAgent        running          Up 4 minutes     mcr.microsoft.com/azureiotedge-agent:1.0
```


#### 8.  Deploy module - from Marketplace 

(see https://docs.microsoft.com/en-us/azure/iot-edge/quickstart-linux#deploy-a-module)

Verify routes for messaging

```
FROM /messages/* INTO $upstream
```

Module deployment output:

```
{
    "modulesContent": {
        "$edgeAgent": {
            "properties.desired": {
                "modules": {
                    "SimulatedTemperatureSensor": {
                        "settings": {
                            "image": "mcr.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0",
                            "createOptions": ""
                        },
                        "type": "docker",
                        "status": "running",
                        "restartPolicy": "always",
                        "version": "1.0"
                    }
                },
                "runtime": {
                    "settings": {
                        "minDockerVersion": "v1.25"
                    },
                    "type": "docker"
                },
                "schemaVersion": "1.0",
                "systemModules": {
                    "edgeAgent": {
                        "settings": {
                            "image": "mcr.microsoft.com/azureiotedge-agent:1.0",
                            "createOptions": ""
                        },
                        "type": "docker"
                    },
                    "edgeHub": {
                        "settings": {
                            "image": "mcr.microsoft.com/azureiotedge-hub:1.0",
                            "createOptions": "{\"HostConfig\":{\"PortBindings\":{\"443/tcp\":[{\"HostPort\":\"443\"}],\"5671/tcp\":[{\"HostPort\":\"5671\"}],\"8883/tcp\":[{\"HostPort\":\"8883\"}]}}}"
                        },
                        "type": "docker",
                        "status": "running",
                        "restartPolicy": "always"
                    }
                }
            }
        },
        "$edgeHub": {
            "properties.desired": {
                "routes": {
                    "route": "FROM /messages/* INTO $upstream",
                    "SimulatedTemperatureSensorToIoTHub": "FROM /messages/modules/SimulatedTemperatureSensor/* INTO $upstream"
                },
                "schemaVersion": "1.0",
                "storeAndForwardConfiguration": {
                    "timeToLiveSecs": 7200
                }
            }
        },
        "SimulatedTemperatureSensor": {
            "properties.desired": {
                "SendData": true,
                "SendInterval": 5
            }
        }
    }
}
```

Verify IoT edge device list again

```
sudo iotedge list
```

#### 9.  View data coming out of the device

```
sudo iotedge logs SimulatedTemperatureSensor -f
```

Result:

```
azureuser@TestIoTEdgeVM:~$ sudo iotedge logs SimulatedTemperatureSensor -f
[2020-01-15 08:49:23 : Starting Module
SimulatedTemperatureSensor Main() started.
Initializing simulated temperature sensor to send 500 messages, at an interval of 5 seconds.
To change this, set the environment variable MessageCount to the number of messages that should be sent (set it to -1 to send unlimited messages).
Information: Trying to initialize module client using transport type [Amqp_Tcp_Only].
Information: Successfully initialized module client of transport type [Amqp_Tcp_Only].
        01/15/2020 08:49:41> Sending message: 1, Body: [{"machine":{"temperature":21.294738574905665,"pressure":1.0335778123310251},"ambient":{"temperature":21.048630642960141,"humidity":25},"timeCreated":"2020-01-15T08:49:41.6091891Z"}]
        01/15/2020 08:49:46> Sending message: 2, Body: [{"machine":{"temperature":22.112142276071079,"pressure":1.126699752970123},"ambient":{"temperature":21.479175623496609,"humidity":24},"timeCreated":"2020-01-15T08:49:46.8283495Z"}]

```
