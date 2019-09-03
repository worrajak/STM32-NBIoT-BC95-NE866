# STM32F103-NBIoT-BC95

Include and Config 
```
#include <stdio.h>
#include <stdarg.h>
#include <string.h>
#include <Arduino.h>

#include <SPI.h>
#include <Wire.h>
#include <Adafruit_Sensor.h>
#include <Adafruit_BME280.h>
#include <SSD1306Ascii.h>
#include <SSD1306AsciiWire.h>

#define I2C_ADDRESS 0x3C
#define RST_PIN -1
SSD1306AsciiWire oled;

#include <libmaple/pwr.h>
#include <libmaple/scb.h>
#include <RTClock.h>
RTClock rtclock(RTCSEL_LSE);

#include "AIS_NB_BC95.h"

String apnName = "devkit.nb";
String serverIP = "xxx.xxx.xxx.xxx"; // Your Server IP
String serverPort = "xxxx"; // Your Server Port

String udpData = "HelloWorld";

AIS_NB_BC95 AISnb;

const long interval = 20000;  //millisecond
unsigned long previousMillis = 0;

```

Setup 

```
  Serial.begin(9600);
  Serial2.begin(9600);
  delay(1000);
  
  AISnb.debug = true;
  
  bool status = bme.begin();  
    if (!status) {
        Serial.println("Could not find a valid BME280 sensor, check wiring!");
        while (1);
    }

  #if RST_PIN >= 0
    oled.begin(&Adafruit128x64, I2C_ADDRESS, RST_PIN);
  #else // RST_PIN >= 0
    oled.begin(&Adafruit128x64, I2C_ADDRESS);
  #endif // RST_PIN >= 0

  oled.setFont(Adafruit5x7);    
  oled.clear();
  oled.println("Water Level Test");
  oled.setCursor(0,2);
  oled.println("BME280 Ok!!!");
  oled.setCursor(0,4);
  oled.println("waiting 1 minute");
    
  AISnb.setupDevice(serverPort);

  String ip1 = AISnb.getDeviceIP();  
  pingRESP pingR = AISnb.pingIP(serverIP);
  
  oled.setCursor(0,3);
  oled.println("Connected AIS->");
  oled.setCursor(0,4);
  oled.println(ip1);

  previousMillis = millis();
```  

Loop Program

```  
void loop()
{ 
      readData();     
      String DataSend ="{\"id\":\"NB-IoT-1\",\"temperature\":"+String(temperature)+",\"humidity\":"+String(humidity)+",\"flowrate\":"+String(flowRate)+",\"level\":"+String(cm)+"}";
      UDPSend udp = AISnb.sendUDPmsgStr(serverIP, serverPort, DataSend);
      UDPReceive resp = AISnb.waitResponse(); 
      delay(10000); 
}
```  
