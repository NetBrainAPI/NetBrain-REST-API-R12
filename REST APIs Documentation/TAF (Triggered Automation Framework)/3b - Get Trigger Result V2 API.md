
# Get Trigger Result V2 API Design

## ***POST*** /V1/TAF/ResultV2
Use this API to query the full results of a triggered automation task, including repeat runs and follow-up triggered diagnoses. <br> This is the recommended replacement for the [Get Trigger Result API](3%20-%20Get%20Trigger%20Result%20API.md) (`/V1/TAF/Result`).

## Overview

When a trigger fires — either through the [Auto Trigger API](2%20-%20Auto%20Trigger%20API.md), the [Manually Trigger API](5%20-%20Manually%20Trigger%20API.md), or NetBrain's AI engine — NetBrain creates an incident and a task, then runs the associated Network Intents (NIs). This API lets you retrieve those NI execution results.

**Key differences from `/V1/TAF/Result`:**

| | Get Trigger Result (`/V1/TAF/Result`) | Get Trigger Result V2 (`/V1/TAF/ResultV2`) |
|---|---|---|
| Initial trigger run results | ✅ | ✅ |
| Repeat run results | ❌ | ✅ |
| Follow-up triggered diagnosis results | ❌ | ✅ |
| Change Management (runbook) results | ❌ | ✅ |
| AI-generated insight message and map | ❌ | ✅ |
| Incremental polling support | ❌ | ✅ |

**Incremental polling:** Because a single trigger can spawn multiple tasks over time (repeat runs, follow-up diagnoses), this API supports incremental result retrieval. Each response includes a `currentTaskIds` array — pass this as `option.excludeTaskIds` in your next request to receive only new results and avoid processing duplicates.

**Polling for status:** The task runs asynchronously. Poll this API repeatedly until `status` is no longer `0` (Pending) or `1` (Running). A `status` of `2` (Finished) means all NIs have completed. Empty `NIResults` on a Finished task typically means no Triggered Diagnosis with Member NIs was matched or the trigger condition was not satisfied — verify your Incoming Incident Type condition and IT System source configuration in Trigger Automation Manager.


## Detail Information

> **Title** : Get Trigger Result V2 API

> **Version** : 06/07/2026

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/TAF/ResultV2

> **Authentication** : 

|**Type**|**In**|**Name**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|Bearer Authentication| Headers | Authentication token | 

## Request body(****required***)

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|options.scope* | string  | If multiple domains, this field is <b>required</b>. <br> If only one domain, this field is <b>optional</b>. <br> If `tenantId` and `domainId` are specified below, this field can be left blank.|
|options.tenantId | string  | To specify a particular working tenant. |
|options.domainId | string  | To specify a particular working domain. |
|options.taskId* | string  | The taskId returned from a particular trigger. |
|options.excludeTaskIds | list of string  | Specify the task data you do <i>not</i> want to retrieve; `taskIds` corresponds to `currentTaskIds` in the Response. |


## Query Parameters(****required***)

> **No query parameter**

## Headers

> **Data Format Headers**

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
| Content-Type | string  | support "application/json" |
| Accept | string  | support "application/json" |

> **Authorization Headers**

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
| token | string  | Authentication token, get from login API. |

## Response

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|tenantId| String | The task execution context - Tenant ID. |
|domainId| String | The task execution context - Domain ID. |
|status| Integer | The task execution status.<br>`0`: Pending;<br>`1`: Running<br>`2`: Finished<br>`3`: Failed<br>`4`: Aborted |
|startTime|dateTime|Start time of the entire Trigger|
|endTime|dateTime|End Time of the entire Trigger (i.e. when status is 'completed')|
|isAITriggered|bool| Whether the result is of AI Trigger|
|currentTaskIds|list of string| `taskIds` retrieved this time will be used to exclude subsequent data retrievals. <br> By passing this result to `option.excludeTaskIds` parameter in the next request, it will not be retrieved again.|
|NIResults| Array | Results of Network Intent (NI) execution. |
|NIResults[].id| String | NI ID. |
|NIResults[].name| String | NI name. |
|NIResults[].isHomeNI| bool | Whether it is Home NI. |
|NIResults[].isRepeat| bool | Whether it is Repeat Run. |
|NIResults[].repeatInterval| int | The time interval of Repeat, in minutes. |
|NIResults[].diagnosisId| String | TAF Diagnosis ID. |
|NIResults[].diagnosisName| String | TAF Diagnosis Name. |
|NIResults[].diagnosisType| int | Types of Diagnosis: <br> `0`: Primary Diagnosis <br> `1` : Follow-up Trigger Diagnosis |
|NIResults[].currentTimes| int | Number of times this NI has run (Only used for repeat cases) |
|NIResults[].niResultTime| dateTime | Time point of the result|
|NIResults[].niResultId| String | NI result ID. |
|NIResults[].mapId| String | Map ID of the NI. |
|NIResults[].mapName| String | Map name of the NI. |
|NIResults[].outputMapId| String | Output map ID of the Intent. |
|NIResults[].outputMapName| String | Output map name of the Intent. |
|NIResults[].dashboardId| String | Dashboard ID. |
|NIResults[].dashboardName| String | Dashboard Name. |
|NIResults[].messages| Array | All Intent status code messages generated by the NI execution (not including Device Status Code). <br> If there is an alert status code, only the alert status code will be stored; no success status code. <br> If there is no alert status code but a success code, one success status code will be stored. <br>If there is no status code, no status code exists. |
|NIResults[].messages[].type| Integer | Status code types.<br>`0`: Unknown<br>`1`: Success<br>`2`: Error |
|NIResults[].messages[].message| String | The message details of a status code. |
|NIResults[].cmResults| Array | Change Management results. |
|NIResults[].cmResults[].logs| Array | Log list recorded during the Change Management process. |
|NIResults[].cmResults[].runbookInfos| Array | Runbook information corresponding to Change Management. |
|NIResults[].cmResults.runbookInfos[].runbookId| string | Runbook ID of Change Management Result |
|NIResults[].cmResults.runbookInfos[].runbookName| string | Runbook Name of Change Management Result |
|NIResults[].cmResults.runbookInfos[].runbookUrl| string | Runbook URL of Change Management Result |
|NIResults[].cmResults.runbookInfos[].deviceNames| Array | Device Names of Change Management Result |
|NIResults[].cmResults.runbookInfos[].isPreApproved| bool | Whether the Change Management Result has been approved or not|
|insightMessage|object|AI-generated insight message summarizing the diagnosis.|
|insightMessage.conclusion|string|Overall conclusion of the diagnosis.|
|insightMessage.keyFindings|list of string|Key findings from the diagnosis, each citing the CLI command(s) used.|
|insightMessage.diagnosticCheckpoints|Array|Ordered diagnostic steps that were performed.|
|insightMessage.diagnosticCheckpoints[].step|int|Step number.|
|insightMessage.diagnosticCheckpoints[].description|string|What the step checked.|
|insightMessage.diagnosticCheckpoints[].result|string|Result / observation of the step.|
|insightMessage.viewFullDiagnosis|string|URL to view the full diagnosis in NetBrain.|
|insightMap|object|Insight Map|
|insightMap.mapId|string|Insight Map ID|
|insightMap.mapName|string|Insight Map Name|
|insightMap.mapType|int|Insight Map Type|
|statusCode| integer | The returned status code of executing the API.  |
|statusDescription| string | The explanation of the status code.  |

## Example
```python
import json
import requests
import requests.packages.urllib3 as urllib3
 
urllib3.disable_warnings()

def getTokens(endpoint, user,password):
    login_api_url = r"/V1/Session"
    Login_url = endpoint + login_api_url
    data = {
        "username": user,
        "password": password
    }
    token = requests.post(Login_url, data=json.dumps(
        data), headers=headers, verify=False)
    if token.status_code == 200:
#         print(token.json())
        return token.json()["token"]
    else:
        return "error"
    
def GetTriggerResultV2(endpoint, API_Body):
    API_URL = r"/V1/TAF/ResultV2"
    api_full_url = endpoint + API_URL
    try:
        api_result = requests.post(api_full_url, data=json.dumps(API_Body), headers=headers, verify=False)
        if api_result.status_code == 200:
            return api_result.json()
        else:
            return api_result.json()
    except Exception as e:
        return str(e)
    
nb_endpoint = "https://netbraintech.com/ServicesAPI/API"
nb_username = ""
nb_password = ""
tenantId = ""
domainId = ""
source = "Python Sample"

api_body = {
    "option": {
        "taskId": "5c8930f8-c5c4-4645-94b9-d0f1a4c80a76",
        "excludeTaskIds": []
    }
}

headers = {
    "Accepted":"application/json",
    "Content-Type":"application/json"
}

token = getTokens(nb_endpoint, nb_username, nb_password)
headers["token"] = token

print(GetTriggerResultV2(nb_endpoint, api_body))
```
## Formatted Sample JSON Response
```python
{
  "tenantId": "de3935a3-831b-9d33-e8a5-bc12302b562a",
  "domainId": "311cd0e7-4482-4af9-9e4f-5783becad33f",
  "status": 2,
  "startTime": "2026-06-04T12:00:00Z",
  "endTime": "2026-06-04T12:01:15Z",
  "isAITriggered": false,
  "currentTaskIds": [
    "5c8930f8-c5c4-4645-94b9-d0f1a4c80a76"
  ],
  "NIResults": [
    {
      "id": "d7983990-e91f-4103-8795-fbd0daf28858",
      "name": "Device Reboot US-BOS-R1",
      "isHomeNI": true,
      "isRepeat": false,
      "repeatInterval": 0,
      "diagnosisId": "79756585-4d0f-ac3b-c26e-aab19bf0eaf1",
      "diagnosisName": "Python Sample",
      "diagnosisType": 0,
      "currentTimes": 1,
      "niResultTime": "2026-06-04T12:01:10Z",
      "niResultId": "c2ac05cd-de61-4662-805f-0da397963752",
      "mapId": "052e2271-3d97-4ab8-a413-b5674f3a04a6",
      "mapName": "Device Reboot Map",
      "outputMapId": "8a31f0d2-1c4a-4b6f-9e3e-71b2d8c40e51",
      "outputMapName": "Device Reboot Output Map",
      "dashboardId": "be1a0f24-3d55-49a8-b1ed-9c2e26411a7a",
      "dashboardName": "TAF Dashboard",
      "messages": [
        {
          "type": 2,
          "message": "US-BOS-R1 was rebooted recently. Uptime: 35 weeks, 1 day, 11 hours, 26 minutes. Last reload reason: Unknown reason"
        }
      ],
      "cmResults": [
        {
          "logs": [],
          "runbookInfos": [
            {
              "runbookId": "92a7e1c4-5d8f-4f3b-a04e-3f8e7b21d6c9",
              "runbookName": "Reboot Recovery Runbook",
              "runbookUrl": "https://<your-host>/runbook/92a7e1c4",
              "deviceNames": ["US-BOS-R1"],
              "isPreApproved": true
            }
          ]
        }
      ]
    }
  ],
  "insightMessage": {
    "conclusion": "No sustained high CPU was found on SW1. Live samples show low average CPU with intermittent spikes. Top short-lived consumers are Virtual Exec and IP Input. No memory pressure or interface errors observed.",
    "keyFindings": [
      "CPU samples: 5s ~8% (peaked 15% in sorted), 1-minute ~7-8%, 5-minute ~3%. (show processes cpu / show processes cpu sorted)",
      "CPU history shows intermittent spikes over 60s/60m/72h but no sustained max. (show processes cpu history)",
      "Top CPU users are Virtual Exec (PIDs 215, 205) and IP Input. (show processes cpu sorted)",
      "Memory: large free pools (Processor free ~117 MB). (show memory statistics)",
      "Interfaces: 0 input/output errors and 0 drops. (show interfaces | include errors|input errors|output errors|drops)",
      "Target resolution: literal device SW1 found (Cisco 2811, mgmt 172.24.101.202). Group expansion returned empty."
    ],
    "diagnosticCheckpoints": [
      {
        "step": 1,
        "description": "Resolve target device",
        "result": "Group expansion empty; literal lookup returned SW1 (Cisco 2811, 172.24.101.202)."
      },
      {
        "step": 2,
        "description": "Live CPU sampling",
        "result": "Ran show processes cpu and show processes cpu sorted; observed low sustained CPU with brief spikes and top processes Virtual Exec and IP Input."
      },
      {
        "step": 3,
        "description": "CPU trend analysis",
        "result": "show processes cpu history shows intermittent spikes across seconds/minutes/hours but no sustained high CPU."
      },
      {
        "step": 4,
        "description": "Memory & interfaces health",
        "result": "show memory statistics shows healthy free memory; show interfaces shows 0 errors/drops."
      }
    ],
    "viewFullDiagnosis": "http://10.10.33.121/i.html?id=100006&dd=ba3a23dc-5d62-4427-8e30-91cbd32dc2a5"
  },
  "insightMap": {
    "mapId": "915b5fe8-e5b3-47e3-8d58-a3576b8fdb2b",
    "mapName": "Context Map",
    "mapType": 17
  },
  "statusCode": 790200,
  "statusDescription": "Success."
}
```
