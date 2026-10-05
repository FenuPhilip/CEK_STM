| Component | STM32 Pin |
|---|---|
| Candidate 1 button | PA0 |
| Candidate 2 button | PA1 |
| Candidate 3 button | PA2 |
| CAST button | PA3 |
| RESULTS button | PA4 |
| LCD SDA | PB7 |
| LCD SCL | PB6 |
| EEPROM CS | PB0 |
| EEPROM MOSI | PA7 |
| EEPROM MISO | PA6 |
| EEPROM SCK | PA5 |


json
{
  "version": 1,
  "author": "STM32 Digital Voting Machine",
  "editor": "wokwi",
  "parts": [
    { "type": "board-stm32-bluepill", "id": "stm32", "top": 20, "left": 250, "attrs": {} },
    { "type": "wokwi-lcd1602", "id": "lcd", "top": 300, "left": 300, "attrs": { "pins": "i2c" } },
    {
      "type": "wokwi-pushbutton",
      "id": "btn1",
      "top": 120,
      "left": 30,
      "attrs": { "color": "red", "label": "CANDIDATE 1" }
    },
    {
      "type": "wokwi-pushbutton",
      "id": "btn2",
      "top": 190,
      "left": 30,
      "attrs": { "color": "blue", "label": "CANDIDATE 2" }
    },
    {
      "type": "wokwi-pushbutton",
      "id": "btn3",
      "top": 260,
      "left": 30,
      "attrs": { "color": "green", "label": "CANDIDATE 3" }
    },
    {
      "type": "wokwi-pushbutton",
      "id": "cast",
      "top": 330,
      "left": 30,
      "attrs": { "color": "yellow", "label": "CAST" }
    },
    {
      "type": "wokwi-pushbutton",
      "id": "results",
      "top": 400,
      "left": 30,
      "attrs": { "color": "white", "label": "RESULTS" }
    }
  ],
  "connections": [
    [ "btn1:1.l", "stm32:PA0", "green", [] ],
    [ "btn1:2.l", "stm32:GND.1", "black", [] ],
    [ "btn2:1.l", "stm32:PA1", "green", [] ],
    [ "btn2:2.l", "stm32:GND.1", "black", [] ],
    [ "btn3:1.l", "stm32:PA2", "green", [] ],
    [ "btn3:2.l", "stm32:GND.1", "black", [] ],
    [ "cast:1.l", "stm32:PA3", "green", [] ],
    [ "cast:2.l", "stm32:GND.1", "black", [] ],
    [ "results:1.l", "stm32:PA4", "green", [] ],
    [ "results:2.l", "stm32:GND.1", "black", [] ],
    [ "lcd:SDA", "stm32:PB7", "green", [] ],
    [ "lcd:SCL", "stm32:PB6", "blue", [] ],
    [ "lcd:VCC", "stm32:5V.1", "red", [] ],
    [ "lcd:GND", "stm32:GND.1", "black", [] ],
    [ "btn1:1.r", "stm32:A0", "green", [ "v0" ] ],
    [ "btn2:1.r", "stm32:A1", "green", [ "v0" ] ],
    [ "btn3:1.r", "stm32:A2", "green", [ "v0" ] ],
    [ "cast:1.r", "stm32:A3", "green", [ "v0" ] ],
    [ "results:1.r", "stm32:A4", "green", [ "v0" ] ],
    [ "lcd:SCL", "stm32:B6", "green", [ "h0" ] ],
    [ "lcd:SDA", "stm32:B7", "green", [ "h0" ] ]
  ],
  "dependencies": {}
}



code

#include <Wire.h>
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);

// Buttons
#define CANDIDATE1 PA0
#define CANDIDATE2 PA1
#define CANDIDATE3 PA2
#define CAST_BUTTON PA3
#define RESULT_BUTTON PA4

// Vote counters
unsigned long vote1 = 0;
unsigned long vote2 = 0;
unsigned long vote3 = 0;

int selectedCandidate = 0;


// ------------------------------------
// Display home screen
// ------------------------------------

void showHome()
{
  lcd.clear();

  lcd.setCursor(0, 0);
  lcd.print("DIGITAL VOTING");

  lcd.setCursor(0, 1);
  lcd.print("Select Candidate");
}


// ------------------------------------
// Show selected candidate
// ------------------------------------

void selectCandidate(int candidate)
{
  selectedCandidate = candidate;

  lcd.clear();

  lcd.setCursor(0, 0);
  lcd.print("Candidate ");
  lcd.print(candidate);

  lcd.setCursor(0, 1);
  lcd.print("Press CAST");
}


// ------------------------------------
// Record vote
// ------------------------------------

void castVote()
{
  if (selectedCandidate == 0)
  {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Select Candidate");

    delay(1000);

    showHome();
    return;
  }

  if (selectedCandidate == 1)
  {
    vote1++;
  }
  else if (selectedCandidate == 2)
  {
    vote2++;
  }
  else if (selectedCandidate == 3)
  {
    vote3++;
  }

  lcd.clear();

  lcd.setCursor(0, 0);
  lcd.print("VOTE RECORDED!");

  lcd.setCursor(0, 1);
  lcd.print("Thank You");

  Serial.print("Vote for Candidate ");
  Serial.println(selectedCandidate);

  selectedCandidate = 0;

  delay(1500);

  showHome();
}


// ------------------------------------
// Show results
// ------------------------------------

void showResults()
{
  lcd.clear();

  lcd.setCursor(0, 0);
  lcd.print("C1:");
  lcd.print(vote1);

  lcd.setCursor(8, 0);
  lcd.print("C2:");
  lcd.print(vote2);

  lcd.setCursor(0, 1);
  lcd.print("C3:");
  lcd.print(vote3);

  Serial.println();
  Serial.println("===== RESULTS =====");

  Serial.print("Candidate 1: ");
  Serial.println(vote1);

  Serial.print("Candidate 2: ");
  Serial.println(vote2);

  Serial.print("Candidate 3: ");
  Serial.println(vote3);

  delay(5000);

  showHome();
}


// ------------------------------------
// Setup
// ------------------------------------

void setup()
{
  Serial.begin(115200);

  pinMode(CANDIDATE1, INPUT_PULLUP);
  pinMode(CANDIDATE2, INPUT_PULLUP);
  pinMode(CANDIDATE3, INPUT_PULLUP);

  pinMode(CAST_BUTTON, INPUT_PULLUP);
  pinMode(RESULT_BUTTON, INPUT_PULLUP);

  Wire.begin();

  lcd.init();
  lcd.backlight();

  lcd.clear();

  lcd.setCursor(0, 0);
  lcd.print("DIGITAL VOTING");

  lcd.setCursor(0, 1);
  lcd.print("SYSTEM READY");

  delay(2000);

  showHome();

  Serial.println("Digital Voting Machine Ready");
}


// ------------------------------------
// Main loop
// ------------------------------------

void loop()
{
  // Candidate 1
  if (digitalRead(CANDIDATE1) == LOW)
  {
    selectCandidate(1);
    delay(300);
  }

  // Candidate 2
  if (digitalRead(CANDIDATE2) == LOW)
  {
    selectCandidate(2);
    delay(300);
  }

  // Candidate 3
  if (digitalRead(CANDIDATE3) == LOW)
  {
    selectCandidate(3);
    delay(300);
  }

  // Cast vote
  if (digitalRead(CAST_BUTTON) == LOW)
  {
    castVote();
    delay(300);
  }

  // Results
  if (digitalRead(RESULT_BUTTON) == LOW)
  {
    showResults();
    delay(300);
  }
}