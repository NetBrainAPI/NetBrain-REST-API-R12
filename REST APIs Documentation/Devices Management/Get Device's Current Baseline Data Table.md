
# Device API Design

## ***GET*** /V1/CMDB/Devices/DeviceTableData
Call this API to export the device's current baseline data table.

## Detail Information

> **Title** : Export (GET) Device's Current Baseline Data Table<br>

> **Version** : 25/04/2025

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/CMDB/Devices/DeviceTableData

> **Authentication** : 

|**Type**|**In**|**Name**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|Bearer Authentication| Headers | Authentication token | 

## Request body(****required***)

> No request body.


## Query Parameters(****required***)

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
| hostname | string  | Hostname of the device. |
| ip | string  | Management IP address of the device. |
|  |  | *At least one of `hostname` or `ip` should have value. <br> They cannot both be empty. |
| tableName* | string  | Name of the Device Data Table. <br>Names for System Tables are constant; <br>1. routeTable <br>2. arpTable <br>3. macTable  <br>4. cdpTable <br>5. stpTable <br>6. bgpNbrTable <br> <br>For NCT Table, please use the real tabe name. e.g. QoS Mapping Table|
| vrf^ | string  | Name of VRF, if applicable; otherwise, empty. |
| subTableName^ | string  | Name of NCT Table's sub-table name, if applicable; otherwise, empty. |
| pageIndex* | integer  | Page Index for gets & sets. Starts from 0. |
| pageSize* | integer  | Page size for gets & sets. <br>If PageSize equals 0, PageIndex must be 0; This will return all rows. |
|  |  | ^ indicates optional parameter. <br>* indicates mandatory parameter.  |


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
|columns| list | List of columns according to the Data Table. |
|rows| nested list | List(s) of values of respective columns, according to the Data Table. |
|isAllLoaded | boolean | Indicates if partial of full rows of Data Table are loaded. |
|statusCode| integer | Code issued by NetBrain server indicating the execution result.  |
|statusDescription| string | The explanation of the status code. |

> ***Example***
```python
{
    "columns": [
        "Alg.",
        "Dest.Addr",
        "Mask",
        "Distance",
        "Metric",
        "Interface",
        "Next Hop IP",
        "Next Hop Device",
        "Age"
    ],
    "rows": [
        [
            "O E2",
            "192.168.29.0",
            24,
            110,
            300,
            "FastEthernet0/0",
            "172.24.32.11",
            "BJ_core_3550",
            "4d13h"
        ],
        [
            "O E2",
            "1.1.1.1",
            32,
            110,
            200,
            "FastEthernet0/0",
            "172.24.32.11",
            "BJ_core_3550",
            "4d13h"
        ]
    ],
    "isAllLoaded": false,
    "statusCode": 790200,
    "statusDescription": "Success."
}
```

# Full Example
```python
# import python modules 
import requests
import time
import urllib3
import pprint
import json
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# Set the request inputs
token = "8cf08bd4-92ef-4f4f-b56f-d6dc95124f51"
full_url = nb_url + "/ServicesAPI/API/V1/CMDB/Devices/DeviceTableData"

headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["Token"] = token

hostname = "F5-MGMT"
ip = ''
tableName = 'routeTable'
vrf = ''
subTableName = ''
pageIndex = 0
pageSize = 0

body = {
        "hostname":hostname,
        "ip":ip,
        "tableName":tableName,
        "vrf":vrf,
        "subTableName":subTableName,
        "pageIndex":pageIndex,
        "pageSize":pageSize,
    
    }
try:
    response = requests.get(full_url, headers=headers, params = body, verify=False)
    print(response)
    if response.status_code == 200:
        result = response.json()
        print (result)
    else:
        print ("Failed to get Device's Current Baseline Data Table! - " + str(response.text))
    
except Exception as e:
    print (str(e)) 
```

```
{
    "columns": [
        "Alg.",
        "Dest.Addr",
        "Mask",
        "Distance",
        "Metric",
        "Interface",
        "Next Hop IP",
        "Next Hop Device",
        "Age"
    ],
    "rows": [
        [
            "O E2",
            "1.1.1.1",
            32,
            110,
            200,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "9.8.130.0",
            24,
            110,
            20,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "10.10.0.0",
            16,
            110,
            300,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "1w5d"
        ],
        [
            "O E2",
            "10.10.0.0",
            22,
            110,
            200,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "10.10.4.0",
            22,
            110,
            200,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "10.10.8.0",
            22,
            110,
            200,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "10.10.12.0",
            22,
            110,
            200,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "10.10.16.0",
            22,
            110,
            200,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "10.10.24.0",
            22,
            110,
            200,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "10.10.32.0",
            22,
            110,
            200,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "10.10.36.0",
            22,
            110,
            200,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "10.99.0.0",
            16,
            110,
            200,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "10.99.97.0",
            24,
            110,
            200,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "158.9.0.0",
            16,
            110,
            300,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "158.15.5.0",
            24,
            110,
            300,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "158.15.6.0",
            24,
            110,
            300,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "158.99.0.0",
            16,
            110,
            300,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "172.21.3.0",
            24,
            110,
            20,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "172.24.10.0",
            30,
            110,
            300,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "172.24.10.8",
            29,
            110,
            300,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "172.24.10.32",
            27,
            110,
            300,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O",
            "172.24.10.224",
            27,
            110,
            13,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "172.24.30.0",
            30,
            110,
            300,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "172.24.30.4",
            30,
            110,
            200,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "1w5d"
        ],
        [
            "O",
            "172.24.31.0",
            26,
            110,
            151,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "172.24.31.64",
            26,
            110,
            200,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "1w5d"
        ],
        [
            "O E2",
            "172.24.31.192",
            26,
            110,
            200,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O",
            "172.24.32.0",
            30,
            110,
            87,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O",
            "172.24.32.4",
            30,
            110,
            23,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O",
            "172.24.32.8",
            29,
            110,
            13,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "172.24.32.16",
            29,
            110,
            200,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "172.24.32.48",
            29,
            110,
            200,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O",
            "172.24.32.64",
            26,
            110,
            23,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "172.24.32.208",
            28,
            110,
            200,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "172.24.32.224",
            28,
            110,
            200,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O",
            "172.24.33.64",
            26,
            110,
            88,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O",
            "172.24.33.128",
            26,
            110,
            87,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O",
            "172.24.36.0",
            29,
            110,
            13,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O",
            "172.24.36.8",
            29,
            110,
            14,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O",
            "172.24.36.16",
            29,
            110,
            14,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O",
            "172.24.100.0",
            30,
            110,
            12,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O",
            "172.24.101.0",
            24,
            110,
            11,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "172.24.252.11",
            32,
            110,
            200,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "172.24.253.12",
            32,
            110,
            300,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "172.24.255.2",
            32,
            110,
            200,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "1w5d"
        ],
        [
            "O",
            "172.24.255.3",
            32,
            110,
            88,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "172.24.255.4",
            32,
            110,
            200,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "172.24.255.5",
            32,
            110,
            200,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O",
            "172.24.255.7",
            32,
            110,
            24,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "172.24.255.8",
            32,
            110,
            200,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O",
            "172.24.255.9",
            32,
            110,
            88,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O",
            "172.24.255.10",
            32,
            110,
            14,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "172.24.255.12",
            32,
            110,
            200,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "1w5d"
        ],
        [
            "O E2",
            "172.24.255.20",
            32,
            110,
            200,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "1w5d"
        ],
        [
            "O E2",
            "172.24.255.51",
            32,
            110,
            300,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "172.24.255.52",
            32,
            110,
            300,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O",
            "172.24.255.61",
            32,
            110,
            11,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O",
            "172.24.255.62",
            32,
            110,
            12,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O",
            "172.25.3.0",
            24,
            110,
            11,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O",
            "172.25.5.0",
            24,
            110,
            11,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O",
            "172.25.6.0",
            24,
            110,
            11,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "C",
            "172.25.7.0",
            24,
            0,
            0,
            "Ethernet0/0",
            "",
            "",
            ""
        ],
        [
            "L",
            "172.25.7.250",
            32,
            0,
            0,
            "Ethernet0/0",
            "",
            "",
            ""
        ],
        [
            "O",
            "172.25.8.0",
            24,
            110,
            11,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O",
            "172.25.16.0",
            24,
            110,
            11,
            "Ethernet0/0",
            "172.25.7.254",
            "Multicast-MGMT",
            "7w0d"
        ],
        [
            "O",
            "172.25.50.0",
            24,
            110,
            12,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O",
            "172.25.51.0",
            24,
            110,
            12,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O",
            "172.25.52.0",
            24,
            110,
            12,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O",
            "172.25.54.0",
            24,
            110,
            12,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O",
            "172.25.94.0",
            24,
            110,
            12,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O",
            "172.25.99.0",
            24,
            110,
            11,
            "Ethernet0/0",
            "172.25.7.2",
            "",
            "7w0d"
        ],
        [
            "O",
            "172.25.111.0",
            24,
            110,
            11,
            "Ethernet0/0",
            "172.25.7.11",
            "VXLAN-MGMT",
            "7w0d"
        ],
        [
            "O",
            "172.25.116.0",
            24,
            110,
            12,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O",
            "172.25.117.0",
            24,
            110,
            12,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O",
            "172.25.120.0",
            24,
            110,
            12,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O",
            "172.25.121.0",
            24,
            110,
            12,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "C",
            "172.25.124.0",
            24,
            0,
            0,
            "Vlan1",
            "",
            "",
            ""
        ],
        [
            "L",
            "172.25.124.254",
            32,
            0,
            0,
            "Vlan1",
            "",
            "",
            ""
        ],
        [
            "O E2",
            "172.26.3.0",
            25,
            110,
            20,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "172.26.4.0",
            24,
            110,
            20,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "172.26.5.0",
            24,
            110,
            20,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "172.26.6.0",
            24,
            110,
            20,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "172.26.7.0",
            24,
            110,
            20,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "172.27.1.0",
            24,
            110,
            300,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "192.168.4.0",
            24,
            110,
            300,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "192.168.20.0",
            24,
            110,
            300,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "192.168.28.0",
            24,
            110,
            300,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "192.168.29.0",
            24,
            110,
            300,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "192.168.30.0",
            24,
            110,
            300,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "192.168.31.0",
            24,
            110,
            300,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "192.168.31.11",
            32,
            110,
            300,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "192.168.32.0",
            24,
            110,
            300,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "192.168.33.0",
            24,
            110,
            300,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "192.168.48.0",
            24,
            110,
            300,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ],
        [
            "O E2",
            "192.168.75.0",
            24,
            110,
            300,
            "Ethernet0/0",
            "172.25.7.1",
            "BJ_L2_Core_5",
            "7w0d"
        ]
    ],
    "isAllLoaded": true,
    "statusCode": 790200,
    "statusDescription": "Success."
}
```

# cURL Code from Postman
```python
http://10.99.98.3:81/ServicesAPI/API/V1/CMDB/Devices/DeviceTableData?hostname=F5-MGMT&ip=&tableName=routeTable&vrf=&subTableName=&pageIndex=0&pageSize=0
```

