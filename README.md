// PROJECT: TheSafeKit
// PURPOSE: Lock and unlock a safe using a passcode
// COURSE: ICS3U-E
// AUTHOR: Nathan Le-Nguyen
// DATE: 6 April 2025
// MCU: 328P
// STATUS: Working
// REFERENCE: http://darcy.rsgc.on.ca/ACES/TEI3M/2425/ISPs.html 
 
#include <Servo.h>
 
Servo myServo;          // Create a Servo object
 
#define DURATION  300   // Delay duration to debounce keypad
#define DEBUG     1     // Enable debug mode to print values
 
const int keypadPin = A5;  // Analog pin for keypad input
const int servoPin = 9;    // Pin for servo
 
// Threshold values for detecting keys based on analog input
uint16_t thresholds[] = {56, 59, 63, 67, 76, 82, 89, 98, 119, 134, 154, 182, 259, 346, 513, 1023};
// Corresponding keys mapped to threshold values
char keys[] =           { '+', 'E', '.', '0', '-', '3', '2', '1', '*', '6', '5', '4', '/', '9', '8', '7' };
 
const char passcode[5] = "6767";  // Adjustable 4-digit passcode
char inputCode[5] = "";           // Stores entered keys for comparison
byte index = 0;                   // Current position in inputCode
 
bool isArmed = false;        // Variable to track if system is armed or disarmed
bool servoMoving = false;    // Flag to check if the servo is moving
bool startReading = false;   // Flag to check if keypad input should be read
 
void setup() {
  myServo.attach(servoPin);  // Attach the servo
  myServo.write(0);          // Start with servo in locked position (disarmed)
  Serial.begin(9600);        // Initialize serial communication
}
 
char getKey() {
  uint16_t value = 0;
  do {
    value = analogRead(keypadPin);    // Read analog value from keypad
  } while (value == 0 || value < 25); // Ignore values below 25
 
  delay(DURATION); // Debounce delay
  if (DEBUG) {
    Serial.print("ADC:\t");
    Serial.print(value);
    Serial.print('\t');
    Serial.print("Voltage:\t");
    Serial.print(value * 5.0 / 1023);
    Serial.print('\t');
  }
 
  uint8_t index = 0;
  while (value > thresholds[index]) { // Find the correct key based on threshold
    index++;
  }
  return keys[index];                 // Return the detected key
}
 
void loop() {
  if (!servoMoving) {                 // Only read keypad if the servo isn't moving and we're ready to read
    if (startReading) {               // Start reading only if 'E' has been pressed
      char key = getKey();            // Read a key press
   
      if (key) {                      // If a key is pressed
        Serial.print(key);            // Print the pressed key
        if (index < 4) {              // Store up to 4 key presses
          inputCode[index++] = key;
        }
       
        if (index == 4) {             // After 4 keys entered, check passcode
          inputCode[4] = '\0';        // Null terminate string for valid C-string
         
          if (strcmp(inputCode, passcode) == 0) { // Compare entered code w passcode
            Serial.println("\nAccess Granted");
           
            // Toggle servo state between disarmed and armed
            if (isArmed) {
              Serial.println("Disarming...");  // Debugging print
              servoMoving = true;              // Set flag that servo is moving
              myServo.write(0);                // Move servo to disarmed position
              isArmed = false;                 // Set system state to disarmed
              servoMoving = false;             // Set flag back to not moving
              delay(10000);
            } else {
              Serial.println("Arming...");     // Debugging print
              servoMoving = true;              // Set flag that servo is moving
              myServo.write(90);               // Move servo to armed position
              isArmed = true;                  // Set system state to armed
              servoMoving = false;             // Set flag back to not moving
              delay(10000);
            }
           
          } else {
            Serial.println("\nAccess Denied");
          }
         
          index = 0;                          // Reset input buffer for next attempt
          startReading = false;               // Stop reading after passcode check
        }
      }
    }
    else {
      char key = getKey();                    // Read a key press to enable startReading flag
      if (key == 'E') {                       // When 'E' is pressed, read passcode
        startReading = true;
        Serial.println("Start entering passcode.");
      }
    }
  }
 
  // Short delay before checking passcode again after servo moves
  delay(100);                                 // Adjust as needed
}
![image](https://github.com/user-attachments/assets/3d05e51a-f6b7-4c11-a246-75eaf0a2d1f3)
