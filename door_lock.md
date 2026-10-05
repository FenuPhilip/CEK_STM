| Component | Pin | STM32 |
|---|---|---|
| Keypad R1 | R1 | PA0 |
| Keypad R2 | R2 | PA1 |
| Keypad R3 | R3 | PA2 |
| Keypad R4 | R4 | PA3 |
| Keypad C1 | C1 | PA4 |
| Keypad C2 | C2 | PA5 |
| Keypad C3 | C3 | PA6 |
| Keypad C4 | C4 | PA7 |
| LCD | SDA | PB7 |
| LCD | SCL | PB6 |
| LCD | VCC | 5V |
| LCD | GND | GND |
| Buzzer | + | PB0 |
| Buzzer | − | GND |
| LED | Anode | PB1 through 220Ω |
| LED | Cathode | GND |


json
{
  "version": 1,
  "author": "STM32 Digital Door Lock",
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
      "type": "wokwi-membrane-keypad",
      "id": "keypad",
      "top": 20,
      "left": 350,
      "attrs": {}
    },

    {
      "type": "wokwi-lcd1602",
      "id": "lcd",
      "top": 260,
      "left": 350,
      "attrs": {
        "pins": "i2c"
      }
    },

    {
      "type": "wokwi-buzzer",
      "id": "buzzer",
      "top": 220,
      "left": 50,
      "attrs": {
        "volume": "0.2"
      }
    },

    {
      "type": "wokwi-led",
      "id": "led",
      "top": 300,
      "left": 80,
      "attrs": {
        "color": "green"
      }
    },

    {
      "type": "wokwi-resistor",
      "id": "resistor",
      "top": 350,
      "left": 80,
      "attrs": {
        "value": "220"
      }
    }
  ],

  "connections": [

    ["keypad:R1", "stm32:PA0", "blue", []],
    ["keypad:R2", "stm32:PA1", "blue", []],
    ["keypad:R3", "stm32:PA2", "blue", []],
    ["keypad:R4", "stm32:PA3", "blue", []],

    ["keypad:C1", "stm32:PA4", "green", []],
    ["keypad:C2", "stm32:PA5", "green", []],
    ["keypad:C3", "stm32:PA6", "green", []],
    ["keypad:C4", "stm32:PA7", "green", []],

    ["lcd:SDA", "stm32:PB7", "green", []],
    ["lcd:SCL", "stm32:PB6", "blue", []],
    ["lcd:VCC", "stm32:5V.1", "red", []],
    ["lcd:GND", "stm32:GND.1", "black", []],

    ["buzzer:1", "stm32:PB0", "orange", []],
    ["buzzer:2", "stm32:GND.1", "black", []],

    ["stm32:PB1", "resistor:1", "orange", []],
    ["resistor:2", "led:A", "orange", []],
    ["led:C", "stm32:GND.1", "black", []]
  ]
}

code
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <Keypad.h>

// =====================================================
// STM32 DIGITAL DOOR LOCK
// STM32F103C8T6 Blue Pill
// =====================================================

// -----------------------------
// LCD
// -----------------------------

LiquidCrystal_I2C lcd(0x27, 16, 2);


// -----------------------------
// KEYPAD
// -----------------------------

const byte ROWS = 4;
const byte COLS = 4;

char keys[ROWS][COLS] =
{
  {'1', '2', '3', 'A'},
  {'4', '5', '6', 'B'},
  {'7', '8', '9', 'C'},
  {'*', '0', '#', 'D'}
};


// Rows
byte rowPins[ROWS] =
{
  PA0,
  PA1,
  PA2,
  PA3
};


// Columns
byte colPins[COLS] =
{
  PA4,
  PA5,
  PA6,
  PA7
};


Keypad keypad = Keypad(
  makeKeymap(keys),
  rowPins,
  colPins,
  ROWS,
  COLS
);


// -----------------------------
// OUTPUTS
// -----------------------------

#define BUZZER_PIN PB0
#define DOOR_LED   PB1


// -----------------------------
// PASSWORD
// -----------------------------

String correctPassword = "1234";
String enteredPassword = "";


void setup()
{
  Serial.begin(115200);

  // Output pins
  pinMode(BUZZER_PIN, OUTPUT);
  pinMode(DOOR_LED, OUTPUT);

  // Initially locked
  digitalWrite(BUZZER_PIN, LOW);
  digitalWrite(DOOR_LED, LOW);


  // Start LCD
  Wire.begin();

  lcd.init();
  lcd.backlight();

  lcd.clear();

  lcd.setCursor(0, 0);
  lcd.print("DIGITAL DOOR");

  lcd.setCursor(0, 1);
  lcd.print("LOCK SYSTEM");

  delay(2000);

  showEnterScreen();
}


void loop()
{
  char key = keypad.getKey();

  if (key)
  {
    Serial.print("Key pressed: ");
    Serial.println(key);

    // ==========================================
    // ENTER KEY
    // ==========================================

    if (key == '#')
    {
      checkPassword();
    }


    // ==========================================
    // CLEAR KEY
    // ==========================================

    else if (key == '*')
    {
      enteredPassword = "";

      showEnterScreen();

      Serial.println("Password cleared");
    }


    // ==========================================
    // NUMBER / OTHER KEY
    // ==========================================

    else
    {
      if (enteredPassword.length() < 10)
      {
        enteredPassword += key;

        lcd.setCursor(0, 1);

        lcd.print("                ");

        lcd.setCursor(0, 1);

        // Show * instead of actual password
        for (int i = 0; i < enteredPassword.length(); i++)
        {
          lcd.print("*");
        }
      }
    }
  }
}


// =====================================================
// SHOW PASSWORD ENTRY SCREEN
// =====================================================

void showEnterScreen()
{
  lcd.clear();

  lcd.setCursor(0, 0);
  lcd.print("Enter Password:");

  lcd.setCursor(0, 1);
  lcd.print("");
}


// =====================================================
// CHECK PASSWORD
// =====================================================

void checkPassword()
{
  lcd.clear();

  if (enteredPassword == correctPassword)
  {
    // ==========================================
    // CORRECT PASSWORD
    // ==========================================

    Serial.println("ACCESS GRANTED");

    lcd.setCursor(0, 0);
    lcd.print("ACCESS GRANTED");

    lcd.setCursor(0, 1);
    lcd.print("DOOR OPEN");

    // Door opens
    digitalWrite(DOOR_LED, HIGH);

    // Short success beep
    tone(BUZZER_PIN, 2000, 200);


    // Keep door open for 5 seconds
    delay(5000);


    // Lock door again
    digitalWrite(DOOR_LED, LOW);

    noTone(BUZZER_PIN);

    enteredPassword = "";

    showEnterScreen();
  }

  else
  {
    // ==========================================
    // WRONG PASSWORD
    // ==========================================

    Serial.println("ACCESS DENIED");

    lcd.setCursor(0, 0);
    lcd.print("ACCESS DENIED");

    lcd.setCursor(0, 1);
    lcd.print("WRONG PASSWORD");

    // Three warning beeps
    tone(BUZZER_PIN, 1000);

    delay(200);

    noTone(BUZZER_PIN);

    delay(100);

    tone(BUZZER_PIN, 1000);

    delay(200);

    noTone(BUZZER_PIN);

    delay(100);

    tone(BUZZER_PIN, 1000);

    delay(200);

    noTone(BUZZER_PIN);

    delay(1500);

    enteredPassword = "";

    showEnterScreen();
  }
}