
# File Download API Design

## ***GET*** /V1/CMDB/Files/BinaryFiles/download/{fileIdOrFolderId}
This API is used to Download Files.

## Detail Information

> **Title** : Download Files API<br>

> **Version** : 29/01/2026.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/CMDB/Files/BinaryFiles/download/{fileIdOrFolderId}

> **Authentication** : 

|**Type**|**In**|**Name**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|Bearer Authentication| Headers | Authentication token | 

## Request body(****required***)
>No parameters required.


## Query Parameters(****required***)
|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|fileIdOrFolderId* | string | ID of the file or folder. This can be retrieved from `Get File ID List by Folder Path API`, or `Get Folder ID by Folder Path API`. |

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
When successful, the API call will download the file.

# Full Example: 
The file will be downloaded in current directory where the script is run, as `downloaded_file.bin`.

```python
# import python modules 
import requests
import time
import urllib3
import pprint
import json
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

token = "005fd6cc-cf08-4742-985b-902503dad2a4"
fileIdOrFolderId = "a5627950-462d-4d9d-8274-040c7ac666be"

full_url = nb_url + f"/ServicesAPI/API/V1/CMDB/Files/BinaryFiles/download/{fileIdOrFolderId}"
headers = {
    'Content-Type': 'application/json',
    'Accept': 'application/octet-stream',
    'Token': token
}

try:
    response = requests.get(full_url, headers=headers, verify=False)
    if response.status_code == 200:
        # Save to a local file, or choose a preferrable location, name, etc.
        with open("downloaded_file.bin", "wb") as f:
            f.write(response.content)
        print("Downloaded Files Successfully!")
    else:
        print("Failed to Download Files! - " + response.text)
except Exception as e:
    print(str(e))
```

# cURL Code from Postman:
```python
curl -X GET \
  "http://192.168.28.44/ServicesAPI/API/V1/CMDB/Files/BinaryFiles/download/a5627950-462d-4d9d-8274-040c7ac666be" \
  -H "Content-Type: application/json" \
  -H "cache-control: no-cache" \
  -H "token: 82b932fe-6126-4eea-b200-a10f88bf6a6c"\
```

