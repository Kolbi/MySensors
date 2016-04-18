/**
 * The MySensors Arduino library handles the wireless radio link and protocol
 * between your home built sensors/actuators and HA controller of choice.
 * The sensors forms a self healing radio network with optional repeaters. Each
 * repeater and gateway builds a routing tables in EEPROM which keeps track of the
 * network topology allowing messages to be routed to nodes.
 *
 * Created by Henrik Ekblad <henrik.ekblad@mysensors.org>
 * Copyright (C) 2013-2015 Sensnology AB
 * Full contributor list: https://github.com/mysensors/Arduino/graphs/contributors
 *
 * Documentation: http://www.mysensors.org
 * Support Forum: http://forum.mysensors.org
 *
 * This program is free software; you can redistribute it and/or
 * modify it under the terms of the GNU General Public License
 * version 2 as published by the Free Software Foundation.
 *
 *******************************
 *
 * DESCRIPTION
 *
 * Example sketch showing how to send in DS1820B OneWire temperature readings back to the controller
 * http://www.mysensors.org/build/temp
 */


// Enable debug prints to serial monitor
#define MY_DEBUG 

// Enable and select radio type attached
#define MY_RADIO_NRF24
//#define MY_RADIO_RFM69

#include <SPI.h>
#include <MySensor.h>  
#include <DallasTemperature.h>
#include <OneWire.h>

#define COMPARE_TEMP 1 // Send temperature only if changed? 1 = Yes 0 = No
#define CELSIUS 1     // Temperatur in Celsius or Fahrenheit? 1 = Celsius 0 = Fahrenheit

#define ONE_WIRE_BUS 3 // Pin where dallase sensor is connected 
#define MAX_ATTACHED_DS18B20 16
unsigned long SLEEP_TIME = 30000; // Sleep time between reads (in milliseconds)
OneWire oneWire(ONE_WIRE_BUS); // Setup a oneWire instance to communicate with any OneWire devices (not just Maxim/Dallas temperature ICs)
DallasTemperature sensors(&oneWire); // Pass the oneWire reference to Dallas Temperature. 
float lastTemperature[MAX_ATTACHED_DS18B20];
int numSensors=0;
DeviceAddress tempDeviceAddress; // We'll use this variable to store a found device address
boolean receivedConfig = false;
boolean metric = true; 
// Initialize temperature message
MyMessage msgTemp(0,V_TEMP);
MyMessage msgId(0,V_ID);
//MyMessage msgHum(CHILD_ID_HUM, V_HUM);
//MyMessage msgTemp(CHILD_ID_TEMP, V_TEMP);

void setup()  
{ 
  // Startup up the OneWire library
  sensors.begin();
  // requestTemperatures() will not block current thread
  sensors.setWaitForConversion(false);
}

void presentation() {
  // Send the sketch version information to the gateway and Controller
  sendSketchInfo("Temperature Sensor", "1.1");

  // Fetch the number of attached temperature sensors  
  numSensors = sensors.getDeviceCount();
  
  //24022016
  // locate devices on the bus
  Serial.print("Locating devices...");
  
  Serial.print("Found ");
  Serial.print(numSensors);
  Serial.println(" devices.");
  
  // Loop through each device, print out address
  for(int i=0;i<numSensors; i++)
  {
    // Search the wire for address
    if(sensors.getAddress(tempDeviceAddress, i))
  {
    Serial.print("Found device ");
    Serial.print(i);
    Serial.print(" with address: ");
    printAddress(tempDeviceAddress);
    Serial.println();
  }else{
    Serial.print("Found ghost device at ");
    Serial.print(i);
    Serial.print(" but could not detect address. Check power and cabling");
  }
  }
  //24022016

  // Present all sensors to controller
  for (int i=0; i<numSensors && i<MAX_ATTACHED_DS18B20; i++) {   
     present(i, S_TEMP);
  }
  
}

void loop()     
{     
  // Fetch temperatures from Dallas sensors
  sensors.requestTemperatures();

  // query conversion time and sleep until conversion completed
  int16_t conversionTime = sensors.millisToWaitForConversion(sensors.getResolution());
  // sleep() call can be replaced by wait() call if node need to process incoming messages (or if node is repeater)
  sleep(conversionTime);

  // Read temperatures and send them to controller 
  for (int i=0; i<numSensors && i<MAX_ATTACHED_DS18B20; i++) {
    
  //24022016
  // Search the wire for address
    if(sensors.getAddress(tempDeviceAddress, i))
  {
    // Output the device ID
    Serial.print("Temperature for device: ");
    //Serial.println(i); 
    printAddress(tempDeviceAddress);
    
    #if CELSIUS == 1 // Celsius
    float temperature = sensors.getTempC(tempDeviceAddress);
    #else // Fahrenheit 
    float temperature = DallasTemperature::toFahrenheit(sensors.getTempC(tempDeviceAddress)); // Converts temperature to Fahrenheit
    #endif
    #if CELSIUS == 1 // Celsius
    Serial.print(" Temp C: ");
    #else // Fahrenheit
    Serial.print(" Temp F: ");
    #endif
    Serial.println(temperature);
   
  //else ghost device! Check your power requirements and cabling
/*
    // Fetch and round temperature to one decimal
    float temperature = static_cast<float>(static_cast<int>((getConfig().isMetric?sensors.getTempCByIndex(i):sensors.getTempFByIndex(i)) * 10.)) / 10.;
*/ 
    // Only send data if temperature has changed and no error 
    #if COMPARE_TEMP == 1
    if (lastTemperature[i] != temperature && temperature != -127.00 && temperature != 85.00) {
    #else
    if (temperature != -127.00 && temperature != 85.00) {
    #endif
 
      // Send in the new temperature
      send(msgTemp.setSensor(i).set(temperature,1));
      //8 sorgt dafür, dass alle 16 Stellen übermittelt werden
      send(msgId.setSensor(i).set(tempDeviceAddress,8));
      // Save new temperatures for next compare
      lastTemperature[i]=temperature;
    }
    }
//24022016
  }
  //sleep(SLEEP_TIME);
}

//24022016
// function to print a device address
void printAddress(DeviceAddress deviceAddress)
{
  for (uint8_t i = 0; i < 8; i++)
  {
    if (deviceAddress[i] < 16) Serial.print("0");
    Serial.print(deviceAddress[i], HEX);
    //return und im Loop als Aussage anzeigen?
    //printAddress(tempDeviceAddress); gibt bisher nur Serial.Print aus aber der Wert wird nicht aktiv in eine Variable übergeben
  }
}
//24022016
