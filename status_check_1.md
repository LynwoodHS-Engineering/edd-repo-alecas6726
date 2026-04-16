#include <Wire.h>
#include <RTClib.h>
#include <LiquidCrystal.h>

RTC_DS3231 rtc;
LiquidCrystal lcd(12, 11, 5, 4, 3, 2);

// Buttons
const int alarmButtonPin = 7;
const int stopButtonPin = 6;
const int mp3PlayPin = 8; // IO1 trigger

// Alarm variables
bool alarmSet = false;
bool alarmPlaying = false;
unsigned long alarmStartTime = 0;
const unsigned long delayBeforeSound = 5000; // 5s countdown

void setup() {
  lcd.begin(16, 2);
  rtc.begin();

  pinMode(alarmButtonPin, INPUT_PULLUP);
  pinMode(stopButtonPin, INPUT_PULLUP);
  pinMode(mp3PlayPin, OUTPUT);

  digitalWrite(mp3PlayPin, HIGH); // idle HIGH
}

void loop() {
  DateTime now = rtc.now();

  // Display current time
  int hour = now.hour();
  String period = "AM";
  if (hour >= 12) period = "PM";
  if (hour > 12) hour -= 12;
  if (hour == 0) hour = 12;

  lcd.setCursor(0,0);
  lcd.print("Time: ");
  if (hour < 10) lcd.print("0");
  lcd.print(hour); lcd.print(":");
  if (now.minute() < 10) lcd.print("0");
  lcd.print(now.minute()); lcd.print(":");
  if (now.second() < 10) lcd.print("0");
  lcd.print(now.second());
  lcd.print(" "); lcd.print(period);

  // Display date / alarm status
  lcd.setCursor(0,1);
  if (alarmSet && !alarmPlaying) {
    unsigned long elapsed = millis() - alarmStartTime;
    if (elapsed < delayBeforeSound) {
      int secondsLeft = 5 - elapsed / 1000;
      lcd.print("Alarm in "); lcd.print(secondsLeft); lcd.print("s ");
    } else {
      lcd.print("Alarm Playing   ");
    }
  } else if (alarmPlaying) {
    lcd.print("Alarm Playing   ");
  } else {
    lcd.print("Date: ");
    lcd.print(now.month()); lcd.print("/");
    lcd.print(now.day()); lcd.print("/");
    lcd.print(now.year());
  }

  // Alarm button → start countdown
  if (digitalRead(alarmButtonPin) == LOW && !alarmSet) {
    alarmStartTime = millis();
    alarmSet = true;
    delay(200); // debounce
  }

  // After 5s → trigger MP3
  if (alarmSet && !alarmPlaying && millis() - alarmStartTime >= delayBeforeSound) {
    digitalWrite(mp3PlayPin, LOW);
    delay(400); // long enough to trigger module
    digitalWrite(mp3PlayPin, HIGH);
    alarmPlaying = true;
  }

  // Stop button → disable alarm in Arduino
  if (alarmPlaying && digitalRead(stopButtonPin) == LOW) {
    alarmPlaying = false;
    alarmSet = false;
    delay(200); // debounce
  }

  delay(50); // smooth LCD refresh
}

