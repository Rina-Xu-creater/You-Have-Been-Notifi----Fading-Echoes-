# You-Have-Been-Notifi----Fading-Echoes-
# OCAD University - Yuhong Xu

#include "RTC.h"
#include <Servo.h>
#include "ArduinoGraphics.h"
#include "Arduino_LED_Matrix.h"

ArduinoLEDMatrix matrix;
Servo timeCollectorServo;
const int servoPin = 9;

// time argument
const int totalMinutes = 60;
int currentMinute = 0;
int lastMinute = -1;
int lastActionSecond = -1;  // up seconds time recording 

// movement argument
const int actionInterval = 20;  // 20 sec intervals
const int minAngle = 20;
const int maxAngle = 160;

void setup() {
  Serial.begin(9600);
  while (!Serial);

  if (!matrix.begin()) {
    Serial.println("LED fail");
    while (1);
  }

  RTC.begin();
  RTCTime initialTime(04, Month::APRIL, 2025, 10, 0, 0, DayOfWeek::FRIDAY, SaveLight::SAVING_TIME_ACTIVE);
  RTC.setTime(initialTime);

  timeCollectorServo.attach(servoPin);
  timeCollectorServo.write(90);  // set in the middle

  // Openning showing
  matrix.beginDraw();
  matrix.clear();
  matrix.stroke(0xFFFFFFFF);
  matrix.textFont(Font_4x6);
  matrix.text("TIME", 1, 1);
  matrix.text("COLL", 1, 8);
  matrix.endDraw();
  delay(1000);
}

void loop() {
  RTCTime currentTime;
  RTC.getTime(currentTime);
  currentMinute = currentTime.getMinutes();
  int currentSecond = currentTime.getSeconds();  // get the current second

  // update LED display in each minute
  if (currentMinute != lastMinute) {
    lastMinute = currentMinute;
    updateMatrixDisplay();
  }

  // every 20 sec trigger the movement
  if (currentSecond % actionInterval == 0 && currentSecond != lastActionSecond) {
    lastActionSecond = currentSecond;
    
    if (currentMinute < totalMinutes) {
      performTimedCollection();  
    }
  }
  
  delay(100);  
}

void performTimedCollection() {
  int actionAngle = random(minAngle, maxAngle);
  
  timeCollectorServo.write(actionAngle);
  delay(300); 
  timeCollectorServo.write(90);
  
  Serial.print("Action @ ");
  Serial.print(currentMinute);
  Serial.print("m ");
  Serial.print(lastActionSecond);
  Serial.print("s | Angle: ");
  Serial.println(actionAngle);
}

void updateMatrixDisplay() {
  matrix.beginDraw();
  matrix.clear();
  matrix.stroke(0xFFFFFFFF);
  matrix.textFont(Font_4x6);

  int remaining = totalMinutes - currentMinute;
  if (remaining >= 10) {
    matrix.text(String(remaining).c_str(), 4, 4);
  } else {
    matrix.text(String(remaining).c_str(), 6, 4);
  }
  
  matrix.endDraw();
}
