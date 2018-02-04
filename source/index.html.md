---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - <a href='mailto:quentin@watereen.com'>Contact us</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  - errors
  - sensor_identifiers

search: true
---

# Introduction

Welcome to the Watereen API documentation page ! You can use our API to access Watereen gateway API endpoints, which can get information on the network status and all the sensors related data.
You can view the Curl code example related to each endpoint in the dark area to the right, you just have to copy-paste the command in a bash terminal to test it.

Please free to contact us for any information you may need by sending us an [email](mailto:quentin@watereen.com).

# Authentification

> Once logged in, use this code to authorize:

```shell
# With shell, you can just pass the correct header with each request
curl "https://demo.watereen.com/api/endpoint" \
  -H "Authorization: Bearer mytoken"
```

> Make sure to replace `mytoken` with your JWT.

The Watereen gateway API uses JWT (JSON Web Token) to allow access to the API. To get a JWT, you first need to login with your credentials as explained in the following subsection.
If you don't have a Watereen account, you can request one for testing purposes by contacting us.

Once you are logged in and for each subsequent API request, you need to include the JWT in a header that looks like the following:
`Authorization: Bearer mytoken`

<aside class="notice">
You must replace <code>mytoken</code> with your personal JWT.
</aside>

<aside class="warning">
The JWT issued by the server is only valid during 30 minutes. After this period, you will need to log in again
</aside>

## Get a JSON Web Token (JWT)

```shell
curl "https://demo.watereen.com/api/login_jwt" \
  -H "Content-Type: application/json" \
  -d '{"email":"youremail@provider.com", "password":"your_password"}'
```

> If your credentials are correct, the above command returns JSON structured like this:

```json
{
  "first_name" : "My_first_name",
  "last_name" : "My_last_name",
  "jwt" : "mytoken",
  "valid" : true
}
```

> Keep the jwt value safe, this is the token needed for all subsequent requests

This endpoint authenticates the user and returns back the JWT.

### HTTP Request

`POST https://demo.watereen.com/api/login_jwt - 200 OK`

### POST Parameters

Parameter | Type    | Description
--------- | ------- | -----------
email     | string  | Email associated to your Watereen account
password  | string  | Account password

<aside class="warning">
Be sure to set the <code>Content-Type</code> header to <code>application/json</code> for every POST request
</aside>

### Response Parameters
Parameter   | Type    | Description
----------- | ------- | -----------
first_name  | string  | First name as registered in your Watereen account
last_name   | string  | Last name
jwt         | string  | Token, valid for the next 30 minutes
valid       | boolean | Indicates if the authentication succeeded

# Gateway configuration

## Get gateway informations

```shell
curl "https://demo.watereen.com/api/gateway_informations" \
  -H "Authorization: Bearer mytoken"
```

> The above command returns JSON structured like this:

```json
{
  "appkey" : "0AB",
  "isApMode" : boolean,
  "connectedInterface": "eth0" or "wlan0",
  "routerIp": "192.168.1.1",
  "routerAccessible": boolean,
  "internetAccessible" : boolean
}
```

This endpoint returns technical informations and the status of the gateway.

### HTTP Request

`GET https://demo.watereen.com/api/gateway_informations - 200 OK`

### Response Parameters
Parameter           | Type      | Description
------------------- | --------- | -----------
appkey              | string    | Watereen radio application key - A unique gateway ID
isApMode            | boolean   | Indicates if the gateway is in Access-Point mode (share its own Wifi network), or is connected to a local network
connectedInterface  | string    | Name of the network interface in use. `wlan0` means a Wifi connection, `eth0` an ethernet connection
routerIp            | string    | IP address of the local network router
routerAccessible    | boolean   | Indicates if the LAN router is turned on
internetAccessible  | boolean   | Indicates if the gateway has an internet access

## Get the nearby Wifi networks list

```shell
curl "https://demo.watereen.com/api/networks_list" \
  -H "Authorization: Bearer mytoken"
```

> The above command returns JSON structured like this:

```json
[
  {
    "ssid" : "Livebox",
    "power" : "-8",
    "encryption" : "RSN (AES)",
    "power_percentage" : 100
  },
  {
    "ssid" : "Freebox",
    "power" : "-60",
    "encryption" : "WPA",
    "power_percentage" : 65
  }
]
```

This endpoint returns all the Wifi networks that the gateway detects and their associated technical informations.

### HTTP Request

`GET https://demo.watereen.com/api/networks_list - 200 OK`

### Response Parameters - Array
Parameter         | Type    | Description
----------------- | ------- | -----------
ssid              | string  | Network SSID
power             | string  | WiFi signal strength in dBm
encryption        | string  | Encryption method
power_percentage  | number  | Estimated power in percentage - 0 means a very weak signal, 100 a strong one

## Switch to Access-Point mode

```shell
curl "https://demo.watereen.com/api/switch_to_ap" -X POST \
  -H "Authorization: Bearer mytoken"
```

> The above command does not return anything

This endpoint will switch the gateway in Access-Point mode. If it was connected to a LAN network, the connection will be lost and an access point with the SSID `Watereen gateway` will be turned on. You will need to connect to this network with your client to continue with the API.

### HTTP Request

`POST https://demo.watereen.com/api/switch_to_ap - 204 OK No Content`

<aside class="notice">
A delay of ~1 minute before the Wifi network is accessible may be observable after the server response.
</aside>

## Switch to Wifi mode (LAN)

```shell
curl "https://demo.watereen.com/api/switch_to_wifi" \
  -H "Authorization: Bearer mytoken" \
  -H "Content-Type: application/json" \
  -d '{"ssid":"wifi_ssid", "password":"wifi_password"}'
```

> The above command does not return anything

This endpoint will ask the gateway to connect to the provided network. If the operation fails, the gateway will automatically switch back to the AP mode.

### HTTP Request

`POST https://demo.watereen.com/api/switch_to_wifi - 204 OK No Content`

### POST Parameters

Parameter     | Type    | Description
------------- | ------- | -----------
ssid          | string  | SSID of the targeted Wifi network
password      | string  | Its associated key/password

<aside class="warning">
Ensure that you will be able to find the DHCP given IP address of the gateway on the local network or you may have difficulty to recover an access.
</aside>

# Sensors and data

## Retrieve the sensors

```shell
curl "https://demo.watereen.com/api/sensors" \
  -H "Authorization: Bearer mytoken"
```

> The above command returns JSON structured like this:

```json
[
  {
    "id" : 2,
    "name" : "Wheat field",
    "battery" : 3.5,
    "packet_lost" : 0
  },
  {
    "id" : 3,
    "name" : "Barley field 2",
    "battery" : 3.6,
    "packet_lost" : 0
  }
]
```

This endpoint returns all the Wifi networks that the gateway detects and their technical informations.

### HTTP Request

`GET https://demo.watereen.com/api/sensors - 200 OK`

### Response Parameters - Array
Parameter     | Type      | Description
------------- | --------- | -----------
id            | number    | Unique node identifier
name          | string    | Registered name of the sensor
battery       | number    | Sensor battery voltage (V)
packet_lost   | number    | Number of radio packet lost by this sensor

## Gather the sensors data

> The -g Curl option is necessary when the URL includes square brackets

```shell
curl -g "https://demo.watereen.com/api/data?date[gt]=2018-02-04T12:00:00.000Z&date[lt]=2018-02-04T13:29:00.000Z" \
  -H "Authorization: Bearer mytoken"
```

> The above command returns JSON structured like this:

```json
[
  {
    "node_id" : 2,
    "date" : "2018-02-04T13:00:00.000Z",
    "packet_informations":
    {
      "rssi" : -103,
      "snr" : 6,
      "packet_loss" : 0
    },
    "measurements":
    [
      {
        "value" : 3,
        "sensor_identifier" : 0
      },
      {
        "value" : 59,
        "sensor_identifier" : 1
      }
    ]
  },
  {
    "node_id" : 3,
    "date" : "2018-02-04T13:00:00.000Z",
    "packet_informations":
    {
      "rssi" : -96,
      "snr" : 6,
      "packet_loss" : 0
    },
    "measurements":
    [
      {
        "value" : 33,
        "sensor_identifier" : 0},
      {
        "value" : 89,
        "sensor_identifier" : 1
      }
    ]
  }
]
```

This endpoint returns the data collected by the sensors. You can specify a sensor id as well as a date range. By default, it will return every sensor measurements received in the last day (Now - 24h).

### HTTP Request

`GET https://demo.watereen.com/api/data/<SENSOR_ID> - 200 OK`

### Response Parameters - Array
Parameter                                             | Type        | Description
----------------------------------------------------- | ----------- | -----------
node_id                                               | number      | Unique node identifier identifying the source of these measurements
date                                                  | date        | Measurements date in ISO 8601 format
packet_informations                                   | object      | Contains all radio-specific informations
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rssi              | number      | Received Signal Strength Indication
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;snr               | number      | Signal-to-Noise Ratio
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;packet_loss       | number      | Packet(s) lost before this one
measurements                                          | array       | Contains the value measured by each physical sensor
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sensor_identifier | number      | [Physical sensor identifiers]("#physical_sensor_identifiers")
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;value             | number      | Value measured

### URL Parameters
Parameter             | Default   | Description
--------------------- | --------- | -----------
SENSOR_ID - Optional  | None      | Set a specific node identifier

### Query Parameters
Parameter   | Default           | Description
----------- | ----------------- | -----------
date[gt]    | Now - 24 Hours    | Only data received after this date are returned (ISO 8601)
date[lt]    | Now               | Only data received before this date are returned (ISO 8601)
