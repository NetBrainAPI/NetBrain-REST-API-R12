
# Network Setting API Design

## ***PUT*** /V3/CMDB/NetworkSettings/CLILoginCredential
Call this API to update CLI Login Credential. <br>
Please note that this API is operated on a Domain Admin Privilege. It means that an engineer or guest user does not have the privilege to successfully call this API.
<br><br>
<b>Important</b>: Although not common, there may be cases of one username with many different passwords. However, upon calling this API, the username will be updated to have the same password.<br>
Please be mindful and use after review.

## Detail Information

> **Title** : Update CLI Login Credential API<br>

> **Version** : 04/06/2025.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V3/CMDB/NetworkSettings/CLILoginCredential

> **Authentication** : 

|**Type**|**In**|**Name**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|Bearer Authentication| Headers | Authentication token | 

## Parameters 
|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|* - mandatory field||
|username*|string|The username of CLI Login account to be modified. |
|password*|string|The password of CLI Login account to be modified. |
|updateLockedSettings|boolean|CLI/SNMP lock setting found in Shared Device Setting.<br>`True` - Force modification to locked accounts.<br>`False` (<b>default</b>) - No modification to locked accounts<br><br> If you lock your device in Shared Device Setting, but do not pass `updateLockedSettings = True`, the device setting will not be modified.|
<br>


![CLI_SNMP lock](https://github.com/NetBrainAPI/NetBrain-REST-API-R12.1/raw/main/REST%20APIs%20Documentation/Network%20Setting/NetworkSettingImages/CLISNMP_setting.png)

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
| token | string  | Authentication token, retrieved from Login API. |

## Response

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|statusCode| integer | Code issued by NetBrain server indicating the execution result.  |
|statusDescription| string | The explanation of the status code. |

## Response Codes 
|**Code**|**Message**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|790200|OK| Success |
|791006|username `xxx` does not exist| Failure |
|795003|Insufficient permissions: the current user has insufficient permissions to perform the requested operation. Sorry, you have no privilege to operate. Please contact your system administrator. required permissions: manageNetworkSettings| Failure |
|793404|NotFound|Not found |


# Examples
# Example 1: Successful API call

```python
# import python modules 
import requests
import time
import urllib3
import pprint
import json
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

token = "9c717c9a-4302-45b5-a068-2a3e9c4ea1a3"
nb_url = "http://192.168.30.191"
full_url = nb_url + "/ServicesAPI/API/V3/CMDB/NetworkSettings/CLILoginCredential"
headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["Token"] = token

data = {
    "username": "admin123",
    "password": "adminpwd",
    "updateLockedSettings": True
}

try:
    response = requests.put(full_url, json.dumps(data), headers=headers, verify=False)
    if response.status_code == 200:
        result = response.json()
        print (result)
    else:
        print ("Failed to Update CLI Login Credential! - " + str(response.text))
    
except Exception as e:
    print (str(e)) 
```
```
  {'statusCode': 790200, 'statusDescription': 'Success.'}
```

# Example 2: Username Doesn't Exist
```python
token = "68f4803c-04ee-499d-af92-3e86c7825ef7"
full_url = nb_url + "/ServicesAPI/API/V3/CMDB/NetworkSettings/CLILoginCredential"

# headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["token"] = token

data = {
    "username": "123",
    "password": "123",
    "updateLockedSettings": False
}

try:
    response = requests.put(full_url, json.dumps(data), headers=headers, verify=False)
    if response.status_code == 200:
        result = response.json()
        print (result)
    else:
        print ("Failed to Update CLI Login Credential! - " + str(response.text))
    
except Exception as e:
    print (str(e))
```
```
  Failed to Update CLI Login Credential! - {"statusCode":791006,"statusDescription":"username: 123 does not exist."}
```

# Example 3: Insufficient Privilege
```python
token = "68f4803c-04ee-499d-af92-3e86c7825ef7"
full_url = nb_url + "/ServicesAPI/API/V3/CMDB/NetworkSettings/CLILoginCredential"

# headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["token"] = token

data = {
    "username": "123",
    "password": "123",
    "updateLockedSettings": False
}

try:
    response = requests.put(full_url, json.dumps(data), headers=headers, verify=False)
    if response.status_code == 200:
        result = response.json()
        print (result)
    else:
        print ("Failed to Update CLI Login Credential! - " + str(response.text))
    
except Exception as e:
    print (str(e)) 
```
```
  Failed to Update CLI Login Credential! - {"statusCode":795003,"statusDescription":"Insufficient permissions: the current user has insufficient permissions to perform the requested operation. Sorry, you have no privilege to operate. Please contact your system administrator. required permissions: manageNetworkSettings"}
```

# cURL Code from Postman
This cURL command is based on Example 1.

```python
curl -X PUT \
  'http://192.168.30.191/ServicesAPI/API/V3/CMDB/NetworkSettings/CLILoginCredential' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'token: ec415367-60f2-41de-90a3-58cd72bfcfc6'
  -d '{
    "username" : "admin123",
    "password" : "adminpwd",
    "updateLockedSettings" : true,
}'
```
