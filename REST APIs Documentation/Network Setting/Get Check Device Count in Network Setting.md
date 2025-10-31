
# Network Setting API Design

## ***GET*** /V1/CMDB/NetworkSettings/TelnetInfo/CheckDeviceCount?alias={?aliasname}
Call this API to get the Device Count in Network Settings of Domain Management.
Use this API along with API RefreshDeviceCount (Update Device Count in Network Setting) to retrieve the latest device count info.

## Detail Information

> **Title** : Get Latest Device Count in Network Setting<br>

> **Version** : 26/03/2024.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/CMDB/NetworkSettings/TelnetInfo/CheckDeviceCount

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
|alias*|string|The alias name of telnet/SSH login credentials. |
|limit|integer|The up limit amount of device records to return per API call. The "limit" value valid range is 10 - 100, if the assigned value exceeds the range, the server will respond error message "Parameter 'limit' must be greater than or equal to 10 and less than or equal to 100".  The default value of limit is 50. |
|index (Skip)|integer|The index number is used to indicate the last record scanned in DB from the last API call. The value of this parameter is the last API call returned index number . For example, the last call returned index is 1000, then provide parameter value as 1000 to avoid duplicate record compare with the last API call. This is a required parameter if amount of records need to be skipped.  |
|           |           |Limit and index parameters are based on the search result from DB. If both limit and index are not provided, return the device list with 50 devices start from the first device result in DB. If only the limit value is provided, return from the first device result in DB. If only the index value is provided, return the device list with 50 devices start from the index number. If both limit and index values are provided, return as required. Error exceptions follow each parameter's description. |
|           |           |The abnormal value of limit/index can cause parameter error, i.e. value equal to 'abc' or 1000000000000. If value is abnormal, API throws an exception {"statusCode":791001,"statusDescription":"The value of parameter 'alias' is invalid"}. The value of limit/index cannot be negative. If the value is negative, API throws an exception {"statusCode":791001,"statusDescription":"Parameter 'index' cannot be negative"}. If the alias is failed to be found, API throws an exception {"statusCode":791001,"statusDescription":"ParameterNotFound"}. |
|checkDeviceCount|string|Used to retrieve the latest information of Device Count of each alias. |


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
|alias| string | The alias name of Telnet/SSH Login Credentials.  |
|deviceCount| integer | Latest device count of each alias in Telnet/SSH Login.  |
|index| integer | The index number of last record scanned in DB from the current API call.  |
|statusCode| integer | The returned status code of executing the API.  |
|statusDescription| string | The explanation of the status code.  |

> ***Example***


```python
{'alias': 'nb', 'deviceCount': 9, 'devices': [{'hostname': "!@#$%^&*()_-=+~`:;.'|\\/[]{}", 'vendor': 'Cisco', 'model': '3560E', 'managementIp': '172.25.52.9'}, {'hostname': 'ASA', 'vendor': 'Cisco', 'model': '3560E', 'managementIp': '172.25.52.1'}, {'hostname': 'ASA,Router', 'vendor': 'Cisco', 'model': '3560E', 'managementIp': '172.25.52.2'}, {'hostname': 'ASA-Router', 'vendor': 'Cisco', 'model': '3560E', 'managementIp': '172.25.52.3'}, {'hostname': 'ASA.Switch', 'vendor': 'Cisco', 'model': '3560E', 'managementIp': '172.25.52.6'}, {'hostname': 'ASA@Switch', 'vendor': 'Cisco', 'model': '3560E', 'managementIp': '172.25.52.7'}, {'hostname': 'ASA\\Router', 'vendor': 'Cisco', 'model': '3560E', 'managementIp': '172.25.52.4'}, {'hostname': 'firewall', 'vendor': 'Cisco', 'model': 'ASA', 'managementIp': '158.4.0.29'}, {'hostname': 'sw-hec01-fwt-01a', 'vendor': 'Juniper', 'model': 'SRX', 'managementIp': '158.8.0.81'}], 'index': 497, 'statusCode': 790200, 'statusDescription': 'Success.'}
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
full_url = nb_url + "/ServicesAPI/API/V1/CMDB/NetworkSettings/TelnetInfo/CheckDeviceCount"
headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["Token"] = token

alias = "nb"

body = {
        "alias":alias,
    }

try:
    response = requests.get(full_url, params=body, headers=headers, verify=False)
#     response = requests.get(full_url, headers=headers, verify=False)
    if response.status_code == 200:
        result = response.json()
        print (result)
    else:
        print ("Failed to Get Check Device Count in Network Setting! - " + str(response.text))
    
except Exception as e:
    print (str(e)) 
```

    {'alias': 'nb', 'deviceCount': 9, 'devices': [{'hostname': 'ASA', 'vendor': 'Cisco', 'model': '3560E', 'managementIp': '172.25.52.1'}, {'hostname': 'ASA,Router', 'vendor': 'Cisco', 'model': '3560E', 'managementIp': '172.25.52.2'}, {'hostname': 'ASA-Router', 'vendor': 'Cisco', 'model': '3560E', 'managementIp': '172.25.52.3'}, {'hostname': 'ASA.Switch', 'vendor': 'Cisco', 'model': '3560E', 'managementIp': '172.25.52.6'}, {'hostname': 'ASA@Switch', 'vendor': 'Cisco', 'model': '3560E', 'managementIp': '172.25.52.7'}, {'hostname': 'firewall', 'vendor': 'Cisco', 'model': 'ASA', 'managementIp': '158.4.0.29'}, {'hostname': 'sw-hec01-fwt-01a', 'vendor': 'Juniper', 'model': 'SRX', 'managementIp': '158.8.0.81'}], 'index': 497, 'statusCode': 790200, 'statusDescription': 'Success.'}
    

# cURL Code from Postman


```python
curl -X GET \
  'http://192.168.31.191/ServicesAPI/API/V1/CMDB/NetworkSettings/TelnetInfo/CheckDeviceCount?alias=nb' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'token: b0181119-7b1b-4faf-be97-b8566a39a640'
```
