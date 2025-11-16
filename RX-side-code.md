#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <RH_ASK.h>
#include <SPI.h>

// Initialize LCD display and RF module
LiquidCrystal_I2C lcd(0x27, 16, 2);
RH_ASK rf_driver;

// Define pin connections
const int ignitionRelay = 4;
const int buzzerPin = 5;

// Variables to store received data
String str_helmet = "0", str_alcohol = "0", str_sleep = "0";
int helmetStatus = 0, alcoholLevel = 0, sleepStatus = 0;
bool engineState = false;

// Timeout and page switching variables
unsigned long lastReceivedTime = 0;
const unsigned long timeout = 5000;
int page = 0, lastPage = -1;
unsigned long lastPageSwitch = 0;
const unsigned long pageInterval = 3000;

void setup() {
  Serial.begin(115200); // Initialize serial communication
  rf_driver.init();
  Serial.println("Waiting for data...");
 // Initialize RF module

  pinMode(ignitionRelay, OUTPUT);
  pinMode(buzzerPin, OUTPUT);
  digitalWrite(ignitionRelay, LOW); // Default: Vehicle OFF
  digitalWrite(buzzerPin, LOW); // Default: No alert

  lcd.init();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("Smart Helmet");
  delay(1000);
  lcd.clear();

}

void loop() {
  uint8_t buf[20];
  uint8_t buflen = sizeof(buf);

  // Check if RF data is received
  if (rf_driver.recv(buf, &buflen)) {
    String str_out = String((char*)buf);
    lastReceivedTime = millis(); // Update last received time
    }
  }

  // Handle timeout if no data received for a set time
  if (millis() - lastReceivedTime > timeout) {
    if (page != 99) {
      digitalWrite(buzzerPin, HIGH);
      lcd.clear();
      lcd.print("No Data Received");
      lcd.setCursor(0, 1);
      lcd.print("System Offline");
      page = 99;
    }
    return;
  }

  // Handle automatic page switching
  if (millis() - lastPageSwitch > pageInterval) {
    lastPageSwitch = millis();
    page = (page + 1) % 2;
    delay(200); // Small delay to prevent rapid switching
  }

  // Clear LCD only when switching pages
  if (page != lastPage) {
    lcd.clear();
    lastPage = page;
  }

  // Display different pages based on current page index
  switch (page) {
    case 0:  // Page 1: Helmet, Alcohol, and Sleep Status
      lcd.setCursor(0, 0);
      lcd.print("Helmet: ");
      lcd.print(helmetStatus == 1 ? "Yes" : "No ");
      lcd.setCursor(0, 1);
      lcd.print("Alcohol: ");
      lcd.print(alcoholLevel == 1 ? "Yes" : "No ");
      break;
  }
}
