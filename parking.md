                       STM32 BLUE PILL
                    ┌───────────────────┐
                    │                   │
 IR SENSOR 1 ──────► PA0              │
 IR SENSOR 2 ──────► PA1              │
                    │                   │
 GREEN 1 ◄────────── PB0              │
 RED 1   ◄────────── PB1              │
 GREEN 2 ◄────────── PB2              │
 RED 2   ◄────────── PB3              │
                    │                   │
 LCD SDA ◄─────────► PB7              │
 LCD SCL ◄─────────► PB6              │
                    │                   │
                    │ 5V                │
                    │ GND               │
                    └───────────────────┘


        SLOT 1                      SLOT 2

   ┌─────────────┐            ┌─────────────┐
   │             │            │             │
   │ IR SENSOR   │            │ IR SENSOR   │
   │      │      │            │      │      │
   └──────┼──────┘            └──────┼──────┘
          │                          │
         PA0                        PA1


   PB0 ──220Ω──🟢                 PB2 ──220Ω──🟢
   PB1 ──220Ω──🔴                 PB3 ──220Ω──🔴

        SLOT 1                 SLOT 2

    🟢 FREE / 🔴 OCCUPIED    🟢 FREE / 🔴 OCCUPIED


| Function | Pin |
|---|---|
| IR Sensor 1 | **PA0** |
| IR Sensor 2 | **PA1** |
| Green LED 1 | **PB0** |
| Red LED 1 | **PB1** |
| Green LED 2 | **PB10** |
| Red LED 2 | **PB11** |
| LCD SDA | **PB7** |
| LCD SCL | **PB6** |

json
{
  "version": 1,
  "author": "STM32 Smart Parking",
  "editor": "wokwi",

  "parts": [
    {
      "type": "board-stm32-bluepill",
      "id": "stm32",
      "top": 20,
      "left": 20,
      "attrs": {}
    },

    {
      "type": "wokwi-pushbutton",
      "id": "ir1",
      "top": 30,
      "left": 350,
      "attrs": {
        "color": "blue",
        "label": "IR SLOT 1"
      }
    },

    {
      "type": "wokwi-pushbutton",
      "id": "ir2",
      "top": 120,
      "left": 350,
      "attrs": {
        "color": "blue",
        "label": "IR SLOT 2"
      }
    },

    {
      "type": "wokwi-led",
      "id": "green1",
      "top": 30,
      "left": 600,
      "attrs": {
        "color": "green"
      }
    },

    {
      "type": "wokwi-resistor",
      "id": "resGreen1",
      "top": 70,
      "left": 600,
      "attrs": {
        "value": "220"
      }
    },

    {
      "type": "wokwi-led",
      "id": "red1",
      "top": 110,
      "left": 600,
      "attrs": {
        "color": "red"
      }
    },

    {
      "type": "wokwi-resistor",
      "id": "resRed1",
      "top": 150,
      "left": 600,
      "attrs": {
        "value": "220"
      }
    },

    {
      "type": "wokwi-led",
      "id": "green2",
      "top": 210,
      "left": 600,
      "attrs": {
        "color": "green"
      }
    },

    {
      "type": "wokwi-resistor",
      "id": "resGreen2",
      "top": 250,
      "left": 600,
      "attrs": {
        "value": "220"
      }
    },

    {
      "type": "wokwi-led",
      "id": "red2",
      "top": 290,
      "left": 600,
      "attrs": {
        "color": "red"
      }
    },

    {
      "type": "wokwi-resistor",
      "id": "resRed2",
      "top": 330,
      "left": 600,
      "attrs": {
        "value": "220"
      }
    },

    {
      "type": "wokwi-lcd1602",
      "id": "lcd",
      "top": 400,
      "left": 300,
      "attrs": {
        "pins": "i2c"
      }
    }
  ],

  "connections": [

    ["ir1:1.l", "stm32:PA0", "blue", []],
    ["ir1:2.l", "stm32:GND.1", "black", []],

    ["ir2:1.l", "stm32:PA1", "blue", []],
    ["ir2:2.l", "stm32:GND.1", "black", []],

    ["stm32:PB0", "resGreen1:1", "green", []],
    ["resGreen1:2", "green1:A", "green", []],
    ["green1:C", "stm32:GND.1", "black", []],

    ["stm32:PB1", "resRed1:1", "red", []],
    ["resRed1:2", "red1:A", "red", []],
    ["red1:C", "stm32:GND.1", "black", []],

    ["stm32:PB2", "resGreen2:1", "green", []],
    ["resGreen2:2", "green2:A", "green", []],
    ["green2:C", "stm32:GND.1", "black", []],

    ["stm32:PB3", "resRed2:1", "red", []],
    ["resRed2:2", "red2:A", "red", []],
    ["red2:C", "stm32:GND.1", "black", []],

    ["lcd:SDA", "stm32:PB7", "green", []],
    ["lcd:SCL", "stm32:PB6", "blue", []],
    ["lcd:VCC", "stm32:5V.1", "red", []],
    ["lcd:GND", "stm32:GND.1", "black", []]
  ],

  "dependencies": {}
}


code
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// =====================================================
// STM32 SMART PARKING SLOT INDICATOR
// STM32F103C8T6
//
// IR Sensor 1 -> PA0
// IR Sensor 2 -> PA1
//
// Slot 1 Green -> PB0
// Slot 1 Red   -> PB1
//
// Slot 2 Green -> PB2
// Slot 2 Red   -> PB3
//
// LCD SDA -> PB7
// LCD SCL -> PB6
// =====================================================

#define IR_SLOT1 PA0
#define IR_SLOT2 PA1

#define GREEN_SLOT1 PB0
#define RED_SLOT1   PB1

#define GREEN_SLOT2 PB10
#define RED_SLOT2   PB11

LiquidCrystal_I2C lcd(0x27, 16, 2);


void setup()
{
  Serial.begin(115200);

  // ==========================================
  // IR SENSOR INPUTS
  // ==========================================

  pinMode(IR_SLOT1, INPUT_PULLUP);
  pinMode(IR_SLOT2, INPUT_PULLUP);


  // ==========================================
  // LED OUTPUTS
  // ==========================================

  pinMode(GREEN_SLOT1, OUTPUT);
  pinMode(RED_SLOT1, OUTPUT);

  pinMode(GREEN_SLOT2, OUTPUT);
  pinMode(RED_SLOT2, OUTPUT);


  // ==========================================
  // INITIALIZE LEDs
  // ==========================================

  digitalWrite(GREEN_SLOT1, LOW);
  digitalWrite(RED_SLOT1, LOW);

  digitalWrite(GREEN_SLOT2, LOW);
  digitalWrite(RED_SLOT2, LOW);


  // ==========================================
  // LCD
  // ==========================================

  Wire.begin();

  lcd.init();
  lcd.backlight();

  lcd.clear();

  lcd.setCursor(0, 0);
  lcd.print("SMART PARKING");

  lcd.setCursor(0, 1);
  lcd.print("Initializing...");

  delay(2000);

  lcd.clear();
}


void loop()
{
  // ==========================================
  // READ IR SENSORS
  // ==========================================

  bool slot1Occupied =
    (digitalRead(IR_SLOT1) == LOW);

  bool slot2Occupied =
    (digitalRead(IR_SLOT2) == LOW);


  // ==========================================
  // SLOT 1
  // ==========================================

  if (slot1Occupied)
  {
    // Vehicle detected

    digitalWrite(GREEN_SLOT1, LOW);
    digitalWrite(RED_SLOT1, HIGH);
  }
  else
  {
    // No vehicle

    digitalWrite(GREEN_SLOT1, HIGH);
    digitalWrite(RED_SLOT1, LOW);
  }


  // ==========================================
  // SLOT 2
  // ==========================================

  if (slot2Occupied)
  {
    // Vehicle detected

    digitalWrite(GREEN_SLOT2, LOW);
    digitalWrite(RED_SLOT2, HIGH);
  }
  else
  {
    // No vehicle

    digitalWrite(GREEN_SLOT2, HIGH);
    digitalWrite(RED_SLOT2, LOW);
  }


  // ==========================================
  // LCD DISPLAY
  // ==========================================

  lcd.setCursor(0, 0);

  lcd.print("S1:");

  if (slot1Occupied)
    lcd.print("OCCUPIED ");
  else
    lcd.print("FREE     ");


  lcd.setCursor(0, 1);

  lcd.print("S2:");

  if (slot2Occupied)
    lcd.print("OCCUPIED ");
  else
    lcd.print("FREE     ");


  // ==========================================
  // SERIAL MONITOR
  // ==========================================

  Serial.print("Slot 1: ");

  if (slot1Occupied)
    Serial.print("OCCUPIED");
  else
    Serial.print("FREE");

  Serial.print(" | Slot 2: ");

  if (slot2Occupied)
    Serial.println("OCCUPIED");
  else
    Serial.println("FREE");


  delay(200);
}