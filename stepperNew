/*
 Basic ESP8266 MQTT example
 This sketch demonstrates the capabilities of the pubsub library in combination
 with the ESP8266 board/library.
 It connects to an MQTT server then:
  - publishes "hello world" to the topic "outTopic" every two seconds
  - subscribes to the topic "inTopic", printing out any messages
    it receives. NB - it assumes the received payloads are strings not binary
  - If the first character of the topic "inTopic" is an 1, switch ON the ESP Led,
    else switch it off
 It will reconnect to the server if the connection is lost using a blocking
 reconnect function. See the 'mqtt_reconnect_nonblocking' example for how to
 achieve the same result without blocking the main loop.
 To install the ESP8266 board, (using Arduino 1.6.4+):
  - Add the following 3rd party board manager under "File -> Preferences -> Additional Boards Manager URLs":
       http://arduino.esp8266.com/stable/package_esp8266com_index.json
  - Open the "Tools -> Board -> Board Manager" and click install for the ESP8266"
  - Select your ESP8266 in "Tools -> Board"
*/

#include <ESP8266WiFi.h>
#include <PubSubClient.h>
#include <Stepper.h>

const int stepsPerRevolution = 200;  // change this to fit the number of steps per revolution
// for your motor

// initialize the stepper library on pins D1 through to D4
Stepper myStepper(stepsPerRevolution, D1, D2, D3, D4);

// Update these with values suitable for your network.

const char* ssid = "your SSID";
const char* password = "your SSID password";
const char* mqtt_server = "mqtt broker IP";
const char* mqttUser = "mqttUsername";
const char* mqttPassword = "mqttPassword";

// Change these to match the 'MQTT Switch' component in Home Assistant. These need to be unique for each sRV.

char* sRV_room = "sRV/move";
char* sRV_room_state = "sRV/state";

// This needs to be unique for each sRV. I just add a letter to the end of ESP8266Client.
char* clientid = "ESP8266Client";

WiFiClient espClient;
PubSubClient client(espClient);
long lastMsg = 0;
char msg[50];
int value = 0;

//Setup Variables
String switch1;
String strTopic;
String strPayload;
char* sRV_topic = sRV_room;
char* sRV_state = "OPEN";
char* last_state = "";

void setup() {
  // Initialise pin that controls power to the stepper motor driver board
  // Set it to HIGH so motor driver board is on and stepper can move to contact position
  pinMode(D8, OUTPUT);
  digitalWrite(D8, HIGH);
  delay(1000);

  // set the speed at 60 rpm
  myStepper.setSpeed(60);
  Serial.begin(115200);

  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);

  // Go home
  Serial.println("home");
  myStepper.step(-170);
  delay(1000);

  // Contact
  Serial.println("contact");
  myStepper.step(60);
  delay(1000);

  digitalWrite(D8, LOW);
  delay(1000);

  // Set sRV state to OPEN
  Serial.println("open");
  sRV_state = "OPEN";
  delay(200);
  client.publish(sRV_room_state, sRV_state, true);
}

void setup_wifi() {

  delay(10);
  // We start by connecting to a WiFi network
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length) {
  //if the 'sRV/move' topic has a payload "ON", then action the stepper motor in
  payload[length] = '\0';
  strTopic = String((char*)topic);
  if (strTopic == sRV_topic)
  {
    switch1 = String((char*)payload);
    if (switch1 == "ON" and sRV_state == "CLOSED")
    {
      // Turn motor driver board ON
      digitalWrite(D8, HIGH);
      delay(1000);

      // Retract:
      myStepper.step(-100);
      delay(1000);

      // Turn motor driver board off again
      digitalWrite(D8, LOW);

      // Set sRV state to OPEN
      Serial.println("open");
      sRV_state = "OPEN";
      delay(200);
      client.publish(sRV_room_state, sRV_state, true);
      delay(1000);

    }
    else if (switch1 == "OFF" and sRV_state == "OPEN")
    {
      // Turn motor driver board ON
      digitalWrite(D8, HIGH);
      delay(1000);

      // close sRV moves the actuator 2 steps at a time, any faster and the stepper will slip
      closesRV();

      // Turn motor driver board off again
      digitalWrite(D8, LOW);

      // Set sRV state to CLOSED
      sRV_state = "CLOSED";
      delay(200);
      client.publish(sRV_room_state, sRV_state, true);
      delay(1000);
    }   
  }
}

void closesRV() {
  //Extend actuator
  for (int i = 0; i < 52; i++)
  {
    Serial.println("close");
    Serial.println(i);
    myStepper.step(2);
    delay(200);
  }
}

void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Attempt to connect
    if (client.connect(clientid, mqttUser, mqttPassword)) {
      Serial.println("connected");
      // Once connected, publish an announcement...
      client.publish("outTopic", "hello world");
      // ... and resubscribe
      client.subscribe(sRV_room, 1);
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}
void loop() {

  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  long now = millis();
  if (now - lastMsg > 2000) {
    lastMsg = now;
    ++value;
    snprintf (msg, 75, sRV_state, value);
    Serial.print("Publish message: ");
    Serial.println(msg);
    client.publish(sRV_room_state, sRV_state, true);
  }
}
