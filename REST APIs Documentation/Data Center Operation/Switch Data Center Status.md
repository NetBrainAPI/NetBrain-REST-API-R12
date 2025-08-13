
# Switch Data Center Status API Design

## ***POST*** API/V1/data-center/opr
This API is used to switch the data center status; 1) activate, or 2) deactivate.

<b>Note</b>: Use the active data center's web service URI to deactivate it, then use the backup data center's web service URI to activate it.

## Detail Information

> **Title** : Switch Data Center Status<br>

> **Version** : 13/08/2025

> **API Server URL** : http(s):// IP address of your NetBrain Web API Server/ServicesAPI/API/V1/data-center/opr

> **Authentication** : 

|**Type**|**In**|**Name**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|Bearer Authentication| Headers | Authentication token | 

## Request body(****required***)
|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|||* - required<br />^ - optional|
|operation*|int| Operation to perform on data center. <br>0: Deactivate <br>1: Activate |
|timeout*|int| Timeout Setting for the operation, in seconds. |


## Parameters(****required***)
>No parameters required.


## Headers

> **Data Format Headers**

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
| Content-Type | string  | support "application/json" |
| Accept | string | support "application/json" |

> **Authorization Headers**

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
| token | string  | Authentication token, get from login API. |

## Response
|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|success| bool | Status of API Call |
|statusCode| integer | The returned status code of executing the API.  |
|statusDescription| string | The explanation of the status code.  |


# Full Example:
```python
full_url = nb_url + "/ServicesAPI/API/V1/data-center/opr"
headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["Token"]=token

data = {
    "operation": 0,
    "timeout": 30
}

try:
    response = requests.post(full_url, data = json.dumps(data), headers = headers, verify = False)
    if response.status_code == 200:
        result = response.json()
        print (result)
    else:
        print("Failed to Switch Data Center Status! - " + str(response.text))
except Exception as e:
    print (str(e))  
```
```python
{
  "success": true,
  "statusCode": 790200,
  "statusDescription": "Success."
}
```

# cURL Code from Postman
```python
curl -X POST \
  http://192.168.36.19/ServicesAPI/API/V1/data-center/opr \
  -H 'Content-Type: application/json' \
  -H 'cache-control: no-cache' \ 
  -H 'token: e474feaf-966c-4e09-9281-121261b6eb8d' \
-d '{
    "operation": 0,
    "timeout": 30
}'
```
