#include <ESP8266WiFi.h>
#include <Adafruit_MQTT.h>
#include <Adafruit_MQTT_Client.h>
#define fireSensorPin D1  
char ssid[] = "2G_SOUTO";        
char pass[] = "12345678";      
#define AIO_SERVER      "io.adafruit.com"
#define AIO_SERVERPORT  1883                   
#define AIO_USERNAME    "marcos_souto"  
#define AIO_KEY         "aio_FxvV46BJDZEaHNOMG5qCd3G911dU"       
WiFiClient client;
Adafruit_MQTT_Client mqtt(&client, AIO_SERVER, AIO_SERVERPORT, AIO_USERNAME, AIO_KEY);
Adafruit_MQTT_Publish fireStatusFeed = Adafruit_MQTT_Publish(&mqtt, AIO_USERNAME "/feeds/firestatus");
unsigned long lastPublishTime = 0;   
unsigned long lastAlarmTime = 0;     
const unsigned long normalPublishInterval = 5000;  
const unsigned long alarmPublishInterval = 2000;   
void connectToMQTT() {
  int8_t ret;
  
  
  while ((ret = mqtt.connect()) != 0) {
    mqtt.disconnect();
    delay(5000);  
  }
}


void monitorSensor() {
  
  int fireStatus = digitalRead(fireSensorPin);
  
  
  if (fireStatus == LOW) {
    
    if (millis() - lastAlarmTime >= alarmPublishInterval) {
      Serial.println("Alarme De Incêndio!");  
      fireStatusFeed.publish("Alarme De Incêndio!");  
      lastAlarmTime = millis(); 
    }
  }
  
  
  else if (millis() - lastPublishTime >= normalPublishInterval) {
    Serial.println("Nenhum Alarme detectado.");  
    fireStatusFeed.publish("Nenhum Alarme detectado.");  
    lastPublishTime = millis();  
    lastAlarmTime = 0;  
  }
}


void connectToWiFi() {
  Serial.print("Conectando ao Wi-Fi");
  WiFi.begin(ssid, pass);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println();
  Serial.println("WiFi conectado com sucesso!");
  Serial.print("Endereço de IP: ");
  Serial.println(WiFi.localIP());
}

void setup() {
  
  Serial.begin(9600);

 
  connectToWiFi();

  
  connectToMQTT();

  
  pinMode(fireSensorPin, INPUT);

  
  monitorSensor();
}

void loop() {
  
  if (!mqtt.connected()) {
    connectToMQTT();
  }

  
  mqtt.ping();

  
  monitorSensor();
}
