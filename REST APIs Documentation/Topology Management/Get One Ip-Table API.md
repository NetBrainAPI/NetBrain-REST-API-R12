
# One Ip Table API Design

## ***GET*** /V1/CMDB/Topology/OneIPTable{?Ip}&{?beginIndex}&{?count}
This API is used to get the One-IP Table.

If user provides an input value of `ip` attribute, then this API will return all items which have the same ip address in One-IP Table;

If user sets `ip = null` or `ip = "" `, but provides the input values of `beginIndex` and `count`, API will return One-IP Table with items number equal to `count` values start from `beginIndex`. <br>
But note that the default maximum value of `count` is 100,000. So if the input value of `count` is greater than 100,000, API will only return 100,000 itmes. If start from `beginIndex` to the end of table, but there are not enough count items, API will return the rest of items.

>**Note:** The One-IP table records the physical connections for all IP addresses in your workspace. It is retrieved during the Layer 2 topology discovery. One-IP table can be used to troubleshoot any Layer 2 connection issues.

## Detail Information

> **Title** : Get One-Ip Table API<br>

> **Version** : 02/06/2019.

> **API Server URL** : http(s):// IP address of your NetBrain Web API Server /ServicesAPI/API/V1/CMDB/Topology/OneIPTable

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
| ip | string  | The IP address of the current device. If the user provides an input value of `ip` attribute, then this API will return all items with the same IP address in One-IP Table. |
|lan|string|The LAN Segment of the IP address. If the user provides an input value of `lan` attribute, then this API will returns all items with the same LAN segement in One-IP Table.|
|mac|string|The MAC address related to the IP address. If the user provides an input value of `mac` attribute, then this API will return all items with the same MAC address in One-IP Table.|
|switch_name|string|The switch name connected to the end system. If the user provides an input value of `switch_name` attribute, then this API will return all items with the same switch name in One-IP Table.|
|switch_port|string|The [fullname](https://www.netbraintech.com/docs/ie71/help/index.html?interface-name-translation.htm) of switchport to connected to the end system or the device interface configured with this IP address. This is not an independent attribute; to use this attribute, `switch_name` is required.|
|dns|string|The resolved DNS name of the end system, or the combination of the device name and interface name. If the DNS name is not resolved, it is null.|
| beginIndex | int | Beginning index of data; API will return OneIP Table items starting from `beginIndex`. <br>Default: `0` |
| count | int | Count number of returned data; API will return OneIP Table items with the total number of items as the value of `count`. <br> Maximum: 100,000. <br>API will only return 100,000 itmes even if the input value of `count` is greater than 100,000. If the total number of items which start from `beginIndex` to the end of table is less than `count` value, API will return the rest of items.<br> ***Note:*** If customer inserts the `beginindex` as 1 and `count` as 100,002 (total number will greater than 10000), then API will return an error without any result returned. |

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
| token* | string  | Authentication token, get from login API. |

## Response

|**Name**|**Type**|**Description**|
|------|------|------|
|<img width=100/>|<img width=100/>|<img width=500/>|
|OneIPList| list of object | list of OneIP item  |
|OneIPList.lanSegment| string | IP subnet |
|OneIPList.ip| string | IP address |
|OneIPList.mac| string | MAC address of which the IP is on  |
|OneIPList.devName| string | Device name of which the IP is on  |
|OneIPList.interfaceName| string | Device interface name of the IP  |
|OneIPList.switchName| string | Switch name in which the IP is connected to.  |
|OneIPList.portName| string | Switch port name in which the IP is connected to  |
|OneIPList.alias| string | Device interface is HSRP/GLBP/VRRP |
|OneIPList.dns| string | DNS of the IP  |
|OneIPList.sourceDevice| string | The source device which this One IP come from |
|OneIPList.serverType| int | Device Type |
|OneIPList.switchType| int | Swtich type |
|OneIPList.updateTime| DataTime | The update time of the One IP  |
|OneIPList.userFlag| int | one ip type<br><br>USERFLAG_AUTO = 0,<br>USERFLAG_MANUAL = 1,<br>OUTSIDE_ANOYMOUS_SOURCE = 2,<br>AUTO_CDPLLDP_TABLE = 5,<br>AUTO_MAC_TABLE = 6,<br>AUTO_ARP_TABLE = 7,<br>AUTO_CDPLLDP_MAC_TABLE = 8,<br>AUTO_DEVICE_INTERFACE = 9,<br>USERFLAG_DRIVER = 10,<br>USERFLAG_VALID_FLAGS_END,<br>USERFLAG_ABNORMAL_FLAGS_START = 0X7FFFFF00,<br>UNSIGNED_USERFLAG = USERFLAG_ABNORMAL_FLAGS_START+1,<br>ERR_USERFLAG ,<br>USERFLAG_ABNORMAL_FLAGS_END,|
|OneIPList.source| string | userFlag tos string<br><br> "Auto", //USERFLAG_AUTO<br>"Manual",//USERFLAG_MANUAL<br>"Provide Outside", //OUTSIDE_ANOYMOUS_SOURCE<br>"NDP Table", //AUTO_CDPLLDP_TABLE<br>"MAC Table",//AUTO_MAC_TABLE<br>"ARP Table", //AUTO_ARP_TABLE<br>"NDP & MAC table",//AUTO_CDPLLDP_MAC_TABLE<br>"Device Interface",//AUTO_DEVICE_INTERFACE<br>"Driver"//USERFLAG_DRIVER |
|OneIPList.vendor| string | Device vendor |
|OneIPList.descr| string | Description of the switch port  |
|OneIPList.vlanId| string | The vlan of the entry learned on the switch  |
|OneIPList.vlanGroup| string | VLAN Group of the device assigned to the same LAN  |
|time| DataTime | The time of last update of the device configuration. |
|statusCode| integer | Code issued by NetBrain server indicating the execution result.  |
|statusDescription| string | The explanation of the status code. |

> ***Example***


```python
{
    "OneIPList": [
        {
            "lanSegment": "123.20.1.8/29",
            "ip": "123.20.1.11",
            "mac": "AABB.CC80.1300",
            "devName": "SW6",
            "interfaceName": "Vlan66",
            "switchName": "",
            "portName": "",
            "alias": "",
            "dns": "SW6.Vlan66",
            "sourceDevice": "SW6",
            "serverType": 2001,
            "switchType": 2001,
            "updateTime": "2019-02-01T19:13:05Z",
            "userFlag": 9,
            "source": "Device Interface",
            "vendor": "",
            "descr": ""
        }
    ],
    "statusCode": 790200,
    "statusDescription": "Success."
}
```

# Full Examples:


```python
# import python modules 
import requests
import time
import urllib3
import pprint
import json
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# Set the request inputs
token = "220d6462-ba64-4058-83cb-affb2d55de78"
nb_url = "http://192.168.28.79"
full_url = nb_url + "/ServicesAPI/API/V1/CMDB/Topology/OneIPTable"
headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
headers["Token"] = token

ip = "123.20.1.11"
beginIndex = 0
count = 5

data = {
    "ip" : ip,
    "beginIndex" : beginIndex,
    "count" : count
}

try:
    response = requests.get(full_url, params = data, headers = headers, verify = False)
    if response.status_code == 200:
        result = response.json()
        print (result)
    else:
        print ("Get One-Ip Table failed! - " + str(response.text))
    
except Exception as e:
    print (str(e))  

```

    {'OneIPList': [{'lanSegment': '192.168.180.6/32', 'ip': '192.168.180.6', 'mac': '0050.56be.10f6', 'devName': 'UCSPE', 'interfaceName': 'Network adapter 1','switchName': '', 'portName': '', 'alias': '', 'dns': 'UCSPE.Network adapter 1', 'sourceDevice': 'UCSPE', 'serverType': 13002, 'switchType': 1009, 'gateway': 'UCSPE.Network adapter 1', 'vlanId': '', 'vlanGroupId': 'UCSPE##Network adapter 1', 'updateTime': '2024-02-27T21:33:59Z', 'userFlag': 9, 'source': 'Device Interface', 'vendor': 'VMware, Inc.', 'descr': '', 'extSwitchPorts': []}], 'statusCode': 790200, 'statusDescription': 'Success.'}
    

# cURL Code from Postman:


```python
curl -X GET \
  'http://192.168.31.191/ServicesAPI/API/V1/CMDB/Topology/OneIPTable' \
  -H 'cache-control: no-cache' \
  -H 'token: fb1e1360-f3c9-4197-929b-886a146d6bdf'
```

# Error Examples：
<!-- ## Error Example 1: Empty Inputs
```python
Input:
    
        ip = ""
        count = None # cannot be null.
        beginIndex = None # cannot be null.
          
Response:
    
    "Get One-Ip Table failed! - 
    {"statusCode":791000,"statusDescription":"Null parameter: the parameter 'BeginIndex(int)' cannot be null."}

    "Get One-Ip Table failed! - 
    {"statusCode":791000,"statusDescription":"Null parameter: the parameter 'Count(int)' cannot be null."}"

 -->

## Error Example 1: Wrong Input Value Types
```python
Input 1:
    
        ip = ""
        count = "100" # Should be integer
        beginIndex = "50" # Should be integer
          
Response 1:

    {'OneIPList': [], 'statusCode': 790200, 'statusDescription': 'Success.'}

```
```python
Input 2:
    
        ip = "hahahahaah" # There is no Ip address like this
        count = 100 
        beginIndex = 50 
          
Response 2:
    
    Get One-Ip Table failed! - {"statusCode":791001,"statusDescription":"Invalid parameter: the parameter 'IP' is invalid."}
    
```
## Error Example 2: `count` > 100,000
```python
Input:
    
        ip = "" 
        count = 1000000 # count must be between 0 to 100,000
        beginIndex = 50 
          
Response:
    
    Get One-Ip Table failed! - {"statusCode":791002,"statusDescription":"Count can between 0 and 100000"}

    
```
## Error Example 3: `beginIndex` > size of One-Ip Table
```python
Input:
    
        ip = "" 
        count = 1 
        beginIndex = 99999999999 # There are only 117 items in this table.
          
Response:
    
    Get One-Ip Table failed! - {"statusCode":791009,"statusDescription":"The parameter 'BeginIndex' is invalid value."}

    
```
