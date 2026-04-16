#include <Wire.h>
#include <RTClib.h>
#include <LiquidCrystal.h>
#include <DHT.h>

// RTC + LCD
RTC_DS3231 rtc;
LiquidCrystal lcd(12, 11, 5, 4, 3, 2);

// DHT11
#define DHTPIN 13
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);

// Buttons
const int alarmButtonPin = 7;
const int stopButtonPin = 6;
const int mp3PlayPin = 8;

// Alarm variables
bool alarmSet = false;
bool alarmPlaying = false;
unsigned long alarmStartTime = 0;
const unsigned long delayBeforeSound = 5000;

void setup() {
  lcd.begin(16, 2);

  if (!rtc.begin()) {
    lcd.print("RTC Error");
    while (1);
  }

  // Set RTC only if power was lost
  if (rtc.lostPower()) {
    rtc.adjust(DateTime(F(__DATE__), F(__TIME__)));
  }

  dht.begin();

  pinMode(alarmButtonPin, INPUT_PULLUP);
  pinMode(stopButtonPin, INPUT_PULLUP);
  pinMode(mp3PlayPin, OUTPUT);

  digitalWrite(mp3PlayPin, HIGH);
}

void loop() {
  DateTime now = rtc.now();

  // ---- TIME (12 HOUR FORMAT) ----
  int hour = now.hour();
  String period = "AM";

  if (hour >= 12) period = "PM";
  if (hour > 12) hour -= 12;
  if (hour == 0) hour = 12;

  // ---- DISPLAY TIME ----
  lcd.setCursor(0, 0);
  lcd.print("Time: ");

  if (hour < 10) lcd.print("0");
  lcd.print(hour); lcd.print(":");

  if (now.minute() < 10) lcd.print("0");
  lcd.print(now.minute()); lcd.print(":");

  if (now.second() < 10) lcd.print("0");
  lcd.print(now.second());

  lcd.print(" ");
  lcd.print(period);
  lcd.print(" ");

  // ---- READ DHT11 ----
  float tempF = dht.readTemperature(true); // Fahrenheit
  float hum = dht.readHumidity();

  // ---- DISPLAY TEMP + HUMIDITY ----
  lcd.setCursor(0, 1);

  if (isnan(tempF) || isnan(hum)) {
    lcd.print("Sensor Error   ");
  } else {
    lcd.print("Temp:");
    lcd.print(tempF, 0);
    lcd.print("F ");

    lcd.print("Hm:");
    lcd.print(hum, 0);
    lcd.print("% ");
  }

  // ---- BUTTON: START ALARM ----
  if (digitalRead(alarmButtonPin) == LOW && !alarmSet) {
    alarmStartTime = millis();
    alarmSet = true;
    delay(250);
  }

  // ---- TRIGGER SOUND AFTER 5s ----
  if (alarmSet && !alarmPlaying && millis() - alarmStartTime >= delayBeforeSound) {
    digitalWrite(mp3PlayPin, LOW);
    delay(300);
    digitalWrite(mp3PlayPin, HIGH);
    alarmPlaying = true;
  }

  // ---- STOP BUTTON ----
  if (alarmPlaying && digitalRead(stopButtonPin) == LOW) {
    alarmPlaying = false;
    alarmSet = false;
    delay(250);
  }

  delay(200);
}
