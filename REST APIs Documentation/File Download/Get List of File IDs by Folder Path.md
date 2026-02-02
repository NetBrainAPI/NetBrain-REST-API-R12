
# File Download API Design

## ***POST*** /V1/CMDB/Files/BinaryFiles/GetAllFilesByFolderPath
This API is used to get the List of File ID by Folder Path.

## Detail Information

> **Title** : Get List of File IDs by Folder Path API<br>

> **Version** : 29/01/2026.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/CMDB/Files/BinaryFiles/GetAllFilesByFolderPath

> **Authentication** : 

|**Type**|**In**|**Name**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|Bearer Authentication| Headers | Authentication token | 

## Request body(****required***)
|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|folderpath* | string | Path of the folder. <br>e.g. `Public/OpenTopo` |

## Parameters(****required***)
>No parameters required.


## Headers

> **Data Format Headers**

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
| Content-Type | string | support "application/json" |
| Accept | string | support "application/json" |

> **Authorization Headers**

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
| token | string  | Authentication token, get from Login API. |

## Response

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|items|array of object| List of retrieved files. |
|items.originalId|string|  |
|items.name|string| Full name of the file including its folder path. |
|items.type|int| Types of file to retrieve. <br> ▪ `0`: Folder<br> ▪ `11`: Map<br> ▪ `21`▪  Dashboard<br> ▪ `999` - Windows files |
|statusCode| int | The returned status code of executing the API. |
|statusDescription| string | The explanation of the status code. |

# Full Example:
```python
# import python modules 
import requests
import time
import urllib3
import pprint
import json
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

token = "005fd6cc-cf08-4742-985b-902503dad2a4"
full_url= nb_url + "/ServicesAPI/API/V1/CMDB/Files/BinaryFiles/GetAllFilesByFolderPath"
headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["Token"]=token

body = { 
    "folderpath": "Public/test"
}

try:
    response = requests.post(full_url, headers = headers, data = json.dumps(body), verify = False)
    if response.status_code == 200:
        result = response.json()
        print (result)
    else:
        print ("Get List of File IDs by Folder Path API! - " + str(response.text))
except Exception as e:
    print (str(e))
```
```python
{
  "items": [
    {
      "originalId": "697ce52d422b5566af6f3c2e",
      "id": "a5627950-462d-4d9d-8274-040c7ac666be",
      "name": "Public/test/CiscoIOSSwitch_deviceinfo",
      "type": 999
    }
  ],
  "statusCode": 790200,
  "statusDescription": "Success."
}
```

# cURL Code from Postman:
```python
curl -X POST \
  "http://192.168.28.44/ServicesAPI/API/V1/CMDB/Files/BinaryFiles/GetAllFilesByFolderPath" \
  -H "Content-Type: application/json" \
  -H "cache-control: no-cache" \
  -H "token: 82b932fe-6126-4eea-b200-a10f88bf6a6c"\
  -d '{ 
    "folderpath": "Public/test"
}'
```

