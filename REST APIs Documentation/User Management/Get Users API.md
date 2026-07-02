
# User API Design

## ***GET*** /V1/CMDB/Users{?username}{&authenticationServer}
Call this API to get user information. If a username is provided, the API returns only that user's information. If no username is provided, the API returns information for all users.

## Detail Information

> **Title** : Get User(s) Information API<br>

> **Version** : 19/03/2024.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/CMDB/Users

> **Authentication** :

|**Type**|**In**|**Name**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|Bearer Authentication| Parameters | Authentication token |

## Request body(****required***)

>No request body.

## Query Parameters(****required***)

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|username | string | The name of the NetBrain system user. This field is the key to get the user information. If `username` is set to null or an empty string, the API returns information for all users. |
|authenticationServer | string | The corresponding name of the authentication server. |

> **Note:** `authenticationServer` is an optional attribute used to prevent mis-retrieving when the same user account name exists on different servers. See the example below for details.

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
|UserData| list of object | List of users info. |
|UserData.username| string | The user name. This field is the key to update the user information. |
|UserData.email| string | The email address of the user. |
|UserData.firstName| string | The first name of the user. |
|UserData.lastName| string | The last name of the user. |
|UserData.allowChangePassword| bool | Decide whether to allow the user to change his/her password independently. |
|UserData.isSystemAdmin| bool | Decide whether to allocate the system administrator role to the user. |
|UserData.isUserManager| bool | Decide whether to allocate the user manager role to the user. |
|UserData.isSystemManager| bool | Decide whether to allocate the system manager role to the user. |
|UserData.creationTime| Time | Date and time of creation. |
|UserData.lastUpdateTime| Time | Date and time of last update. |
|UserData.status| integer | Status of the user account.<br>▪ 1 - Active<br>▪ 2 - Inactive<br>▪ 3 - Pending<br>▪ 4 - Deactivated<br>▪ 5 - Unavailable |
|UserData.authenticationType| int | The authentication type for the user account.<br>▪ -2 - Token<br>▪ -1 - PortalAutoCreate<br>▪ 0 - NetBrain<br>▪ 1 - AD<br>▪ 2 - LDAP<br>▪ 3 - TACACS<br>▪ 4 - SAML |
|UserData.phoneNumber| string | The phone number of the user. |
|UserData.department| string | The department that the user belongs to. |
|UserData.description| string | Any description about the user. |
|UserData.deactivatedTime| string | Specify the time when the user account is expired. |
|UserData.TenantAndRole| list of TenantAndRole object | Specify Tenant and Role for the user.<br>▪ tenantId (string) - the ID of the tenant that the user can access.<br>▪ tenantName (string) - the name of the tenant that the user can access.<br>▪ isAdmin (bool) - decide whether to allocate the tenant administrator role to the user. If false, you need to specify a domain for the user to access.<br>▪ domains (list of objects) - the list of domains this user can access.<br>▪ canAddDomain (bool) - decide whether to allow the user to create domains. |
|users| list of object | The list containing the duplicate account information on different servers. |
|users.authenticationServer| string | The name of the authentication server. |
|users.userName| string | The name of the user account. |
|statusCode| integer | Code issued by the NetBrain server indicating the execution result. |
|statusDescription| string | The explanation of the status code. |

> ***Example***

Normal response:

```python
{
  "UserData": [
    {
      "username": "Automation@Incident",
      "description": "Automation incident portal user for logins via access code.",
      "allowChangePassword": false,
      "isSystemAdmin": false,
      "isUserManager": false,
      "isSystemManager": false,
      "lastUpdateTime": "2022-03-14T18:23:27.185Z",
      "status": 1,
      "TenantAndRole": [
        {
          "tenantId": "22dc22dc-ab2d-cd8a-60a0-b6d667d61e4c",
          "tenantName": "Patch_Tenant",
          "isAdmin": false,
          "domains": [
            {
              "id": "7dd695d2-1990-40bf-bfd2-586f794a584c",
              "name": "Patch_Domain",
              "roles": [
                "Domain User",
                "Portal Temp User"
              ]
            },
            {
              "id": "833df35f-f550-4c9f-8fb0-c208834cf617",
              "name": "AutoBasicDomain",
              "roles": [
                "Domain User",
                "Portal Temp User"
              ]
            }
          ],
          "canAddDomain": false
        }
      ]
    },
    {
      "username": "NBPatchTester",
      "email": "aabc@netbraintech.com",
      "firstName": "test",
      "lastName": "patch",
      "allowChangePassword": false,
      "isSystemAdmin": true,
      "isUserManager": true,
      "isSystemManager": true,
      "creationTime": "2022-03-22T19:52:53.605Z",
      "lastUpdateTime": "2022-03-22T19:52:53.605Z",
      "status": 1,
      "TenantAndRole": [
        {
          "tenantId": "22dc22dc-ab2d-cd8a-60a0-b6d667d61e4c",
          "tenantName": "Patch_Tenant",
          "isAdmin": true,
          "domains": [
            {
              "id": "7dd695d2-1990-40bf-bfd2-586f794a584c",
              "name": "Patch_Domain",
              "roles": [
                "Domain Admin",
                "Domain User"
              ]
            }
          ],
          "canAddDomain": true
        },
        {
          "tenantId": "9c39e0a1-1f92-4fbd-9bab-c30f517f0f6c",
          "tenantName": "authTenant",
          "isAdmin": true,
          "domains": [],
          "canAddDomain": true
        }
      ]
    },
    {
      "username": "cheng",
      "email": "cheng@netbraintech.com",
      "firstName": "pu",
      "lastName": "cheng",
      "allowChangePassword": true,
      "isSystemAdmin": true,
      "isUserManager": true,
      "isSystemManager": true,
      "creationTime": "2022-03-25T15:49:10.554Z",
      "lastUpdateTime": "2022-03-25T15:49:10.555Z",
      "status": 1,
      "TenantAndRole": [
        {
          "tenantId": "22dc22dc-ab2d-cd8a-60a0-b6d667d61e4c",
          "tenantName": "Patch_Tenant",
          "isAdmin": true,
          "domains": [
            {
              "id": "7dd695d2-1990-40bf-bfd2-586f794a584c",
              "name": "Patch_Domain",
              "roles": [
                "Domain Admin",
                "Domain User"
              ]
            },
            {
              "id": "d4c69ed9-2220-4fc9-88cb-1472b4476a91",
              "name": "TableBasedDiagnosis",
              "roles": [
                "Domain Admin",
                "Domain User"
              ]
            }
          ],
          "canAddDomain": true
        },
        {
          "tenantId": "9c39e0a1-1f92-4fbd-9bab-c30f517f0f6c",
          "tenantName": "authTenant",
          "isAdmin": true,
          "domains": [],
          "canAddDomain": true
        }
      ]
    },
    {
      "username": "ydu",
      "email": "dsa@das.com",
      "firstName": "das",
      "lastName": "dsa",
      "allowChangePassword": false,
      "isSystemAdmin": true,
      "isUserManager": true,
      "isSystemManager": true,
      "creationTime": "2022-03-25T19:18:22.32Z",
      "lastUpdateTime": "2022-03-25T19:18:22.32Z",
      "status": 1,
      "TenantAndRole": [
        {
          "tenantId": "22dc22dc-ab2d-cd8a-60a0-b6d667d61e4c",
          "tenantName": "Patch_Tenant",
          "isAdmin": true,
          "domains": [
            {
              "id": "7dd695d2-1990-40bf-bfd2-586f794a584c",
              "name": "Patch_Domain",
              "roles": [
                "Domain Admin",
                "Domain User"
              ]
            }
          ],
          "canAddDomain": true
        },
        {
          "tenantId": "9c39e0a1-1f92-4fbd-9bab-c30f517f0f6c",
          "tenantName": "authTenant",
          "isAdmin": true,
          "domains": [],
          "canAddDomain": true
        }
      ]
    },
    {
      "username": "newuser",
      "email": "test@123.com",
      "firstName": "test",
      "lastName": "test",
      "phoneNumber": "",
      "allowChangePassword": false,
      "isSystemAdmin": false,
      "isUserManager": false,
      "isSystemManager": true,
      "creationTime": "2024-03-14T21:04:26.911Z",
      "lastUpdateTime": "2024-03-14T21:04:26.911Z",
      "status": 1,
      "TenantAndRole": [
        {
          "tenantId": "9c39e0a1-1f92-4fbd-9bab-c30f517f0f6c",
          "tenantName": "authTenant",
          "isAdmin": true,
          "domains": [],
          "canAddDomain": true
        }
      ]
    }
  ],
  "statusCode": 790200,
  "statusDescription": "Success."
}
```

Response with duplicate user accounts on different servers when no authentication server is provided in the input:

```python
{
    "users": [
        {
            "authenticationServer": "NetBrain",
            "userName": "user1"
        },
        {
            "authenticationServer": "AD",
            "userName": "user1"
        }
    ],
    "statusCode": 792032,
    "statusDescription": "There are users with the same name 'user1' in the system. You need to specify the authentication server."
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
token = "9eaef30e-3a21-44b7-8600-d21625a2198e"
nb_url = "http://192.168.31.191"
full_url = nb_url + "/ServicesAPI/API/V1/CMDB/Users"
headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["Token"] = token

username = ""

data = {
        "username": username,
        "authenticationServer": "NetBrain"
    }

try:
    response = requests.get(full_url, params = data, headers = headers, verify = False)
    if response.status_code == 200:
        result = response.json()
        print (result)
    else:
        print ("Get Users Information failed! - " + str(response.text))
    
except Exception as e:
    print (str(e))
```

```python
{
  "UserData": [
    {
      "username": "Automation@Incident",
      "description": "Automation incident portal user for logins via access code.",
      "allowChangePassword": false,
      "isSystemAdmin": false,
      "isUserManager": false,
      "isSystemManager": false,
      "lastUpdateTime": "2022-03-14T18:23:27.185Z",
      "status": 1,
      "TenantAndRole": [
        {
          "tenantId": "22dc22dc-ab2d-cd8a-60a0-b6d667d61e4c",
          "tenantName": "Patch_Tenant",
          "isAdmin": false,
          "domains": [
            {
              "id": "7dd695d2-1990-40bf-bfd2-586f794a584c",
              "name": "Patch_Domain",
              "roles": [
                "Domain User",
                "Portal Temp User"
              ]
            },
            {
              "id": "833df35f-f550-4c9f-8fb0-c208834cf617",
              "name": "AutoBasicDomain",
              "roles": [
                "Domain User",
                "Portal Temp User"
              ]
            }
          ],
          "canAddDomain": false
        }
      ]
    },
    {
      "username": "ydu",
      "email": "dsa@das.com",
      "firstName": "das",
      "lastName": "dsa",
      "allowChangePassword": false,
      "isSystemAdmin": true,
      "isUserManager": true,
      "isSystemManager": true,
      "creationTime": "2022-03-25T19:18:22.32Z",
      "lastUpdateTime": "2022-03-25T19:18:22.32Z",
      "status": 1,
      "TenantAndRole": [
        {
          "tenantId": "22dc22dc-ab2d-cd8a-60a0-b6d667d61e4c",
          "tenantName": "Patch_Tenant",
          "isAdmin": true,
          "domains": [
            {
              "id": "7dd695d2-1990-40bf-bfd2-586f794a584c",
              "name": "Patch_Domain",
              "roles": [
                "Domain Admin",
                "Domain User"
              ]
            }
          ],
          "canAddDomain": true
        },
        {
          "tenantId": "9c39e0a1-1f92-4fbd-9bab-c30f517f0f6c",
          "tenantName": "authTenant",
          "isAdmin": true,
          "domains": [],
          "canAddDomain": true
        }
      ]
    },
    {
      "username": "newuser",
      "email": "test@123.com",
      "firstName": "test",
      "lastName": "test",
      "phoneNumber": "",
      "allowChangePassword": false,
      "isSystemAdmin": false,
      "isUserManager": false,
      "isSystemManager": true,
      "creationTime": "2024-03-14T21:04:26.911Z",
      "lastUpdateTime": "2024-03-14T21:04:26.911Z",
      "status": 1,
      "TenantAndRole": [
        {
          "tenantId": "9c39e0a1-1f92-4fbd-9bab-c30f517f0f6c",
          "tenantName": "authTenant",
          "isAdmin": true,
          "domains": [],
          "canAddDomain": true
        }
      ]
    }
  ],
  "statusCode": 790200,
  "statusDescription": "Success."
}
```

# cURL Code from Postman:


```python
curl -X GET \
  'http://192.168.31.191/ServicesAPI/API/V1/CMDB/Users?username=&authenticationServer=NetBrain' \
  -H 'Accept: */*' \
  -H 'Accept-Encoding: gzip, deflate' \
  -H 'Cache-Control: no-cache' \
  -H 'Connection: keep-alive' \
  -H 'Host: 192.168.28.173' \
  -H 'Postman-Token: 3cd9a344-50f6-4ae9-8c79-41a01f90ca41,e008e5f9-4fd7-4f2e-8e39-fd77d54d0651' \
  -H 'User-Agent: PostmanRuntime/7.15.2' \
  -H 'cache-control: no-cache' \
  -H 'token: b5bb9a78-f1e2-4a6a-bc67-d9ec18944ecd'
```

# Error Examples:


```python
"""Error 1: wrong inputs"""

Input:

        username = "kakakakaakak" # No user with a name called "kakakakaakak".

Response:

    "Get Users Information failed! - 
        {
            "statusCode": 791006,
            "statusDescription": "user kakakakaakak does not exist."
        }"
```
