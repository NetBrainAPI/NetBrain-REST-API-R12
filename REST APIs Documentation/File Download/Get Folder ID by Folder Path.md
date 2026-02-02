
# File Download API Design

## ***POST*** /V1/CMDB/Files/GetFolderIdByFolderPath
This API is used to get the Folder ID by its folder path.

## Detail Information

> **Title** : Get Folder ID by Folder Path API<br>

> **Version** : 29/01/2026.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/CMDB/Files/GetFolderIdByFolderPath

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
|folderId|string| Folder ID. |
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
full_url= nb_url + "/ServicesAPI/API/V1/CMDB/Files/GetFolderIdByFolderPath"
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
        print ("Failed to Get FolderId by folderPath! - " + str(response.text))
except Exception as e:
    print (str(e))
```
```python
{'folderId': '3db78383-8eeb-45b0-937e-9ea2986a5f50', 'statusCode': 790200, 'statusDescription': 'Success.'}
```

# cURL Code from Postman:
```python
curl -X POST \
  "http://192.168.28.44/ServicesAPI/API/V1/CMDB/Files/GetFolderIdByFolderPath" \
  -H "Content-Type: application/json" \
  -H "cache-control: no-cache" \
  -H "token: 82b932fe-6126-4eea-b200-a10f88bf6a6c"\
  -d '{ 
    "folderpath": "Public/test"
}'
```

