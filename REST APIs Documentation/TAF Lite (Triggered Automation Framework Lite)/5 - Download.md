
# Download API Design

## ***GET*** V3/download?dl_ticket={downloadTicketId}
When the intent result data from [API](https://github.com/NetBrainAPI/NetBrain-REST-API-R12.1/blob/main/REST%20APIs%20Documentation/TAF%20Lite%20(Triggered%20Automation%20Framework%20Lite)/3%20-%20Get%20NI%20running%20results%20of%20Trigger%20Task.md) `API/V3/TAF/Lite/result/datas` is too large to return, a `downloadTicketId` will be returned instead.
Use `downloadTicketId` in this API `API/V3/download?dl_ticket={downloadTicketId}` to download the result zip file. <br>
PA license is <b>not</b> required to use this API.

## Detail Information

> **Title** : Download Intent Result Data in zip file<br>

> **Version** : 17/07/2024

> **API Server URL** : http(s):// IP address of your NetBrain Web API Server/ServicesAPI/API/V3/download?dl_ticket={downloadTicketId}

> **Authentication** : 

|**Type**|**In**|**Name**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|Bearer Authentication| Headers | Authentication token | 

## Request body(****required***)
|**Name**|**Type**|**Description**|
|------|------|------|
|downloadTicketId*|string|The ID returned by the system when the result data is too large. <br> generated from API `API/V3/TAF/Lite/result/datas`|

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
|taskId|string|taskID executed by this Trigger|
|<img width=100/>|<img width=100/>|<img width=500/>|
|statusCode| integer | The returned status code of executing the API.  |
|statusDescription| string | The explanation of the status code.  |

> ***Example***


```python
{
  "niId": "4be2334a-5608-4b2c-809b-a72da260ece7",
  "niName": "DrawMap30002",
  "niResultId": "efba5e9e-a3d0-4d83-8aa4-42f0e3c29e7a",
  "timePoint": "2024-06-20T02:17:40.853Z",
  "statusCodes": [
    "GigabitEthernet0/0.4 is interface"
  ],
  "csvs": [],
  "rawDatas": [
    {
      "deviceName": "GW2Lab",
      "command": "show interface",
      "rawData": "GW2Lab>show interface\r\nGigabitEthernet0/1 is up, line protocol is up \r\n  Hardware is iGbE, address is f44e.051e.b600 (bia f44e.051e.b600)\r\n  MTU 1500 bytes, BW 1000000 Kbit/sec, DLY 10 usec, \r\n     reliability 255/255, txload 1/255, rxload 1/255\r\n  Encapsulation 802.1Q Virtual LAN, Vlan ID  1., loopback not set\r\n  Keepalive set (10 sec)\r\n  Full Duplex, 1Gbps, media type is RJ45\r\n  output flow-control is unsupported, input flow-control is unsupported\r\n  ARP type: ARPA, ARP Timeout 04:00:00\r\n  Last input 00:00:00, output 00:00:00, output hang never\r\n  Last clearing of \"show interface\" counters never\r\n  Input queue: 0/75/134/2307 (size/max/drops/flushes); Total output drops: 0\r\n  Queueing strategy: fifo\r\n  Output queue: 0/40 (size/max)\r\n  5 minute input rate 2722000 bits/sec, 2859 packets/sec\r\n  5 minute output rate 2427000 bits/sec, 2398 packets/sec\r\n     3360884949 packets input, 2528161170 bytes, 0 no buffer\r\n     Received 2077327882 broadcasts (0 IP multicasts)\r\n     0 runts, 0 giants, 0 throttles \r\n     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored\r\n     0 watchdog, 2562393 multicast, 0 pause input\r\n     262866017 packets output, 944523538 bytes, 0 underruns\r\n     0 output errors, 0 collisions, 0 interface resets\r\n     407895 unknown protocol drops\r\n     0 babbles, 0 late collision, 0 deferred\r\n     1 lost carrier, 0 no carrier, 0 pause output\r\n     0 output buffer failures, 0 output buffers swapped out\r\nGigabitEthernet0/0.4 is up, line protocol is up \r\n  ..."
    }
  ],
  "maps": [
    {
      "id": "c71f1fb8-3a53-4aca-a8cc-0cbd0b0db053",
      "name": "DrawMap30002",
      "url": "map.html?t=1650bf6e-67ed-3c07-3357-b070528c4b19&d=2b662e5d-fe4e-436a-8fa9-c847c1752511&id=c71f1fb8-3a53-4aca-a8cc-0cbd0b0db053&maptype=13",
      "type": "Intent Map"
    }
  ],
  "statusCode": 790200,
  "statusDescription": "Success."
}
```

# Full Example:

```python
# import python modules 
import requests
import json
import time
import requests.packages.urllib3 as urllib3
 
urllib3.disable_warnings()
 
user = "abc"
pwd = "1234"
host_url = "http://10.10.0.29"
headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}

# Get token for netbrain
def getTokens(user,password):
    login_api_url = r"/ServicesAPI/API/V1/Session"
    Login_url = host_url + login_api_url
    data = {
        "username": user,
        "password": password
    }
    token = requests.post(Login_url, data=json.dumps(
        data), headers=headers, verify=False)
    if token.status_code == 200:
        print(token.json())
        return token.json()["token"]
    else:
        print("Failed to get token with log:", token.error)
        return None


def do_download(ticketId):
 
    # Trigger  API url
    API_URL = r"/ServicesAPI/API/V3/download?dl_ticket=" + ticketId
    # Trigger API payload
    print(headers)
    api_full_url = host_url + API_URL
    api_result = requests.get(api_full_url, headers=headers, verify=False)
    file_name = download_file(api_result, ticketId)
    print(file_name)
 
 
def download_file(r, ticketId):
    local_filename = "d:\\" + ticketId + ".zip"
 
    with open(local_filename, 'wb') as f:
        for chunk in r.iter_content(chunk_size=8192):
            # If you have chunk encoded response uncomment if
            # and set chunk_size parameter to None.
            # if chunk:
            f.write(chunk)
    return local_filename
 
if __name__ =="__main__": 
    ticketId = "ZTA2MjAxYWItMTU5MC00MGU5LWExOTItNDZmNjk5Y2E5ZTI0fDYzODU1NzkzMzE2ODgyNzA5Mg=="

    try:
        # get token
        token = getTokens(user,pwd)
        if token:
            headers["Token"] = token
            print(print(do_download(ticketId)))
    except Exception as e:
        print (str(e)) 
```

# cURL Code from Postman

```python
# call login API
curl -X POST \
http://10.10.0.29/ServicesAPI/API/V1/Session \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -d '{
    "username" : "abc",
    "password" : "1234"  
}'

# call Download
curl -X GET \
  http://10.10.0.29/ServicesAPI/API/V3/download?dl_ticket=ZTA2MjAxYWItMTU5MC00MGU5LWExOTItNDZmNjk5Y2E5ZTI0fDYzODU1NzkzMzE2ODgyNzA5Mg== \
  -H "Content-Type: application/json"
  -H 'token: 32ef82d4-44a5-4b48-a089-97daf5e1d92e'
```
