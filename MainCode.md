
# Color Sorter Robot - Source Code Documentation
*Last Updated: 2025-03-02 00:22:53 UTC*  
*Author: Soompa911*

## Source Code

This document contains the complete source code for the Color Sorter Robot project. The code is written for Arduino and requires several external libraries.

### Required Libraries
- Wire.h
- Adafruit_TCS34725.h
- LiquidCrystal_I2C.h
- Adafruit_PWMServoDriver.h

### Complete Source Code

```cpp
#include <Wire.h>
#include "Adafruit_TCS34725.h"
#include <LiquidCrystal_I2C.h>
#include <Adafruit_PWMServoDriver.h>

// Initialize hardware components
Adafruit_PWMServoDriver srituhobby = Adafruit_PWMServoDriver();
Adafruit_TCS34725 tcs = Adafruit_TCS34725(TCS34725_INTEGRATIONTIME_600MS, TCS34725_GAIN_16X);
LiquidCrystal_I2C lcd(0x27, 20, 4);

// Global variables for tracking detected colors
int redDetected = 0, greenDetected = 0, blueDetected = 0, totalDetected = 0;
char lastColorDetected = 'N'; // N for None, R for Red, G for Green, B for Blue

// Servo definitions
#define servo1 0
#define servo2 1
#define servo3 2
#define servo4 3

// Motor control pin definitions
const int motorPWM = 6;  // PWM control on pin D6
const int motorDir1 = 4; // First direction control pin on D4
const int motorDir2 = 5; // Second direction control pin on D5

// LED indicator pin definitions
const int ledRed = 10;   // LED 1 for Red at pin D10
const int ledGreen = 9;  // LED 2 for Green at pin D9
const int ledBlue = 8;   // LED 3 for Blue at pin D8

void setup() {
  Serial.begin(9600);
  Wire.begin();
  srituhobby.begin();
  srituhobby.setPWMFreq(60);
  if (!tcs.begin()) {
    Serial.println("Color Sensor not found. Please check wiring.");
    while (1);
  }
  tcs.setInterrupt(false);
  lcd.begin(20, 4);
  lcd.backlight();

  srituhobby.setPWM(servo1, 0, 150);
  srituhobby.setPWM(servo2, 0, 450);
  srituhobby.setPWM(servo3, 0, 250);
  srituhobby.setPWM(servo4, 0, 570);

  // Initialize motor control pins
  pinMode(motorPWM, OUTPUT);
  pinMode(motorDir1, OUTPUT);
  pinMode(motorDir2, OUTPUT);
  
  // Initialize LED pins
  pinMode(ledRed, OUTPUT);
  pinMode(ledGreen, OUTPUT);
  pinMode(ledBlue, OUTPUT);

  // Set initial motor and LED states to off (for safety)
  digitalWrite(motorDir1, LOW);
  digitalWrite(motorDir2, LOW);
  analogWrite(motorPWM, 0);
  digitalWrite(ledRed, LOW);
  digitalWrite(ledGreen, LOW);
  digitalWrite(ledBlue, LOW);
}

void detectColor(uint16_t r, uint16_t g, uint16_t b) {
  char currentColor = 'N'; // Assume no color detected initially

  // Turn off all LEDs initially
  digitalWrite(ledRed, LOW);
  digitalWrite(ledGreen, LOW);
  digitalWrite(ledBlue, LOW);

  if (r > g && r > b && (r - max(g, b)) > 150) {
    currentColor = 'R'; // Red detected
    digitalWrite(ledRed, HIGH);
  } else if (g > r && g > b && (g - max(r, b)) > 150) {
    currentColor = 'G'; // Green detected
    digitalWrite(ledGreen, HIGH);
  } else if (b > r && b > g && (b - max(r, g)) > 150) {
    currentColor = 'B'; // Blue detected
    digitalWrite(ledBlue, HIGH);
  }

  // Motor control based on color detection
  if (currentColor == 'R' || currentColor == 'G' || currentColor == 'B') {
    analogWrite(motorPWM, 0); // Stop motor
  } else {
    digitalWrite(motorDir1, HIGH);
    digitalWrite(motorDir2, LOW);
    analogWrite(motorPWM, 255);
  }

  // Update counts and move servos if new color detected
  if (currentColor != lastColorDetected) {
    switch (currentColor) {
      case 'R':
        redDetected++;
        moveServosForRed();
        break;
      case 'G':
        greenDetected++;
        moveServosForGreen();
        break;
      case 'B':
        blueDetected++;
        moveServosForBlue();
        break;
    }
    lastColorDetected = currentColor;
    totalDetected = redDetected + greenDetected + blueDetected;
  }
}

void moveServosForRed() {
  // Movement sequence for red objects
  servoMovementSequence(265, 550);
}

void moveServosForGreen() {
  // Movement sequence for green objects
  servoMovementSequence(265, 475);
}

void moveServosForBlue() {
  // Movement sequence for blue objects
  servoMovementSequence(265, 350);
}

void servoMovementSequence(int firstPos, int finalPos) {
  // Common servo movement sequence
  moveServo(servo1, 150, firstPos, 3);
  delay(1000);
  
  moveServo(servo3, 300, 450, 3);
  delay(1000);
  
  moveServo(servo4, 570, 510, -3);
  delay(1000);
  
  moveServo(servo3, 450, 250, -3);
  delay(1000);
  
  moveServo(servo1, firstPos, finalPos, 3);
  delay(1000);
  
  moveServo(servo3, 250, 475, 3);
  delay(1000);
  
  moveServo(servo4, 510, 570, 3);
  delay(1000);
  
  moveServo(servo3, 450, 250, -3);
  delay(1000);
  
  moveServo(servo1, finalPos, 150, -3);
  delay(1000);
}

void moveServo(int servo, int startPos, int endPos, int step) {
  for (int pos = startPos; 
       step > 0 ? pos <= endPos : pos >= endPos; 
       pos += step) {
    srituhobby.setPWM(servo, 0, pos);
    delay(15);
  }
}

void updateLCD() {
  lcd.setCursor(0, 0); lcd.print("RED: "); lcd.print(redDetected);
  lcd.setCursor(0, 1); lcd.print("GREEN: "); lcd.print(greenDetected);
  lcd.setCursor(0, 2); lcd.print("BLUE: "); lcd.print(blueDetected);
  lcd.setCursor(0, 3); lcd.print("TOTAL: "); lcd.print(totalDetected);
}

void loop() {
  uint16_t r, g, b, c;
  tcs.getRawData(&r, &g, &b, &c);
  
  Serial.print("Red: "); Serial.println(r);
  Serial.print(" Green: "); Serial.println(g);
  Serial.print(" Blue: "); Serial.println(b);
  
  detectColor(r, g, b);
  updateLCD();
  
  delay(200);
}
```

### Code Structure

The code is organized into several main sections:

1. **Initialization and Setup**
   - Libraries inclusion
   - Hardware component initialization
   - Pin configurations
   - Initial state setup

2. **Color Detection**
   - RGB value reading
   - Color classification logic
   - Motor control based on detection

3. **Servo Movement**
   - Position control for each color
   - Movement sequences
   - Timing controls

4. **Display and Monitoring**
   - LCD updates
   - Serial debugging output
   - LED status indicators

### Function Documentation

#### `void setup()`
Initializes all hardware components and sets initial states.

#### `void detectColor(uint16_t r, uint16_t g, uint16_t b)`
Analyzes RGB values to determine the color of the detected object.

#### `void moveServosForRed/Green/Blue()`
Specialized movement sequences for each color.

#### `void servoMovementSequence(int firstPos, int finalPos)`
Generic servo movement pattern used by all color sequences.

#### `void moveServo(int servo, int startPos, int endPos, int step)`
Controls individual servo movement with specified parameters.

#### `void updateLCD()`
Updates the LCD display with current count statistics.

#### `void loop()`
Main program loop handling continuous color detection and sorting.

### Notes
- Servo movement delays are calibrated for smooth operation
- Color detection thresholds may need adjustment based on lighting conditions
- LCD update rate is set to 200ms for stable display

---

## Version History
- Initial Release: 2025-03-02
- Author: Soompa911
