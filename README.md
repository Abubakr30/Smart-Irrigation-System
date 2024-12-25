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
