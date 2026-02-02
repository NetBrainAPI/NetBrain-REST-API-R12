
# File Download API Design

## ***POST*** /V1/CMDB/Files/
This API is used to Get All List of Files.

## Detail Information

> **Title** : Get All File List API<br>

> **Version** : 03/11/2019.

> **API Server URL** : http(s)://< IP address of NetBrain Web API Server >/ServicesAPI/API/V1/CMDB/Files/

> **Authentication** : 

|**Type**|**In**|**Name**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|Bearer Authentication| Headers | Authentication token | 

## Request body(****required***)
|**Name**|**Type**|**Description**|
|------|------|------|
| folderId | string  | ID of the folder from which you want to get the files.<br> Root folder (public folder) will be returned if folderId is null. |
| fileTypes* | array  | Types of file to retrieve. <br> ▪ `0`: Folder<br> ▪ `11`: Map<br> ▪ `21`: Dashboard ▪ `999` - Windows files |

> ***Example:***


```python
{ 
    "folderId": "", 
    "fileTypes": [0, 11 ,21] 
}
```


## Parameters(****required***)

> No parameters required.

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
|items| array of object | List of folders and files. |
|id| string | The ID of a folder in the file tree. |
|name| string | Name of a file. |
|originalId| string | The ID of a specific dashboard or file. <br> It is used for Map or Dashboard type only. |
|type | integer | Retrieved file types. <br> ▪ `0`: Folder<br> ▪ `11`: Map<br> ▪ `21`: Dashboard<br> ▪ `999` - Windows files |
|statusCode| integer | Code issued by NetBrain server indicating the execution result.  |
|statusDescription| string | The explanation of the status code. |

> ***Example***


```python
{ 
    "items": 
    [ 
        { 
            "id":"ad09aa07-b31d-4f42-a0aa-319697825b09", 
            "name":"Public/Site Maps", 
            "type":0 
        }, 
        { 
            "originalId":"75ff3cdf-dff4-48c6-a736-7a86e4374a29", 
            "id":"2a19165f-a4a5-4488-ac5d-acdf9e287ed6", 
            "name":"Public/New Folder/New Folder/New Map",
            "type":11 
        }, 
        { 
            "originalId":"d2650deb-5276-44cb-be21-43e2b129380a", 
            "id":"a84cdca3-3710-47b1-b037-665e38fd6d08", 
            "name":"Public/New Folder(1)/New Map", 
            "type":11 
        } 
    ], 
    "statusCode":790200, 
    "statusDescription":"Success." 
}
```

# Full Example:
```python
# import python modules 
import requests
import time
import urllib3
import pprint
import json
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# Set the request inputs
token = "cdba9af6-1f4d-45d0-8933-7ee38c3223b1"
full_url= nb_url + "/ServicesAPI/API/V1/CMDB/Files/"
headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["Token"]=token

body = { 
    "folderId": "", 
    "fileTypes": [0, 11 ,21] 
}

try:
    response = requests.post(full_url, headers = headers, data = json.dumps(body), verify = False)
    if response.status_code == 200:
        result = response.json()
        print (result)
    else:
        print ("Failed to Get All File List! - " + str(response.text))
except Exception as e:
    print (str(e))
```
```python
{
  "items": [
    {
      "id": "eda5e839-067d-485b-b4c0-a493281c1b43",
      "name": "Public/OpenTopo",
      "type": 0
    },
    {
      "id": "3db78383-8eeb-45b0-937e-9ea2986a5f50",
      "name": "Public/test",
      "type": 0
    },
    {
      "id": "911d31ec-a35b-4238-8d29-315fea952c73",
      "name": "Public/OpenTopo/Input",
      "type": 0
    },
    {
      "id": "46d5b216-355c-4dba-97e9-358daf92d7b5",
      "name": "Public/OpenTopo/Output",
      "type": 0
    },
    {
      "originalId": "10b2ae17-b02a-4340-b843-465e018db82a",
      "id": "078cb2a9-c4e8-41b8-acb6-2af1c339af4d",
      "name": "Public/test/Map3 (1) (1)",
      "type": 11
    }
  ],
  "statusCode": 790200,
  "statusDescription": "Success."
}
```

# cURL Code from Postman
```python
curl -X POST \
  http://192.168.28.44/ServicesAPI/API/V1/CMDB/Files/ \
  -H 'Content-Type: application/json' \
  -H 'cache-control: no-cache' \
  -H 'token: 6feda8f4-7a92-4e83-9703-40ac18484b5b' \
  -d '{ 
    "folderId": "", 
    "fileTypes": [0, 11 ,21] 
}'
```
