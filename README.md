# sRV
Smart Radiator Valve controllable with MQTT

First you will need to add your unique credentials for connection to your WiFi and MQTT broker.

```Python
const char* ssid = "your SSID";
const char* password = "your SSID password";
const char* mqtt_server = "mqtt broker IP";
const char* mqttUser = "mqttUsername";
const char* mqttPassword = "mqttPassword";
```

You also need to change these to match the 'MQTT Switch' component in Home Assistant. These need to be unique for each sRV.

```Python
char* sRV_room = "sRV/move";
char* sRV_room_state = "sRV/state";
```

The "clientid" also needs to be unique for each sRV. I just add a letter to the end of ESP8266Client.

```Python
char* clientid = "ESP8266Client";
```
