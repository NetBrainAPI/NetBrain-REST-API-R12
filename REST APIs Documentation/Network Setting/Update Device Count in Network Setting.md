
# Network Setting API Design

## ***GET*** /V1/CMDB/NetworkSettings/TelnetInfo/RefreshDeviceCount
Call this API to update (refresh) the Device Count in Network Settings of Domain Management.
Use this API along with API CheckDeviceCount (Get Device Count in Network Setting) to retrieve the latest device count info.

## Detail Information

> **Title** : Update Device Count in Network Setting<br>

> **Version** : 26/03/2024.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/CMDB/NetworkSettings/TelnetInfo/RefreshDeviceCount

> **Authentication** : 

|**Type**|**In**|**Name**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|Bearer Authentication| Headers | Authentication token | 

## Request body(****required***)

>No parameters required.

## Parameters(****required***)

>No parameters required.

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
|statusCode| integer | The returned status code of executing the API.  |
|statusDescription| string | The explanation of the status code.  |

> ***Example***


```python
{
    'statusCode': 790200, 
    'statusDescription': 'Success.'
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
token = "9c717c9a-4302-45b5-a068-2a3e9c4ea1a3"
nb_url = "http://192.168.28.79"
full_url = nb_url + "/ServicesAPI/API/V1/CMDB/NetworkSettings/TelnetInfo/RefreshDeviceCount"
headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["token"] = token

try:
    response = requests.get(full_url, headers=headers, verify=False)
    if response.status_code == 200:
        result = response.json()
        print (result)
    else:
        print ("Failed to Update Device Count in Network Setting! - " + str(response.text))
    
except Exception as e:
    print (str(e)) 
```

    {'statusCode': 790200, 'statusDescription': 'Success.'}
    

# cURL Code from Postman

```python
curl -X GET \
  http://192.168.31.191/ServicesAPI/API/V1/CMDB/NetworkSettings/TelnetInfo/RefreshDeviceCount \
  -H 'Content-Type: application/json' \
  -H 'cache-control: no-cache' \ 
  -H 'token: b0181119-7b1b-4faf-be97-b8566a39a640' \
```