
# Get the Latest Execution Results from NI API Design

## ***POST*** V3/CMDB/NI/result
This API is used to obtain the latest execution information.

## Detail Information

> **Title** : Get the Latest Execution Results from NI<br>

> **Version** : 17/07/2024

> **API Server URL** : http(s):// IP address of your NetBrain Web API Server/ServicesAPI/API/V3/CMDB/NI/result

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
|niIdOrPath*|string| ID or path of NI|
|startTime^|DateTime|Start time of NI execution |
|endTime^|DateTime|End time of NI execution |
|output*|array|The result type returned to the third-party system <br><table><tr><th>Value</th><th>Enum</th><th>Description</th></tr> <tr><td>0</td><td>all</td><td>All types of data, except raw data</td></tr> <tr><td>1</td><td>raw_data</td><td>Raw data</td></tr> <tr><td>2</td><td>status_code</td><td>Status code and message</td></tr> <tr><td>3</td><td>csv</td><td>CSV</td></tr> <tr><td>4</td><td>map</td><td>Intent maps and external maps</td></tr> </table>|

> ***Example***
```python
{      
   niIdOrPath: 'Public\\ni1',
   output: [0, 1, 2, 3, 4] //0:all, 1: rawdata, 2: status code, 3: csv, 4: map
}
```

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
The returned NI execution result content includes Home NI and Follow NI results, hence the result is of type List.

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|[].niId|string|Intent ID|
|[].niName|string|Intent Name|
|[].niResultId|string|Intent Result ID|
|[].timePoint|string|NI execution time point|
|[].statusCodes|list|NI status codes|
|[].csvs|list of objects|NI results in CSV format|
|[].rawDatas|list of objects|Results obtained from executing commands on each device|
|[].rawDatas[].deviceName|string|Device name|
|[].rawDatas[].command|string|Command executed on the device|
|[].rawDatas[].rawData|string|Raw text data|
|[].maps|list of objects|Map results executed by NI|


> ***Example 1***
```python
[
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
        "deviceName": "GW22Lab",
        "command": "show interface",
        "rawData": "GW22Lab>show interface\r\nGigabitEthernet0/0 is up, line protocol is up \r\n  Hardware is iGbE, address is f44e.051e.b600 (bia f44e.051e.b600)\r\n  MTU 1500 bytes, BW 1000000 Kbit/sec, DLY 10 usec, \r\n     reliability 255/255, txload 1/255, rxload 1/255\r\n  Encapsulation 802.1Q Virtual LAN, Vlan ID  1., loopback not set\r\n  Keepalive set (10 sec)\r\n  Full Duplex, 1Gbps, media type is RJ45\r\n  output flow-control is unsupported, input flow-control is unsupported\r\n  ARP type: ARPA, ARP Timeout 04:00:00\r\n  Last input 00:00:00, output 00:00:00, output hang never\r\n  Last clearing of \"show interface\" counters never\r\n  Input queue: 0/75/134/2307 (size/max/drops/flushes); Total output drops: 0\r\n  Queueing strategy: fifo\r\n  Output queue: 0/40 (size/max)\r\n  5 minute input rate 2722000 bits/sec, 2859 packets/sec\r\n  5 minute output rate 2427000 bits/sec, 2398 packets/sec\r\n     3360884949 packets input, 2528161170 bytes, 0 no buffer\r\n     Received 2077327882 broadcasts (0 IP multicasts)\r\n     0 runts, 0 giants, 0 throttles \r\n     \r\n........... output buffer failures, 0 output buffers swapped out\r\n"
      }
    ],
    "maps": [
      {
        "id": "c71f1fb8-3a53-4aca-a8cc-0cbd0b0db053",
        "name": "DrawMap30002",
        "url": "map.html?t=16-67ed-3c07-3&d=2b662e5d-fe4e-436a-8fa9-c847c1752511&id=c71f1fb8-3a53-4aca-a8cc-0cbd0b0db053&maptype=13",
        "type": "Intent Map"
      }
    ],
    "statusCode": 790200,
    "statusDescription": "Success."
  }
]
```

> ***Example 2***

```python
[{
    "niId": "intent1 id",
    "niName": "intent1 name",
    "statusCodes": [
      "intent status code1",
      "intent status code2"
    ],
    "csvs": [ {
        "fileName": "csv file name1", 
        "csv": "a1,b1,c1"
      }],
    "rawDatas": [
      {
        "device": "BJ-R1", 
        "command": "show config",
        "rawData": "raw data1"
      }
    ],
    "maps": [
        {
            "id": "map id 1",
            "name": "map Name 1",
            "url": "map url 1",
            "type": "Intent Map" //Intent Map; External Map
        }
    ]
}]
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
headers1 = {'Content-Type': 'application/json', 'Accept': 'application/json'}

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


def get_ni_latest_execution_data(API_Body):
 
    # Trigger  API url
    API_URL = r"/ServicesAPI/API/V3/CMDB/NI/result"
    # Trigger API payload
    print(headers)
    api_full_url = host_url + API_URL
    api_result = requests.post(api_full_url, data=json.dumps(
        API_Body), headers=headers, verify=False)
    if api_result.status_code == 200:
        if "downloadTicketId" in api_result.json():
            ticketId = api_result.json()["downloadTicketId"]
            do_download(ticketId)
        return api_result.json()
    else:
        return api_result.json()
 
def do_download(ticketId):
 
    # Trigger  API url
    API_URL = r"/ServicesAPI/API/V3/download?dl_ticket=" + ticketId
    # Trigger API payload
    print(headers)
    api_full_url = host_url + API_URL
    api_result = requests.get(api_full_url, headers=headers, verify=False)
    download_file(api_result, ticketId)
 
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
    API_BODY = {
        "niIdOrPath": "4be2334a-5608-4b2c-809b-a72da260ece7",
        "output": [0]
    }

    try:
        # get token
        token = getTokens(user,pwd)
        if token:
            headers["Token"] = token
            print(get_ni_latest_execution_data(API_BODY))
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

# call get latest execution results from NI
curl -X POST \
  http://10.10.0.29/ServicesAPI/API/V3/CMDB/NI/result \
  -H "Content-Type: application/json"
  -H 'token: 32ef82d4-44a5-4b48-a089-97daf5e1d92e' \
  -d '{
        "niIdOrPath": "4be2334a-5608-4b2c-809b-a72da260ece7",
        "output": [0]
    }'
```
