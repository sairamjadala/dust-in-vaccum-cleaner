# dust-in-vaccum-cleaner

#include <WiFi.h>
#include "ThingSpeak.h"

#define TRIG_PIN 27
#define ECHO_PIN 14
#define LED_PIN_1 21
#define LED_PIN_2 23
#define BUZZER_PIN 5

const char *ssid = "12345";
const char *password = "mrrecharge";
const unsigned long channelId = 2532380;
const char *writeAPIKey = "8ZR4GO9HWAP5KRC2";

WiFiClient client;

void setup() {
  Serial.begin(9600);  // Initialize serial
  while (!Serial) {
    ; // wait for serial port to connect. Needed for Leonardo native USB port only
  }
  
  WiFi.mode(WIFI_STA);
  ThingSpeak.begin(client);  // Initialize ThingSpeak

  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
  pinMode(LED_PIN_1, OUTPUT);
  pinMode(LED_PIN_2, OUTPUT);
  pinMode(BUZZER_PIN, OUTPUT);

  // Connect to WiFi
  Serial.print("Attempting to connect to ssid: ");
  Serial.println(ssid);
  while(WiFi.status() != WL_CONNECTED) {
    WiFi.begin(ssid, password);
    Serial.print(".");
    delay(1500);
  } 
  Serial.println("\nConnected.");
}

void loop() {
  long duration, distance;
  
  // Ultrasonic sensor reading
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);
  duration = pulseIn(ECHO_PIN, HIGH);
  distance = duration * 0.034 / 2;

  Serial.print("Distance: ");
  Serial.print(distance);
  Serial.println(" cm");

  // Update ThingSpeak channel
  ThingSpeak.writeField(channelId, 2, distance, writeAPIKey);

  // Actuations based on distance
  if (distance <= 3) {  
    digitalWrite(LED_PIN_1, HIGH);  
    digitalWrite(BUZZER_PIN, HIGH);  
    delay(5000);  
    digitalWrite(BUZZER_PIN, LOW);  
  } else {
    digitalWrite(LED_PIN_1, LOW);  
  }
  
  if (distance <= 4) {  
    digitalWrite(LED_PIN_2, HIGH);
    digitalWrite(BUZZER_PIN, HIGH);  
    delay(500);  
    digitalWrite(BUZZER_PIN, LOW);   
  } else {
    digitalWrite(LED_PIN_2, LOW);  
  }

  delay(100);  
}
