#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// -------------------------------
// Pin definitions
// -------------------------------

#define LM35_PIN PA0

// I2C LCD
// STM32 I2C1:
// PB6 = SCL
// PB7 = SDA

LiquidCrystal_I2C lcd(0x27, 16, 2);

void setup()
{
  Serial.begin(115200);

  // Start I2C
  Wire.begin();

  // Start LCD
  lcd.init();
  lcd.backlight();

  // Configure LM35
  pinMode(LM35_PIN, INPUT);

  // Startup message
  lcd.setCursor(0, 0);
  lcd.print("Temperature");
  
  lcd.setCursor(0, 1);
  lcd.print("Monitoring...");

  delay(2000);

  lcd.clear();
}

void loop()
{
  // --------------------------------
  // Read ADC
  // --------------------------------

  int adcValue = analogRead(LM35_PIN);

  // --------------------------------
  // Convert ADC to voltage
  // --------------------------------

  float voltage = (adcValue * 3.3) / 4095.0;

  // --------------------------------
  // Convert voltage to temperature
  //
  // LM35:
  // 10mV = 1 degree C
  // --------------------------------

  float temperature = voltage * 100.0;

  // --------------------------------
  // Display on LCD
  // --------------------------------

  lcd.setCursor(0, 0);
  lcd.print("Temperature:   ");

  lcd.setCursor(0, 1);

  lcd.print(temperature, 1);
  lcd.write(223);   // Degree symbol
  lcd.print("C");

  // --------------------------------
  // Serial Monitor
  // --------------------------------

  Serial.print("ADC = ");
  Serial.print(adcValue);

  Serial.print(" | Voltage = ");
  Serial.print(voltage, 3);

  Serial.print(" V | Temperature = ");
  Serial.print(temperature, 1);

  Serial.println(" C");

  delay(1000);
}


wiring
| Component | Pin | STM32 |
|---|---|---|
| LM35 | OUT | **PA0** |
| LM35 | VCC | **3.3V** |
| LM35 | GND | **GND** |
| LCD | SDA | **PB7** |
| LCD | SCL | **PB6** |
| LCD | VCC | **5V** |
| LCD | GND | **GND** |


{
  "version": 1,
  "author": "STM32 Temperature Monitor",
  "editor": "wokwi",

  "parts": [
    {
      "type": "board-stm32-bluepill",
      "id": "stm32",
      "top": 40,
      "left": 20,
      "attrs": {}
    },

    {
      "type": "wokwi-potentiometer",
      "id": "lm35",
      "top": 40,
      "left": 350,
      "attrs": {
        "value": "310"
      }
    },

    {
      "type": "wokwi-lcd1602",
      "id": "lcd",
      "top": 230,
      "left": 300,
      "attrs": {
        "pins": "i2c"
      }
    }
  ],

  "connections": [

    ["lm35:VCC", "stm32:3V3", "red", []],
    ["lm35:GND", "stm32:GND.1", "black", []],
    ["lm35:SIG", "stm32:PA0", "green", []],

    ["lcd:VCC", "stm32:5V.1", "red", []],
    ["lcd:GND", "stm32:GND.1", "black", []],

    ["lcd:SCL", "stm32:PB6", "blue", []],
    ["lcd:SDA", "stm32:PB7", "green", []]
  ],

  "dependencies": {}
}