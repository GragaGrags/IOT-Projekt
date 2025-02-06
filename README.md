# IOT-Projekt

- Programski kod projekta motoriziranih dvoriših vrata upravljanih sa mobilnom aplikacijom:

```
#include "BluetoothSerial.h"

String device_name = "ESP32-BT-Driver";
String data;
String dataBT;

// Provjera dostupnosti bluetootha
#if !defined(CONFIG_BT_ENABLED) || !defined(CONFIG_BLUEDROID_ENABLED)
#error Bluetooth is not enabled! Please run `make menuconfig` to and enable it
#endif

//provjera Serial Port profila
#if !defined(CONFIG_BT_SPP_ENABLED)
#error Serial Port Profile for Bluetooth is not available or not enabled. It is only available for the ESP32 chip.
#endif

BluetoothSerial SerialBT;

//definiranje pinova za elektromotor
#define MOTOR_PIN1 14
#define MOTOR_PIN2 12
#define ENABLE_PIN 13

//definiranje pin-a za IR senzor
#define IR_PIN 15

//postavke za l298n
const int motorSpeed = 100;
// brzina rada motora: 0 - 255 (0=off, 255=max)

void setup() {
  
  Serial.begin(115200);

  //postavljanje pinmode-a za motor
  pinMode(MOTOR_PIN1, OUTPUT);
  pinMode(MOTOR_PIN2, OUTPUT);
  pinMode(ENABLE_PIN, OUTPUT);

  //postavljanje pinmode-a za IR senzor
  pinMode(IR_PIN, INPUT);
  
  //postavljanje BT-a
  SerialBT.begin(device_name);  //Ime bluetooth uredaja
  Serial.printf("The device with name \"%s\" is started.\nNow you can pair it with Bluetooth!\n", device_name.c_str());
}

void loop() {

  if (Serial.available() > 0) {
    data = Serial.readString();
    data.trim();
    if (data.equals("OPEN")){
      //otvori
      open();
    }
    else if (data.equals("CLOSE")){
      //zatvori
      close();
    }
    data = "";
  }
  if (SerialBT.available() > 0) {
    dataBT = SerialBT.readString();
    dataBT.trim();
    if (dataBT.equals("OPEN")){
      //otvori BT
      open();
    }
    else if (dataBT.equals("CLOSE")){
      //zatvori BT
      close();
    }
    dataBT = "";
  }

  delay(20);
}

void open(){
  Serial.println("otvaram!");
  analogWrite(ENABLE_PIN, motorSpeed); //Enable pin
  digitalWrite(MOTOR_PIN1, HIGH);
  digitalWrite(MOTOR_PIN2, LOW);
  delay(9500);
  digitalWrite(MOTOR_PIN1, LOW);
  digitalWrite(MOTOR_PIN2, LOW);
}

void close(){
  int time = 12000;
  Serial.println("zatvaram!");
  analogWrite(ENABLE_PIN, motorSpeed); //Enable pin
  while(time > 0){
    if(checkIR() == 0){
      digitalWrite(MOTOR_PIN1, LOW);
      digitalWrite(MOTOR_PIN2, HIGH);
      delay(1);
      time = time - 1;
    }
    else{
        digitalWrite(MOTOR_PIN1, LOW);
        digitalWrite(MOTOR_PIN2, LOW);
    }
  }
  digitalWrite(MOTOR_PIN1, LOW);
  digitalWrite(MOTOR_PIN2, LOW);
}

int checkIR(){
  digitalWrite(IR_PIN, HIGH);
  long value = digitalRead(IR_PIN);
  digitalWrite(IR_PIN, LOW);
  return value;
}
```
