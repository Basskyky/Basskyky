

#define BLYNK_PRINT Serial

// define PIN Mapping
#define PIN_RELAY1  D0  // relay 1 PUMP
#define PIN_RELAY2  D1  // relay 2 วาล์ว
#define PIN_RELAY3  D2  // relay 3 วาล์ว2
#define PIN_RELAY4  D3  // relay 4 วาล์ว 3
#define PIN_RELAY5  D4  // relay 5 วาล์ว 4
#define PIN_DHT     D7  // dht temperture & humidity sensor
#define ON  LOW 
#define OFF HIGH

#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>



#include <Adafruit_Sensor.h>
#include <DHT.h>
#include <DHT_U.h>
#define DHTPIN      PIN_DHT   
#define DHTTYPE     DHT22     // DHT 22 (AM2302)
DHT_Unified dht(DHTPIN, DHTTYPE);
sensors_event_t event;  //เก็บค่าDHT

char auth[] = ""; // token
char ssid[] = "";  //wifi name
char pass[] = "";  //password 

float tempSensorData  = 0;
float humiSensorData  = 0;
bool  autoCondition;  // เช็คออโต้
int   limitHigh       = 35;
int   limitLow        = 33;
BlynkTimer timer;  // สั่งฟังก์ชั่นทีละส่วน ตามที่ตั้งเวลา

void programProcess()
{
  
  if(autoCondition)
  {
    Serial.println(limitHigh);
    Serial.println(limitLow);
    Serial.println(tempSensorData);
    if(tempSensorData < limitLow)
    {
      //เปิด Heater
      Serial.println("Condition 1");
      digitalWrite(PIN_RELAY1,ON);
      Blynk.virtualWrite(V11,HIGH);
    }

    if((tempSensorData >= limitLow) && (tempSensorData <= limitHigh))
    {
      //เปิด Heater
      Serial.println("Condition 1");
      digitalWrite(PIN_RELAY1,ON);
      Blynk.virtualWrite(V11,HIGH);
    }
  
    if(tempSensorData > limitHigh)
    {
      // ปืก Heater
      Serial.println("Condition 2");
      digitalWrite(PIN_RELAY1,OFF);
      Blynk.virtualWrite(V11,LOW);
    }
  }
}

void updateSensor() //
{
  // check temperature 
  dht.temperature().getEvent(&event);
  if (isnan(event.temperature)) {
    Serial.println(F("Error reading temperature!"));
  }
  else {
    tempSensorData = event.temperature;
    Serial.print(F("Temperature: "));
    Serial.print(tempSensorData);
    Serial.println(F("°C"));

    Blynk.virtualWrite(V0,tempSensorData);
  }
  
  // check humidity
  dht.humidity().getEvent(&event);
  if (isnan(event.relative_humidity)) {
    Serial.println(F("Error reading humidity!"));
  }
  else {
    humiSensorData = event.relative_humidity;
    Serial.print(F("Humidity: "));
    Serial.print(humiSensorData);
    Serial.println(F("%"));

    Blynk.virtualWrite(V1,humiSensorData);
  }
}


BLYNK_WRITE(V2) //รับค่าจากแอพ
{
  limitHigh = param.asInt();
}

BLYNK_WRITE(V3)
{
  limitLow = param.asInt();
}

BLYNK_WRITE(V10)
{
  autoCondition = param.asInt();
}

BLYNK_WRITE(V11) // ปุ่มสั่งเปิดปั้มน้ำ
{
  if(!autoCondition)
  {
    if(param.asInt())
    {
      digitalWrite(PIN_RELAY1,ON);
    }
    else
    {
      digitalWrite(PIN_RELAY1,OFF);
    }
  }
}

BLYNK_WRITE(V12)
{
  if(param.asInt())
  {
    digitalWrite(PIN_RELAY2,LOW);
    Blynk.setProperty(V11, "offLabel", "disable");
  }
  else
  {
    digitalWrite(PIN_RELAY2,HIGH);
  }
}

BLYNK_CONNECTED()
{
  Blynk.syncAll();
}

void setup()
{
  // Set Pin Mode Output
  pinMode(PIN_RELAY1, OUTPUT);     
  pinMode(PIN_RELAY2  , OUTPUT); 
  pinMode(PIN_RELAY3  , OUTPUT);   
  pinMode(PIN_RELAY4  , OUTPUT);   
  pinMode(PIN_RELAY5  , OUTPUT);   
      
  // Set default PIN Value HIG=0,LOW=1
  digitalWrite(PIN_RELAY1 , OFF);
  digitalWrite(PIN_RELAY2 , OFF);
  digitalWrite(PIN_RELAY3 , OFF);
  digitalWrite(PIN_RELAY4 , OFF);
  digitalWrite(PIN_RELAY5 , OFF);
  
  // Debug console
  Serial.begin(9600);
  dht.begin(); //สั่งสักอุณหภูมิเริ่มทำงาน

  
  
  Blynk.begin(auth, ssid, pass);
  updateSensor();
  timer.setInterval(2500L,updateSensor);
  timer.setInterval(5000L,programProcess);

}

void loop()
{
  Blynk.run();
  timer.run();
}
