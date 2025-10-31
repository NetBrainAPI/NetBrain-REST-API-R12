
# Network Setting API Design

## ***GET*** /V1/CMDB/NetworkSettings
Call this API to get network settings Telnet/SSH information. 

## Detail Information

> **Title** : Get Network Settings API<br>

> **Version** : 02/24/2023.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/CMDB/NetworkSettings/{type}

> **Authentication** : 

|**Type**|**In**|**Name**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|Bearer Authentication| Headers | Authentication token | 

## Path Parameters(****required***)  
|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|type|string|The type of Network Settings:<br>telnetInfo |

## Parameters 
|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|alias|string|The alias name of telnet/SSH login credentials. |
|userName|string|The user name of the non-privilege login. |

> **Note** : If provided both of "alias" and "userName", userName has higher priority. 

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
|deviceCount | integer  | The total number of devices that using the current credential. |
|alias | string  | The alias name of telnet/SSH login credentials.  |
|userName | string  | The user name of the non-privilege login. |
|cliMode  | integer  | The authentication method. <br>Options:<br>`0`: Telnet/SSH Password<br>`2`: SSH public key |
|keyName  | string  | The name of the SSH public key. This field is only available when the current cliMode is 2 in system. |
|statusCode| integer | Code issued by NetBrain server indicating the execution result.  |
|statusDescription| string | The explanation of the status code. |

## Response Codes 
|**Code**|**Message**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|790200|ok| Success |
|793404|NotFound|Not found |

> ***Example***

```python
{
    "telnetInfo"[
        {
            "alias":"Test1",
            "userName":"netbrain",
            "cliMode":2,
            "KeyName":"testKey",
            "deviceCount":25
        },
        {
            "alias":"Test2",
            "userName":"netbrain",
            "cliMode":0,
            "deviceCount":15
        }

    ]
    'statusCode': 790200, 
    'statusDescription': 'Success.'
}
```

# Full Example:
## Example 1
```python
full_url = nb_url + "/ServicesAPI/API/V1/CMDB/NetworkSettings"
headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["Token"] = token

data = {
    "type" : "telnetInfo"
}

try:
    response = requests.get(full_url, params = data, headers = headers, verify = False)
    if response.status_code == 200:
        result = response.json()
        print (result)
    else:
        print ("Failed to Get Network Settings! - " + str(response.text))
    
except Exception as e:
    print (str(e))  

```
```python
    {
        "telnetInfo": [
            {
            "deviceCount": 2,
            "alias": "ABC",
            "username": "netbrain",
            "cliMode": 0
            },
            {
            "deviceCount": 3,
            "alias": "ROOT",
            "username": "root",
            "cliMode": 0
            }
        ],
        "statusCode": 790200,
        "statusDescription": "Success."
    }
```
## Example 2
```python
token = "9308f7b2-8d0b-40be-92ba-5e0cde30b9a5"
full_url = nb_url + "/ServicesAPI/API/V1/CMDB/NetworkSettings/"

headers["token"] = token

body = {
    "alias": "Viptela",
}

try:
    response = requests.get(full_url, params=body, headers=headers, verify=False)
    if response.status_code == 200:
        result = response.json()
        print (result)
    else:
        print ("Failed to Get Network Settings! - " + str(response.text))
    
except Exception as e:
    print (str(e)) 
```
```python
{
  "telnetInfo": [
    {
      "deviceCount": 0,
      "alias": "Viptela",
      "username": "admin",
      "cliMode": 0
    }
  ],
  "statusCode": 790200,
  "statusDescription": "Success."
}
```
# cURL Code from Postman

```
curl -X GET \
  'https://nextgen-training.netbrain.com/ServicesAPI/API/V1/CMDB/NetworkSettings' \
  -H 'cache-control: no-cache' \
  -H 'token: f602bc7c-b5d6-47bb-8498-59b2e29aa7ed'
  -d '{
    "type" : "telnetInfo"
}'
```