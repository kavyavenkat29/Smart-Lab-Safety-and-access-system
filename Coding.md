# \#include \<Keypad.h\>

# \#include \<Servo.h\>

# \#include \<DHT.h\>

# 

# // \================= KEYPAD \=================

# const byte ROWS \= 4;

# const byte COLS \= 4;

# 

# char keys\[ROWS\]\[COLS\] \= {

#   {'1','2','3','A'},

#   {'4','5','6','B'},

#   {'7','8','9','C'},

#   {'\*','0','\#','D'}

# };

# 

# byte rowPins\[ROWS\] \= {2, 3, 4, 5};

# byte colPins\[COLS\] \= {6, 7, 8, 9};

# 

# Keypad keypad \= Keypad(makeKeymap(keys),

#                        rowPins,

#                        colPins,

#                        ROWS,

#                        COLS);

# 

# // \================= SERVO \=================

# Servo doorServo;

# \#define SERVO\_PIN 10

# 

# // \================= ULTRASONIC \=================

# \#define TRIG\_PIN 11

# \#define ECHO\_PIN 12

# 

# // \================= DHT11 \=================

# \#define DHT\_PIN A0

# \#define DHT\_TYPE DHT11

# 

# DHT dht(DHT\_PIN, DHT\_TYPE);

# 

# // \================= MQ2 \=================

# \#define MQ2\_PIN A1

# 

# // Change this value after testing your MQ-2

# int GAS\_THRESHOLD \= 500;

# 

# // \================= RELAY \=================

# \#define RELAY\_PIN A2

# 

# // Most relay modules are ACTIVE LOW.

# // If yours works opposite, change LOW to HIGH.

# \#define RELAY\_ON LOW

# \#define RELAY\_OFF HIGH

# 

# // \================= BUZZER \=================

# \#define BUZZER\_PIN A3

# 

# // \================= PASSWORD \=================

# String correctPassword \= "1234";

# String enteredPassword \= "";

# 

# // \================= DOOR \=================

# bool doorUnlocked \= false;

# 

# unsigned long doorOpenTime \= 0;

# const unsigned long DOOR\_OPEN\_TIME \= 5000;

# 

# // \================= WRONG PASSWORD ALARM \=================

# bool wrongPasswordAlarm \= false;

# unsigned long wrongAlarmStart \= 0;

# 

# const unsigned long WRONG\_ALARM\_TIME \= 120000UL; // 2 minutes

# 

# // \================= SENSOR TIMING \=================

# unsigned long lastSensorCheck \= 0;

# const unsigned long SENSOR\_INTERVAL \= 10000; // 10 seconds

# 

# 

# void setup() {

# 

#   Serial.begin(9600);

# 

#   // Servo

#   doorServo.attach(SERVO\_PIN);

# 

#   // Start locked

#   doorServo.write(0);

# 

#  **CODING** 

# // Ultrasonic

#   pinMode(TRIG\_PIN, OUTPUT);

#   pinMode(ECHO\_PIN, INPUT);

# 

#   // MQ2

#   pinMode(MQ2\_PIN, INPUT);

# 

#   // Relay

#   pinMode(RELAY\_PIN, OUTPUT);

# 

#   // Initially relay OFF

#   digitalWrite(RELAY\_PIN, RELAY\_OFF);

# 

#   // Buzzer

#   pinMode(BUZZER\_PIN, OUTPUT);

#   digitalWrite(BUZZER\_PIN, LOW);

# 

#   // DHT

#   dht.begin();

# 

#   Serial.println("================================");

#   Serial.println(" SMART LAB SAFETY SYSTEM");

#   Serial.println("================================");

#   Serial.println("Enter Password:");

# }

# 

# 

# void loop() {

# 

#   // \------------------------------------------------

#   // 1\. KEYPAD

#   // \------------------------------------------------

# 

#   char key \= keypad.getKey();

# 

#   if (key) {

# 

#     Serial.print("Key: ");

#     Serial.println(key);

# 

#     // Clear password

#     if (key \== '\*') {

# 

#       enteredPassword \= "";

# 

#       Serial.println("Password cleared");

#     }

# 

#     // Enter password

#     else if (key \== '\#') {

# 

#       checkPassword();

#     }

# 

#     // Add number

#     else {

# 

#       if (enteredPassword.length() \< 10\) {

# 

#         enteredPassword \+= key;

# 

#         Serial.print("Password: ");

# 

#         for (int i \= 0; i \< enteredPassword.length(); i++) {

#           Serial.print("\*");

#         }

# 

#         Serial.println();

#       }

#     }

#   }

# 

# 

#   // \------------------------------------------------

#   // 2\. WRONG PASSWORD 2-MINUTE ALARM

#   // \------------------------------------------------

# 

#   if (wrongPasswordAlarm) {

# 

#     if (millis() \- wrongAlarmStart \< WRONG\_ALARM\_TIME) {

# 

#       digitalWrite(BUZZER\_PIN, HIGH);

# 

#     } else {

# 

#       digitalWrite(BUZZER\_PIN, LOW);

# 

#       wrongPasswordAlarm \= false;

# 

#       Serial.println("Wrong password alarm finished");

#       Serial.println("Enter Password:");

#     }

#   }

# 

# 

#   // \------------------------------------------------

#   // 3\. AUTOMATIC DOOR LOCK

#   // \------------------------------------------------

# 

#   if (doorUnlocked) {

# 

#     if (millis() \- doorOpenTime \>= DOOR\_OPEN\_TIME) {

# 

#       doorServo.write(0);

# 

#       doorUnlocked \= false;

# 

#       Serial.println("Door LOCKED");

#       Serial.println("Enter Password:");

#     }

#   }

# 

# 

#   // \------------------------------------------------

#   // 4\. SENSOR CHECK EVERY 10 SECONDS

#   // \------------------------------------------------

# 

#   if (millis() \- lastSensorCheck \>= SENSOR\_INTERVAL) {

# 

#     lastSensorCheck \= millis();

# 

#     checkSensors();

#   }

# }

# 

# 

# // \==================================================

# // PASSWORD FUNCTION

# // \==================================================

# 

# void checkPassword() {

# 

#   Serial.println();

# 

#   if (enteredPassword \== correctPassword) {

# 

#     Serial.println("ACCESS GRANTED");

# 

#     // Stop wrong alarm if running

#     wrongPasswordAlarm \= false;

#     digitalWrite(BUZZER\_PIN, LOW);

# 

#     // Unlock servo

#     doorServo.write(90);

# 

#     doorUnlocked \= true;

#     doorOpenTime \= millis();

# 

#     Serial.println("Door UNLOCKED");

#     Serial.println("Door will lock after 5 seconds");

# 

#   }

# 

#   else {

# 

#     Serial.println("ACCESS DENIED");

#     Serial.println("Wrong Password\!");

#     Serial.println("Buzzer ON for 2 minutes");

# 

#     // Start 2-minute buzzer

#     wrongPasswordAlarm \= true;

#     wrongAlarmStart \= millis();

# 

#   }

# 

#   // Clear entered password

#   enteredPassword \= "";

# 

#   Serial.println();

# }

# 

# 

# // \==================================================

# // SENSOR FUNCTION

# // \==================================================

# 

# void checkSensors() {

# 

#   Serial.println();

#   Serial.println("----------- SENSOR STATUS \-----------");

# 

# 

#   // \------------------------------------------------

#   // DHT11

#   // \------------------------------------------------

# 

#   float temperature \= dht.readTemperature();

#   float humidity \= dht.readHumidity();

# 

#   if (isnan(temperature) || isnan(humidity)) {

# 

#     Serial.println("DHT11 ERROR");

# 

#   }

# 

#   else {

# 

#     Serial.print("Temperature: ");

#     Serial.print(temperature);

#     Serial.println(" C");

# 

#     Serial.print("Humidity: ");

#     Serial.print(humidity);

#     Serial.println(" %");

#   }

# 

# 

#   // \------------------------------------------------

#   // MQ2

#   // \------------------------------------------------

# 

#   int gasValue \= analogRead(MQ2\_PIN);

# 

#   Serial.print("MQ2 Gas Value: ");

#   Serial.println(gasValue);

# 

# 

#   // \------------------------------------------------

#   // GAS DETECTION

#   // \------------------------------------------------

# 

#   if (gasValue \>= GAS\_THRESHOLD) {

# 

#     Serial.println("\!\!\! GAS/SMOKE DETECTED \!\!\!");

# 

#     // Buzzer ON

#     digitalWrite(BUZZER\_PIN, HIGH);

# 

#     // Relay OFF

#     digitalWrite(RELAY\_PIN, RELAY\_OFF);

# 

#     Serial.println("Relay: OFF");

#     Serial.println("Safety alarm: ON");

# 

#   }

# 

#   else {

# 

#     Serial.println("Gas Status: NORMAL");

# 

#     // Only turn buzzer off if wrong password alarm

#     // is not currently active.

#     if (\!wrongPasswordAlarm) {

# 

#       digitalWrite(BUZZER\_PIN, LOW);

#     }

# 

#     // Normal operation

#     digitalWrite(RELAY\_PIN, RELAY\_ON);

# 

#     Serial.println("Relay: ON");

#   }

# 

# 

#   // \------------------------------------------------

#   // ULTRASONIC

#   // \------------------------------------------------

# 

#   long duration;

#   float distance;

# 

#   digitalWrite(TRIG\_PIN, LOW);

#   delayMicroseconds(2);

# 

#   digitalWrite(TRIG\_PIN, HIGH);

#   delayMicroseconds(10);

# 

#   digitalWrite(TRIG\_PIN, LOW);

# 

#   duration \= pulseIn(ECHO\_PIN, HIGH, 30000);

# 

#   if (duration \== 0\) {

# 

#     Serial.println("Ultrasonic: No object");

# 

#   }

# 

#   else {

# 

#     distance \= duration \* 0.0343 / 2;

# 

#     Serial.print("Distance: ");

#     Serial.print(distance);

#     Serial.println(" cm");

# 

#     if (distance \< 30\) {

# 

#       Serial.println("Person detected near door");

# 

#     }

#   }

# 

# 

#   Serial.println("-------------------------------------");

# }

# 