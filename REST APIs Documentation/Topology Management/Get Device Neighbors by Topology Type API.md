
# Topology API Design

## ***GET*** /V1/CMDB/Topology/Devices/Neighbors
This API returns the neighbor relationships of devices in the current working domain. For each connection, it returns the neighbor device's hostname and the interface names on both ends of the interface pair.<br><br>
**Note: The API follows the privilege control of NB system. If there is restriction set by Access Control Policy for the target querying resources, the response will not return queried data.**
<br><br>

<b>Important</b>: This page describes the `version=1` design of `GET /V1/CMDB/Topology/Devices/Neighbors` - the recommended format.<br><br>

It is recommended to pass parameter <i>version=1</i> instead of <i>version=0</i>
<br>
For legacy `version=0` design, see [Get Device Neighbors by Topology Type API_Version_0](https://github.com/NetBrainAPI/NetBrain-REST-API-R12.3/blob/main/REST%20APIs%20Documentation/Topology%20Management/Get%20Device%20Neighbors%20by%20Topology%20Type_Version_0.md)
.
### Version interaction between URL and request body
Starting from R12, the API supports multiple versions. The version segment `V` in the URL path (e.g. <b>V1</b> in API/V1/CMDB/Topology/Devices) identifies the NetBrain API version, and also controls the shape of the response payload.

If the <i>query parameter `version`</i> is <b>not</b> provided, the `V` in the URL path takes precedence and determines the fields of the response using the latest version logic.

If the <i>query parameter `version`</i> is provided, the query parameter version takes precedence over the URL path when selecting the response format. <br>
&nbsp; When version=0 is used, the response falls back to the old (pre-multi-version) logic. <br><br>
As a result, the shape of the API response can differ based on the passed version and query parameters.

## Detail Information

> **Title** : Get Device Neighbors by Topology Type API<br>

> **Version** : 04/09/2026.

> **API Server URL** : http(s)://IP address of NetBrain Web API Server/ServicesAPI/API/V1/CMDB/Topology/Devices/Neighbors

> **Authentication** : 

|**Type**|**In**|**Name**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|Bearer Authentication| Headers | Authentication token | 

## Request body(****required***)

>No request body.

## Query Parameters(****required***)

> **Passing multiple values:** `hostname` and `topoType` each accept multiple values. Pass them as repeated query parameters - one occurrence of the parameter per value. A JSON-style array in the query string is not supported.
>
> Supported: `?hostname=r1&topoType=1&topoType=2`
>
> Not supported: `?hostname=["r1"]&topoType=[1,2]` - a bracketed array containing more than one value returns `500 Internal Server Error`.
>
> If you build the request in Python with `requests`, a list in `params` is serialized into the repeated form automatically: `params={"topoType": [1, 2]}` is sent on the wire as `topoType=1&topoType=2`. The Python examples below rely on this behavior.

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|||* - required <br>^ - optional|
|hostname^ | list of string  | The device name. Repeat the parameter once per device. <br>e.g. `hostname=US-BOS-R1`, or `hostname=US-BOS-R2&hostname=US-BOS-R3&hostname=US-BOS-R4`|
|topoType* | list of int | Returns the neighbors in specified topology types:<br> `1`: `L3_Topo_Type`, <br>`2`: `L2_Topo_Type`, <br>`3`: `Ipv6_L3_Topo_Type`, <br>`4`: `L3_VPN_Topo_Type`, <br>`5`: `L2_Overlay_Topo_Type` <br>Repeat the parameter once per value. <br>e.g. `topoType=1`, or `topoType=2&topoType=3&topoType=4`. |
|version^ | string | This is a minor version number. Value of this parameter is `1`.|
|skip^|integer|The amount of device records to be skipped. <br>The value cannot be negative. If the value is negative, API throws exception `{"statusCode":791001,"statusDescription":"Parameter 'skip' cannot be negative"}`. <br> No upper bound for this parameter.<br><br> default value: `0`|
|limit^|integer|The up limit amount of device records to return per API call. <br>The range of this parameter: `10`-`100`. Otherwise, API throws exception `{"statusCode":791001,"statusDescription":"Parameter 'limit' must be greater than or equal to 10 and less than or equal to 100"}`. <br><br> default value: `50`|
|||If only `skip` is provided, returns the rest of the full device list. <br>If only `limit` is provided, returns from the first device in DB. <br>If both `skip` and `limit` are provided, returns as required. Error exceptions follow each parameter's descriptions.<br><br>**Note:** The `skip` and `limit` parameters are based on the device search result, not topology result record.|
|filterBus^ | boolean | Without this filter (or when set to `False`), the API returns a list of all neighbors. <br> With this filter set to `True`, the API does not return the group in same link (uplink or downlink) within the same media. |

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
** Null values of the response properties will be ignored; hence in some cases, some properties will not be contained.
|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|hostdevice | list of object | List of object content detailed information of neighbor base on this device. |
|hostdevice.interface | string | Interface name of the interface that belongs to `hostdevice`. |
|hostdevice.connected_device | object | Detailed information of neighbor device that connects to `hostdevice`. |
|hostdevice.connected_device.nbr_device| string | The hostname of the neighbor device that connect to `hostdevice`.|
|hostdevice.connected_device.inteface_name| string | The interface name of the neighbor device that connects to the `hostdevice` interface.|
|hostdevice.topology | string | The topology name in which this neighbor device belongs to, such as `L2_Topo_Type`, `L3_Topo_Type`, `Ipv6_L3_Topo_Type`, `VPN_Topo_Type` or `L2_Overlay_Topo_Type`. |
|statusCode| integer | The returned status code of executing the API.  |
|statusDescription| string | The explanation of the status code.  |


# Full Examples:

## Example 1:
```python
# import python modules 
import requests
import time
import urllib3
import pprint
import json
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# Set the request inputs
token = "e9c7af7c-eedd-40fb-8b4b-356974a12b91"
nb_url = "http://192.168.28.143"
full_url = nb_url + "/ServicesAPI/API/V1/CMDB/Topology/Devices/Neighbors"
headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["Token"] = token

hostnames = ["US-ORD-EDGE-SW1"]
topoTypes = [1]

data = {
        "hostname" : hostnames,
        "topoType" : topoTypes,
        "version": "1"
    }

try:
    response = requests.get(full_url, params = data, headers = headers, verify = False)
    if response.status_code == 200:
        result = response.json()
        print (result)
    else:
        print ("Failed to Get Neighbors by Topology! - " + str(response.text))
    
except Exception as e:
    print (str(e)) 
```
```python
{
  "topology": [
    {
      "hostname": "US-ORD-EDGE-SW1",
      "neighbors": [
        {
          "interface": "Ethernet1/3 1.1.1.1/22",
          "media": "1.1.1.1/22",
          "topology": "L3_Topo_Type",
          "connected_device": {
            "nbr_device": "ABC",
            "nbr_intf": "Management1 1.1.1.1/22"
          }
        }
      ]
    }
  ],
  "statusCode": 790200,
  "statusDescription": "Success."
}
```

## Example 2: Get All Devices' Topology Information In Current Domain

```python
full_url = nb_url + "/ServicesAPI/API/V1/CMDB/Topology/Devices/Neighbors"
headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["Token"] = token

hostnames = ["US-ORD-EDGE-SW1", "ASA", 'xxx', ...] # get the list of all device hostnames in current domain by calling get device API first.
topoTypes = [1]

skip = 0
count = 50
try:
    while count == 50:
        data = {
            "hostname" : hostnames,
            "topoType" : topoTypes,
            "version": "1",
            "skip" : skip
        }
        response = requests.get(full_url, params = data, headers = headers, verify = False)
        if response.status_code == 200:
            result = response.json()
            count = len(result["topology"])
            skip = skip + count
            print (result)
	    #Un-comment the below line if want to test calling result length.
            #print (len(result['topology'])) 
        else:
            print ("Failed to Get Neighbors by Topology! - " + str(response.text))
except Exception as e:
    print (str(e)) 
```
```python
{
  "topology": [
    {
      "hostname": "ASA",
      "neighbors": [
        {
          "interface": "Ethernet1/3 1.1.1.1/24",
          "media": "1.1.1.1/24",
          "topology": "L3_Topo_Type",
          "connected_device": {
            "nbr_device": "ABC",
            "nbr_intf": "Ethernet1/3 1.1.1.1/24"
          }
        },
        {...}
      ]
    },
    {
      "hostname": "US-ORD-EDGE-SW1",
      "neighbors": [
        {
          "interface": "Ethernet1/3 1.1.1.1/22",
          "media": "1.1.1.1/22",
          "topology": "L3_Topo_Type",
          "connected_device": {
            "nbr_device": "ABC",
            "nbr_intf": "Management1 1.1.1.1/22"
          }
        },
        {...}
      ]
    }
  ],
  "statusCode": 790200,
  "statusDescription": "Success."
}
```

## Example 3: Using `filterBus`
```python
full_url = nb_url + "/ServicesAPI/API/V1/CMDB/Topology/Devices/Neighbors"
headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["Token"] = token

hostname = "BJ_L2_Core_3"

data = {
        "hostname" : hostname,
        "topoType" : [2],
        "filterBus": True
    }

try:
    response = requests.get(full_url, params = data, headers = headers, verify = False)
    if response.status_code == 200:
        result = response.json()
        print (result)
    else:
        print ("Failed to Get Neighbors by Topology! - " + str(response.text))
    
except Exception as e:
    print (str(e)) 
```
```python
{
  "topology": [
    {
      "hostname": "BJ_L2_Core_3",
      "neighbors": [
        {
          "interface": "FastEthernet1/0/12",
          "topology": "L2_Topo_Type",
          "connected_device": {
            "nbr_device": "ABC",
            "nbr_intf": "FastEthernet0/1"
          }
        },
        {
          "interface": "FastEthernet1/0/15",
          "topology": "L2_Topo_Type",
          "connected_device": {
            "nbr_device": "BCD",
            "nbr_intf": "FastEthernet0/1"
          }
        }
      ]
    }
  ],
  "statusCode": 790200,
  "statusDescription": "Success."
}
```

## Example 4: Using `version=0`; it is not recommended to use version=0
When using version=0, <i>topoType</i> takes strings, not integers.
```python
# version = 0
full_url = nb_url + "/ServicesAPI/API/V1/CMDB/Topology/Devices/Neighbors"
headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["Token"] = token

hostname = ["US-ORD-EDGE-SW1"] 
topoType = ['L3_Topo_Type'] # takes strings, not integers

data = {
        "hostname" : hostname,
        "topoType" : topoType,
        "version": 0
    }

try:
    response = requests.get(full_url, params = data, headers = headers, verify = False)
    if response.status_code == 200:
        result = response.json()
        print (result)
    else:
        print ("Failed to Get Neighbors by Topology! - " + str(response.text))
    
except Exception as e:
    print (str(e)) 
```
```python
{
  "neighbors": [
    {
      "hostname": "AAA",
      "interface": "Lab_DMZ_Vlan400 1.1.1.1/22"
    },
    {
      "hostname": "BBB",
      "interface": "GigabitEthernet7 1.1.1.2/22"
    },
    {
      "hostname": "CCC",
      "interface": "Ethernet1/3 1.1.1.3/22"
    }
  ],
  "statusCode": 790200,
  "statusDescription": "Success."
}
```

# cURL Code from Postman:


```python
curl -X GET \
  'http://192.168.28.143/ServicesAPI/API/V1/CMDB/Topology/Devices/Neighbors?hostname=BJ_Acc_SW1&topoType=1&topoType=2&version=1' \
  -H 'Postman-Token: d43de85c-8de9-4bcf-be28-9bc16ce7b329' \
  -H 'cache-control: no-cache' \
  -H 'token: 3d0f475d-dbae-4c44-9080-7b08ded7d35b'
```

# Error Examples:
## Error Example 1: Empty Inputs
```python
Input:
        hostname = [""] # Cannot be null.
        topoType = [] # Cannot be null.

Response:
    "{'statusCode': 791001, 'statusDescription': 'The parameter "topoType" is required.'}"
        
```

## Error Example 2: Wrong Inputs
```python
Input:
        hostname = "dummy" # No device with a hostname called "dummy"
        topoType = [1]

Response:
    "{'topology': [], 'statusCode': 790200, 'statusDescription': 'Success.'}"

```

## Error Example 3: Wrong topoType
```python
Input:
        hostname = "R1" 
        topoType = [7] # No topology code for 7.

Response:
    "Failed to Get Neighbors by Topology! - 
        {'statusCode': 791001, 'statusDescription': "Parameter 'TopoType' value must be greater than 0 and less than 5"}"
```
