
#include <WiFi.h>
#include <esp_now.h>

// Pin Definitions
#define TRIG_PIN 26
#define ECHO_PIN 33
#define LED_PIN 2

// Constants
#define SPEED_OF_SOUND 0.0343
#define THRESHOLD_DIST 50
#define uS_TO_S_FACTOR 1000000
#define X  20

// Global Variables
String message;

// MAC address of the receiver
uint8_t broadcastAddress[] = {0x8C, 0xAA, 0xB5, 0x84, 0xFB, 0x90};

// ESP-NOW peer information
esp_now_peer_info_t peerInfo;

// Variables to track time
unsigned long wifiOnStartTime = 0;
unsigned long wifiOffStartTime = 0;
unsigned long wakeUpTime = 0;

RTC_NOINIT_ATTR uint64_t deepSleepStartTime ; 


// Function prototypes
void turnOnWiFi();
void turnOffWiFi();
void deepSleepMode();
float readDistance();


void setup() {
  Serial.begin(115200);
  Serial.println("----------ESP32 RESET----------");
 
  unsigned long wakeUpMoment = millis() ; // Moment to wake up [seconds in simulation]
  Serial.println("Wake up moment: " +String(wakeUpMoment/1000.0));
  unsigned long timeSpentDeepSleep = (wakeUpMoment - deepSleepStartTime) ;// Time duration of deep sleep
  Serial.println("Time spent in Deep Sleep: " +String(timeSpentDeepSleep/1000.0) +"s");


  // Initialize pins
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
  pinMode(LED_PIN, OUTPUT);
  
  // Measure time spent in WiFi ON section
  wifiOnStartTime = millis();
  turnOnWiFi();

  // Read distance
  unsigned long sensorReadingStartTime = millis();
  float dist = readDistance();
  unsigned long sensorReadingTime = millis() - sensorReadingStartTime;
  Serial.println("Time spent in sensor reading: " + String(sensorReadingTime) + "ms");
  Serial.println("The distance is " + String(dist) + " cm");

  // Send status message based on distance
  if (dist < THRESHOLD_DIST) {
    message = "OCCUPIED";
    digitalWrite(LED_PIN, HIGH);  
  } else {
    message = "FREE";
    digitalWrite(LED_PIN, LOW); 
  }
  delay(100);
  
  // Send the message via ESP-NOW
  if (esp_now_send(broadcastAddress, (uint8_t*)message.c_str(), message.length() + 1) != ESP_OK) {
    Serial.println("Failed to send message via ESP-NOW");
  } else {
    Serial.println("Message received: " + message);
  }
  unsigned long wifiOnTime = millis() - wifiOnStartTime;
  Serial.println("Time spent in WiFi ON: " + String(wifiOnTime / 1000.0) + " s");
  
  // Measure time spent in WiFi OFF section
  wifiOffStartTime = millis();
  turnOffWiFi();
  unsigned long wifiOffTime = millis() - wifiOffStartTime;
  Serial.println("Time spent in WiFi OFF: " + String(wifiOffTime / 1000.0) + " s");
 
  deepSleepStartTime = millis();
  Serial.println("Deep Sleep Start Moment: " +String(deepSleepStartTime/1000.0)); //The moment of starting deep sleep [seconds in simulation]

  deepSleepMode();
}

void loop() {
  // Nothing is done here
}

// Function to turn on WiFi
void turnOnWiFi() {
  WiFi.mode(WIFI_STA);
  WiFi.setTxPower(WIFI_POWER_2dBm);
  esp_now_init();
  memcpy(peerInfo.peer_addr, broadcastAddress, 6);
  peerInfo.channel = 0;
  peerInfo.encrypt = false;
  esp_now_add_peer(&peerInfo);
  delay(500);
}

// Function to turn off WiFi
void turnOffWiFi() {
  WiFi.mode(WIFI_OFF);
  delay(1000);
}

// Function to set up deep sleep
void deepSleepMode() {
  esp_sleep_enable_timer_wakeup(X * uS_TO_S_FACTOR);
  Serial.println("Setup ESP32 to sleep for every " + String(X) + " seconds");
  Serial.flush();
  delay(18800); // Adjustment of simulation time 
  esp_deep_sleep_start();
}

// Function to read distance from ultrasonic sensor
float readDistance() {
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);
  
  long duration = pulseIn(ECHO_PIN, HIGH);
  float dist = duration * SPEED_OF_SOUND / 2;
  return dist;
}
