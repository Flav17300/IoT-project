# IoT-project
projet connexion wifi ESP32/Raspberry PI

#include <WiFi.h> // Enables the ESP32 to connect to the local network (via WiFi)
#include <PubSubClient.h> // Connect and publish to the MQTT broker
// WiFi
const char* ssid = "Redmi Note 12 Pro+ 5G"; // Your personal network SSID
const char* wifi_password = "mp3instru"; // Your personal network password
// MQTT
const char* mqtt_server = "192.168.227.1"; // IP of the MQTT broker
const char* temperature_topic = "temperature";
const char* mqtt_username = "augustin"; // MQTT username
const char* mqtt_password = "MP3"; // MQTT password
const char* clientID = "client_cter_esp32_classroom"; // MQTT client ID
// Initialise the WiFi and MQTT Client objects
WiFiClient wifiClient;
// 1883 is the listener port for the Broker
PubSubClient client(mqtt_server, 1883, wifiClient);
// Custom function to connet to the MQTT broker via WiFi
void connect_MQTT(){
// Connect to MQTT Broker
// client.connect returns a boolean value to let us know if the connection was successful.
// If the connection is failing, make sure you are using the correct MQTT Username and Password (Setup Earlier in the Instructable)
if (client.connect(clientID, mqtt_username, mqtt_password)) {
 Serial.println("Connected to MQTT Broker!");
}
else {
 Serial.println("Connection to MQTT Broker failed...");
}
}
void setup() {
Serial.begin(9600);
// Oublie de l'ancienne config Wifi
WiFi.disconnect(true);
delay(1000);
WiFi.mode(WIFI_STA); // mode station
// Connect to Wifi
Serial.print("Connecting to ");
Serial.println(ssid);
WiFi.begin(ssid, wifi_password);
// Wait until the connection has been confirmed before continuing
while (WiFi.status() != WL_CONNECTED) {
 delay(500);
 Serial.print(".");
}

// Debugging - Output the IP Address of the ESP32
Serial.println("WiFi connected");
Serial.print("IP address: ");
Serial.println(WiFi.localIP());
pinMode(12, OUTPUT);
}
void loop() {
connect_MQTT();
Serial.setTimeout(2000);
int raw = analogRead(33);
Serial.print("raw : ");
Serial.println(raw);
float volts = (float)raw*3.3/4095; // il faut forcer volt a Ãªtre un float sinon la division renvoie un int (donc 0 au lieu de 0.2)
Serial.print("volts : ");
Serial.println(volts);
float degres = volts/0.01 + 10;
Serial.print("degres : ");
Serial.println(degres);
// MQTT can only transmit strings
String temperature_string = String(degres);
// PUBLISH to the MQTT Broker (topic = Temperature, defined at the beginning)
if (client.publish(temperature_topic, temperature_string.c_str())) {
 Serial.println("Temperature sent!");
 delay(5000);
}
// client.publish will return a boolean value depending on whether it succeded or not.
// If the message failed to send, we will try again, as the connection may have broken.
else {
 Serial.println("Temperature failed to send. Reconnecting to MQTT Broker and trying again");
 client.connect(clientID, mqtt_username, mqtt_password);
 delay(10); // This delay ensures that client.publish doesn't clash with the  qaclient.connect call
 client.publish(temperature_topic, temperature_string.c_str());
}
client.disconnect(); // disconnect from the MQTT broker
delay(2000); // print new values every 10 seconds
if (degres> 21){
  digitalWrite(12, HIGH);
}
else {
  digitalWrite(12, LOW);
}

}
