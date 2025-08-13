
# Transfer Tenants from One FSC to Another API Design

## ***POST*** API/V1/CMDB/fsc/transfer-tenants
This API is used to transfer tenants from one FSC (Front Server Controller) or FSC group to another.
This API allows migrations of all tenants under one specific FSC to the destination FSC.

<b>Note</b>: Logically, FS and FSC are directly connected through tenants, therefore when the FSC associated with a tenant changes, the related FS is also migrated.

## Detail Information

> **Title** : Transfer Tenants from One FSC to Another<br>

> **Version** : 13/08/2025

> **API Server URL** : http(s):// IP address of your NetBrain Web API Server/ServicesAPI/API/V1/CMDB/fsc/transfer-tenants

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
|sourceFSC*|string| Tenant names to migrate |
|targetFSC*|string| Name of the target FSC or FSC group of the tenant(s) migration |


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
full_url = nb_url + "/ServicesAPI/API/V1/CMDB/fsc/transfer-tenants"
headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["Token"]=token

data = {
    "sourceFSC": "FSC_Cluster_In_West_America",
    "targetFSC": "FSC_New_Cluster_In_East_America"
}

try:
    response = requests.post(full_url, data = json.dumps(data), headers = headers, verify = False)
    if response.status_code == 200:
        result = response.json()
        print (result)
    else:
        print("Failed to Transfer Tenants from one FSC or FSC Group to Another! - " + str(response.text))
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
  http://192.168.30.172/ServicesAPI/API/V1/CMDB/fsc/transfer-tenants \
  -H 'Content-Type: application/json' \
  -H 'cache-control: no-cache' \ 
  -H 'token: f06ebc8b-48de-42ca-975d-4e59da760d7d' \
-d '{
    "sourceFSC": "FSC_Cluster_In_West_America",
    "targetFSC": "FSC_New_Cluster_In_East_America"
}'
```
