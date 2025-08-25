```
#include <Servo.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SH110X.h>

// --- OLED setup ---
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
Adafruit_SH1106G display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);

// --- Servo + Ultrasonic ---
Servo servo;
int pos = 0;
bool goingForward = true;

void setup() {
  servo.attach(9);

  pinMode(2, OUTPUT);  // ultrasonic trigger
  pinMode(4, INPUT);   // ultrasonic echo

  Serial.begin(9600);

  // Initialize OLED
  if (!display.begin(0x3C, true)) {
    Serial.println(F("SH1106 not found"));
    for (;;);
  }
  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(SH110X_WHITE);
}

void loop() {
  // --- Ultrasonic Measurement ---
  digitalWrite(2, LOW);
  delayMicroseconds(2);
  digitalWrite(2, HIGH);
  delayMicroseconds(10);
  digitalWrite(2, LOW);

  long duration = pulseIn(4, HIGH, 20000);  // 20ms timeout
  long cm = duration / 29 / 2;

  display.clearDisplay();  // clear OLED each loop
  display.setCursor(0, 0);

  if (cm > 0 && cm <= 15) {
    // --- Object Detected ---
    display.setTextSize(1);
    display.println("waring! Object Detected!");
    display.setTextSize(2);
    display.print(cm);
    display.println(" cm");
    display.setTextSize(1);
    display.print("Servo Angle: ");
    display.print(pos);
    display.println(" deg");
    display.display();

    delay(100);  // slight delay to avoid rapid flicker
    return;     // skip movement if object detected
  }

  // --- Move Servo if No Object Detected ---
  servo.write(pos);

  display.setTextSize(2);
  display.print("Angle: ");
  display.println(pos);
  display.setTextSize(1);
  display.println("No object detected");
  display.display();

  delay(10);  // adjust for servo speed

  if (goingForward) {
    pos += 5;
    if (pos >= 180) goingForward = false;
  } else {
    pos -= 5;
    if (pos <= 0) goingForward = true;
  }
}```
