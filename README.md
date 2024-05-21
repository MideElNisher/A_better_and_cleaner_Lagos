# A_better_and_cleaner_Lagos
The purpose of this code is to solve the waste management issue in Lagos, Nigeria.
Code:
#include <SoftwareSerial.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// Define the pins for the ultrasonic sensor
const int trigPin = 2;
const int echoPin = 3;

// Initialize the LCD display (address, columns, rows)
LiquidCrystal_I2C lcd(0x27, 16, 2);

// Initialize SoftwareSerial for ESP8266
SoftwareSerial esp8266(10, 11); // RX, TX

// Function to send data to the server
void sendToServer(String data) {
  esp8266.println("AT+CIPSTART=\"TCP\",\"YOUR_SERVER_ADDRESS\",YOUR_PORT");
  delay(2000);
  
  String cmd = "AT+CIPSEND=";
  cmd += data.length();
  esp8266.println(cmd);
  delay(2000);
  
  esp8266.println(data);
  delay(2000);
  
  esp8266.println("AT+CIPCLOSE");
}

void setup() {
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  
  Serial.begin(9600);
  esp8266.begin(9600);
  
  lcd.init();
  lcd.backlight();
  
  // Initialize ESP8266
  esp8266.println("AT");
  delay(1000);
  esp8266.println("AT+CWJAP=\"YOUR_SSID\",\"YOUR_PASSWORD\"");
  delay(5000);
}

void loop() {
  // Measure distance using the ultrasonic sensor
  long duration, distance;
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
  
  duration = pulseIn(echoPin, HIGH);
  distance = (duration / 2) / 29.1;
  
  // Display distance on LCD
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Distance: ");
  lcd.print(distance);
  lcd.print(" cm");
  
  // Send data to server
  if (distance < 10) { // For example, if distance is less than 10 cm, bin is considered full
    sendToServer("Bin is full");
  } else {
    sendToServer("Bin is not full");
  }
  
  delay(10000); // Delay between measurements
}
