# mirthpy
Python api wrapper for Mirth Connect

> **_NOTE:_**  This api wrapper is not totally complete with all of Mirth's REST api calls. If you have a request for a specific api call, please make an issue on GitHub.

#### Table of Contents
[Installation](#Installation)

[Setup](#Setup)

[Usage](#Usage)

[Supported Mirth API Calls](#SupportedCalls)

## Installation
<a name="Installation"></a>

Use the package manager [pip](https://pip.pypa.io/en/stable/) to install mirthpy.

```bash
pip install mirthpy
```

<a name="Setup"></a>
## Setup
Config file must match the following: 
```sh
{
    "instanceName": "{instance IP or DNS Name}",
    "credentials": {
        "username": "{username}", 
        "password": "{password}"
    }
}
```

alternatively you can provide the mirth instance (ip or domain name) and user/pass combo


```sh
service = MirthService(username="{username}", password="{password}", instance="{mirthInstance}")
```
<a name="Usage"></a>
## Usage

```python
import json
import mirthpy
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning) #to supress ssl warning

config = json.load(open('config.json'))

service = MirthService(config)
service.open()  # to login and initialize any service variables

# Get channel names
namesAndIds = service.getChannelIdsAndNames()
for ni in namesAndIds.entry:
    print(f"{ni.string[1]} - {ni.string[0]}") # channelName - channelId

# Get Id by name
id = service.getChannelIdByName('Test Channel')

channel = service.getChannel(id)

print(channel.name)
print(channel.description)
print(channel.sourceConnector.transformer)

if type(channel.sourceConnector.properties) == JavaScriptReceiverProperties:
    # do stuff we'd know only a javascript reader would have
    script = channel.sourceConnector.properties.script

# Get multiple channels
channelIds = ["44dacc20-718a-490c-bfcf-82355ed9209d", "3bb08639-7725-4b1b-a2ac-13687101fe9b"]
c = service.getChannels(channelIds)

for channel in c.channels: print(channel.name)


# Get all channels
channels = service.getChannels()

for channel in channels.channels:
    print(f"{channel.name} - {channel.description}")


# get all enabled channels
enabledChannels = [channel for channel in channels.channels if channel.exportData.metadata.enabled == 'true']

# get only javascript readers
jReaders = [channel for channel in enabledChannels if type(channel.sourceConnector.properties) == JavaScriptReceiverProperties]

# get certain code templates
codeTemplates = service.getCodeTemplates(['0d42f17b-54a4-40c1-8e7b-b43c3eb1b433', '37139e86-88db-41fa-bf80-8c08456caa10'])

for ct in codeTemplates.codeTemplates:
    print(f"{ct.name} - {ct.id}")

# get all code templates
codeTemplates = service.getCodeTemplates()

for ct in codeTemplates.codeTemplates:
    print(f"{ct.name} - {ct.id}")
    code = ct.properties.find('code').text

service.close() # to log out and close any service related objects
```
<a name="SupportedCalls"></a>
## Supported Mirth API Calls
### Server

| Mirth API Route | mirthpy method | Description                |
| :-------- | :------- | :------------------------- |
| `GET /server/version` | `service.getVersion()` | Gets Mirth Instance version |
| `POST /server/_generateGUID` | `service.getGUID()` | Gets Mirth specific GUID for ids |
| `GET /server/time` | `service.getTime()` | Gets current Mirth Server time |
| `GET /server/channelTags` | `service.getTags()` | Gets all tags |
| `GET /server/configurationMap` | `service.getConfigurationMaps()` | Gets all configuration mappings |

### Events

| Mirth API Route | mirthpy method | Description                |
| :-------- | :------- | :------------------------- |
| `/events` | `service.getEvents(jsonParams)` | Gets mirth events, can specify what parameters to pass |

### Channel

| Mirth API Route | mirthpy method | Description                |
| :-------- | :------- | :------------------------- |
| `POST /channels` | `service.createChannel(channel)` | Creates a channel in mirth instance based on channel object |
| `GET /channels` | `service.getChannels(channelIds, pollingOnly, includeCodeTemplates)` | Get all channels or certain channels in criteria |
| `GET /channels/{channelId}` | `service.getChannel(channelId, includeCodeTemplates)` | Get a single channel |
| `GET /channels/{channelId}` | `service.getChannel(channelId, includeCodeTemplates)` | Get a single channel |
| `GET /channels/idsAndNames` | `service.getChannelIdsAndNames()` | Get all channel name and ids |
| `GET /channels/idsAndNames` | `service.getChannelIdByName(name)` | Get only the channel id based on channel name |
| `POST /channels/{channelId}/_stop` | `service.stopChannel(channelId)` | Stop specified channel |
| `POST /channels/{channelId}/_start` | `service.startChannel(channelId)` | Start specified channel |

### Channel Status

| Mirth API Route | mirthpy method | Description                |
| :-------- | :------- | :------------------------- |
| `GET /channels/statuses` | `service.getChannelStatus(channelIds, filter, includeUndeployed)` | Get Dashboard status of channels |

### Code Template Calls

| Mirth API Route | mirthpy method | Description                |
| :-------- | :------- | :------------------------- |
| `GET /codeTemplates` | `service.getCodeTemplates(codeTemplateIds)` | Get all or some Code Templates  |

### Message Calls

| Mirth API Route | mirthpy method | Description                |
| :-------- | :------- | :------------------------- |
| `GET /channels/{channelId}/messages` | `service.getMessages(channelId, limit, includeContent, offset, startDate, endDate, status)` | Get messages by channelId with filtering options  |
| `GET /channels/{channelId}/messages/count` | `service.getMessageCount(channelId, startDate, endDate, status)` | Get messages count by channelId with filtering options  |

### Channel Deployment

| Mirth API Route | mirthpy method | Description                |
| :-------- | :------- | :------------------------- |
| `POST /channels/_deploy` | `service.deployChannels(channelIds, returnErrors)` | Deploy set of channels  |

### Channel Statistics

| Mirth API Route | mirthpy method | Description                |
| :-------- | :------- | :------------------------- |
| `GET /channels/statistics` | `service.getChannelStatistics(channelIds, includeUndeployed, includeMetadataId, excludeMetaDataId, aggregateStates)` | Get Channel Statistics  |


### Extensions (Commercial License)

| Mirth API Route | mirthpy method | Description                |
| :-------- | :------- | :------------------------- |
| `GET /extensions/history/channels/{channelId}/snapshots` | `service.getChannelHistory(channelId)` | Get Channel History  |
| `GET /extensions/history/channels/{channelId}/snapshots/{snapshotId}` | `service.getSnapshot(channelId, snapshotId)` | Get Channel History Snapshot  |
