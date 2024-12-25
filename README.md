This project demonstrates the development of an IoT-based automated water irrigation system using MQTT 
for communication. The system monitors soil moisture levels, temperature, and humidity and automatically 
controls a water pump to optimize irrigation. It also displays real-time data on an I2C LCD and publishes 
system states to an MQTT topic. Simulation was carried out using PICSimLab, while real-time data 
transmission was achieved using the HiveMQ public broker.

External Libraries Used:
 Ethernet.h: Manages the Ethernet connection for the Arduino.
 PubSubClient.h: Enables MQTT communication.
 DHT.h: Reads data from the DHT11 sensor.
 LiquidCrystal_I2C.h: Controls the I2C LCD display.

The code

#include <Ethernet.h>
#include <PubSubClient.h>
#include <DHT.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
byte mac[] = {0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED};
IPAddress ip(192, 168, 1, 177);
const char* mqtt_server = "broker.hivemq.com";
const int mqtt_port = 1883;
const char* Sys_topic = "/PLANT";
EthernetClient ethClient;PubSubClient client(ethClient);
#define SOIL_SENSOR_PIN A0
#define LED_PIN 5
#define PUMP 6
#define DHT_PIN 7
#define DHT_TYPE DHT11
LiquidCrystal_I2C lcd(0x27, 16, 2);
DHT dht(DHT_PIN, DHT_TYPE);
bool irrigationOn = false;
void reconnect() {
while (!client.connected()) {
Serial.print("Connecting to MQTT...");
if (client.connect("ArduinoClient")) {
Serial.println("Connected!");
} else {
Serial.print("Failed, rc=");
Serial.print(client.state());
Serial.println(". Retrying in 5 seconds...");
delay(5000);
}
}
}
void setup() {
Serial.begin(9600);
Ethernet.begin(mac, ip);
client.setServer(mqtt_server, mqtt_port);
dht.begin();
lcd.init();
lcd.backlight();
pinMode(LED_PIN, OUTPUT);
pinMode(PUMP, OUTPUT);
digitalWrite(LED_PIN, LOW);
digitalWrite(PUMP, LOW);
lcd.print("System Ready");
delay(2000);lcd.clear();
}
void loop() {
if (!client.connected()) {
reconnect();
}
client.loop();
int soilValue = analogRead(SOIL_SENSOR_PIN);
int soilPercentage = map(soilValue, 0, 1023, 100, 0);
float temperature = dht.readTemperature();
float humidity = dht.readHumidity();
if (isnan(temperature) || isnan(humidity)) {
Serial.println("Failed to read from DHT sensor!");
return;
}
lcd.setCursor(0, 0);
lcd.print("Soil: ");
lcd.print(soilPercentage);
lcd.print("%");
lcd.setCursor(0, 1);
lcd.print("T:");
lcd.print(temperature);
lcd.print("C H:");
lcd.print(humidity);
lcd.print("%");
if (soilPercentage < 30 && humidity < 60 && temperature > 25 && !irrigationOn) {
irrigationOn = true;
digitalWrite(PUMP, HIGH);
digitalWrite(LED_PIN, HIGH);
client.publish(Sys_topic, "Irrigation ON");
Serial.println("Irrigation ON");
} 
else if ((soilPercentage >= 30 || humidity >= 60 || temperature <= 25) && irrigationOn) {
irrigationOn = false;
digitalWrite(PUMP, LOW);
digitalWrite(LED_PIN, LOW);
client.publish(Sys_topic, "Irrigation OFF");
Serial.println("Irrigation OFF");}
delay(2000);
}
