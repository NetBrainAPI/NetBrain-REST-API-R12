
# Device API Design

## ***GET*** /V1/CMDB/DeviceGroups
Call this API to export the device's current baseline data table

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
| hostname | string  | Hostname of the device |
| ip | string  | Management IP address of the device |
| tableName | string  | Name of the Device Data Table; <br>1. routeTable <br>2. arpTable <br>3. cdpTable <br>4. stpTable <br>5.bgpNbrTable <br>... <br>(for NCT Table, please use the real tabe name. e.g. Qos Mapping Table)|
| vrf | string  | Name of VRF, if applicable; otherwise, empty |
| subTableName | string  | Name of NCT Table's sub-table name, if applicable; otherwise, empty |
| pageIndex | integer  | Page Index for gets & sets. Starts from 0 |
| pageSize | integer  | Page size for gets & sets. <br>If PageSize equals 0, PageIndex must be 0; This will return all rows |


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
|statusCode| integer | Code issued by NetBrain server indicating the execution result  |
|statusDescription| string | The explanation of the status code |
|columns| list | List of columns according to the Data Table |
|rows| nested list | List(s) of values of respective columns, according to the Data Table |
|isAllLoaded | boolean | Indicates if partial of full rows of Data Table are loaded |

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
