
# Get Search Results API Design

## ***GET*** V3/CMDB/Search
Call this API to get the list of device names with the matching search keyword from NetBrain IE.

## Detail Information

> **Title** : Get Search Results API<br>

> **Version** : 03/24/2025.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V3/CMDB/Search

> **Authentication** : 

|**Type**|**In**|**Name**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|Bearer Authentication| Headers | Authentication token | 

## Request body(****required***)

>No parameters required.

## Parameters(****required***)
|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|limit|integer|The up limit amount of device records to return per API call. The "limit" value valid range is 10 - 100, if the assigned value exceeds the range, the server will respond error message "Parameter 'limit' must be greater than or equal to 10 and less than or equal to 100." The value cannot be negative. If the value is negative, API throws exception {"statusCode":791001,"statusDescription":"Parameter 'limit' cannot be negative"}. |
|index|integer|The index number is used to indicate the last record scanned in DB from the last API call. The value of this parameter is the last API call returned index number plus 1. For example, the last call returned index is 1000, then provide parameter value as 1001 to avoid duplicate record compare with the last API call. This is a required parameter if amount of records need to be skipped. The value cannot be negative. If the value is negative, API throws exception {"statusCode":791001,"statusDescription":"Parameter 'index' cannot be negative"}. |
|||limit and index parameters are based on the search result from DB. If both limit and index are not provided, return the device list with 50 devices start from the first device result in DB. If only provide limit value, return from the first device result in DB. If only provide index value, return the device list with 50 devices start from the index number. If provided both limit and index, return as required. Error exceptions follow each parameter's description. |
|keyword|string|Search keyword inputted by user. Max length: 10 chars, rest will be ignored. This keyword cannot be empty. |

<!-- |type|integer|Search type, device search value is 1. Other values can be considered for expansion in the future. | -->

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
|searchResult| object | List of search results. This result contains the device name.|
|devices| array of objects | List of searched devices. |
|deviceName| string | Name of each device. |
|statusCode| integer | The returned status code of executing the API.  |
|statusDescription| string | The explanation of the status code.  |

> ***Example***

```python
{
    "searchResult": {
        "devices": [
            {
                "deviceName": "BJ*POP"
            },
            {
                "deviceName": "BJ_core_3550"
            },
            {
                "deviceName": "BJ_Acc_Sw4-bbb-eee-ii-JJJJ-II1-NNNN-GGGGG-000-BBBBB-0OOO-SSS18"
            },
            {
                "deviceName": "bjta002444-SW14"
            },
            {
                "deviceName": "bjta002237-SW2"
            },
            {
                "deviceName": "BJ-R3"
            },
            {
                "deviceName": "BJ-Cat-5000"
            },
            {
                "deviceName": "BJ-Arista-1"
            },
            {
                "deviceName": "BJ_Dis_SW2"
            },
            {
                "deviceName": "BJ_Acc_SW2-bbb-eee-ii-JJJJ-II1-NNNN-GGGGG-000-BBBBB-0OOO-SSS18-TTTT-OOOO0-NNNNNN-MMM-DDDD-UUUUUo-KKKK-MMMM-RRRR-EEEEE-TTTT99999999999999999999999999999999999ddd-MMM-JJJJ-II1-NNNN-GGGGG-000-BBBBB-0OOO-SSS18-TTT"
            }
        ]
    },
    "statusCode": 790200,
    "statusDescription": "Success."
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
nb_url = "http://192.168.1.1"
full_url = nb_url + "/ServicesAPI/API/V3/CMDB/Search"
headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["token"] = token

data = {
    "limit": 10,
    "index": 0,
    "keyword": "bj"
}

try:
    response = requests.get(full_url, headers = headers, params = data, verify = False)
    if response.status_code == 200:
        result = response.json()
        print (result)
    else:
        print ("Failed to Get Search Results! - " + str(response.text))
    
except Exception as e:
    print (str(e)) 
```

# cURL Code from Postman

```python
curl -X GET \
  'http://192.168.1.1/ServicesAPI/API/V3/CMDB/Search?limit=10&index=0&keyword=bj' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'token: 34edefc9-5c33-48a3-955d-0d9744d9fe9d'
```