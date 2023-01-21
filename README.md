// RPLiDAR-A1M8

#include <RPLidar.h>

#include <LiquidCrystal.h>

#include <Servo.h>

RPLidar lidar;
LiquidCrystal lcd(7, 8, 9, 10, 11, 12);
Servo servoX, servoY;
const int buttonPin = 2;
const int Pin = 3;

void setup() {
    lidar.begin();
    lcd.begin(16, 2);
    servoX.attach(11);
    servoY.attach(12);
    servoX.write(0);
    servoY.write(0);
    pinMode(buttonPin, INPUT);
    pinMode(Pin, OUTPUT);
    digitalWrite(Pin, LOW);
}

void loop() {
    rplidar_response_measurement_node_t nodes[360*2];
    size_t count = _countof(nodes);
    lidar.grabScanData(nodes, count);
    lidar.ascendScanData(nodes, count);
    
    if(digitalRead(buttonPin) == HIGH) {
        digitalWrite(Pin, HIGH);
    } else {
        digitalWrite(Pin, LOW);
    }

    for (int pos = 0; pos < count; pos++) {
        int angle = nodes[pos].angle_q6_checkbit >> RPLIDAR_RESP_MEASUREMENT_ANGLE_SHIFT;
        int distance = nodes[pos].distance_q2/4.0f;
        if (distance >= 400 || distance <= 2) {
            // distance out of range
        } else {
            lcd.clear();
            lcd.setCursor(0, 0);
            lcd.print("Angle:");
            lcd.print(angle);
            lcd.setCursor(0, 1);
            lcd.print("Distance:");
            lcd.print(distance);
            delay(100);
            int servoXPos = map(angle, 0, 360, 0, 180);
            int servoYPos = map(distance, 2, 400, 0, 180);
            servoX.write(servoXPos);
            servoY.write(servoYPos);
        }
    }
}
