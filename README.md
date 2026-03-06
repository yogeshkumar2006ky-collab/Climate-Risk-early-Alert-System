# Climate-Risk-early-Alert-System

# Climatic Risk Early Alert System

## Project Overview
A system that detects environmental changes (temperature, air quality, water levels) and alerts users using LCD, buzzer, and ThingSpeak cloud.

## Hardware Used
- DHT11 Sensor
- Air Quality Sensor
- Ultrasonic Sensor
- ESP8266 WiFi Module
- LCD Display
- Buzzer

## How to Run
1. Connect sensors to ESP8266 as per the circuit diagram.
2. Upload the Arduino code.
3. Turn on the system; monitor alerts on LCD and ThingSpeak.

## Features
- Real-time monitoring of environment
- Early warning system
- Cloud data visualization via ThingSpeak

## Author
Team Name CRUSADES

#include <DHT.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

#define DHTPIN 2
#define DHTTYPE DHT11

DHT dht(DHTPIN, DHTTYPE);

LiquidCrystal_I2C lcd(0x27,16,2);

int gasSensor = A0;

int trigPin = 9;
int echoPin = 10;

long duration;
int distance;

int prevDistance = 0;

void setup()
{

Serial.begin(9600);

dht.begin();

lcd.init();
lcd.backlight();

pinMode(trigPin, OUTPUT);
pinMode(echoPin, INPUT);

}

void loop()
{

float temp = dht.readTemperature();
float hum = dht.readHumidity();

int air = analogRead(gasSensor);

digitalWrite(trigPin, LOW);
delayMicroseconds(2);

digitalWrite(trigPin, HIGH);
delayMicroseconds(10);

digitalWrite(trigPin, LOW);

duration = pulseIn(echoPin, HIGH);

distance = duration * 0.034 / 2;

lcd.clear();

lcd.setCursor(0,0);
lcd.print("T:");
lcd.print(temp);

lcd.print(" H:");
lcd.print(hum);

lcd.setCursor(0,1);
lcd.print("Air:");
lcd.print(air);

Serial.print(temp);
Serial.print(",");
Serial.print(hum);
Serial.print(",");
Serial.print(air);
Serial.print(",");
Serial.println(distance);

if(distance < prevDistance - 5)
{
lcd.clear();
lcd.print("Flood Warning!");
}

if(temp > 35 && hum < 40)
{
lcd.clear();
lcd.print("Heatwave Alert!");
}

if(air > 350)
{
lcd.clear();
lcd.print("Air Pollution!");
}

prevDistance = distance;

delay(30000);
}


#include <ESP8266WiFi.h>
#include <ThingSpeak.h>

const char* ssid = "YOUR_WIFI";
const char* password = "YOUR_PASSWORD";

WiFiClient client;

unsigned long channelID = YOUR_CHANNEL_ID;
const char* apiKey = "YOUR_API_KEY";

void setup()
{

Serial.begin(9600);

WiFi.begin(ssid,password);

while(WiFi.status()!=WL_CONNECTED)
{
delay(500);
}

ThingSpeak.begin(client);

}

void loop()
{

if(Serial.available())
{

float temp = Serial.parseFloat();
float hum = Serial.parseFloat();
int air = Serial.parseInt();
int water = Serial.parseInt();

ThingSpeak.setField(1,temp);
ThingSpeak.setField(2,hum);
ThingSpeak.setField(3,air);
ThingSpeak.setField(4,water);

ThingSpeak.writeFields(channelID,apiKey);

}

delay(30000);

}

