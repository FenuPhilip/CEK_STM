| Function | STM32 |
|---|---|
| PC Serial TX → STM32 RX | **PA10** |
| PC Serial RX ← STM32 TX | **PA9** |
| LCD SCL | **PB6** |
| LCD SDA | **PB7** |
| LCD VCC | 5V |
| LCD GND | GND |


json
{
  "version": 1,
  "author": "STM32 Digital Notice Board",
  "editor": "wokwi",

  "parts": [
    {
      "type": "board-stm32-bluepill",
      "id": "stm32",
      "top": 40,
      "left": 40,
      "attrs": {}
    },

    {
      "type": "wokwi-lcd1602",
      "id": "lcd",
      "top": 250,
      "left": 300,
      "attrs": {
        "pins": "i2c"
      }
    }
  ],

  "connections": [

    ["lcd:SDA", "stm32:PB7", "green", []],
    ["lcd:SCL", "stm32:PB6", "blue", []],

    ["lcd:VCC", "stm32:5V.1", "red", []],
    ["lcd:GND", "stm32:GND.1", "black", []]
  ],

  "dependencies": {}
}



code
// =====================================================
// ARM-BASED DIGITAL NOTICE BOARD
// STM32F103C8T6 BLUE PILL
//
// USART1:
// PA9  -> TX
// PA10 -> RX
//
// LCD I2C:
// PB6 -> SCL
// PB7 -> SDA
//
// MESSAGE RECEIVED THROUGH UART
// IS DISPLAYED ON LCD
// =====================================================

#include <Wire.h>
#include <LiquidCrystal_I2C.h>


// =====================================================
// LCD
// =====================================================

LiquidCrystal_I2C lcd(0x27, 16, 2);


// =====================================================
// UART
// =====================================================

// USART1 pins
HardwareSerial Serial1(PA10, PA9);


// =====================================================
// MESSAGE BUFFER
// =====================================================

const int BUFFER_SIZE = 64;

volatile char messageBuffer[BUFFER_SIZE];

volatile int bufferIndex = 0;

volatile bool messageReady = false;


// =====================================================
// SETUP
// =====================================================

void setup()
{
  // ---------------------------------------------------
  // USB / PC Serial Monitor
  // ---------------------------------------------------

  Serial.begin(115200);


  // ---------------------------------------------------
  // USART1
  //
  // PA10 = RX
  // PA9  = TX
  // ---------------------------------------------------

  Serial1.begin(9600);


  // ---------------------------------------------------
  // I2C
  // ---------------------------------------------------

  Wire.begin();


  // ---------------------------------------------------
  // LCD
  // ---------------------------------------------------

  lcd.init();
  lcd.backlight();

  lcd.clear();

  lcd.setCursor(0, 0);
  lcd.print("DIGITAL NOTICE");

  lcd.setCursor(0, 1);
  lcd.print("BOARD READY");

  delay(2000);

  lcd.clear();


  // ---------------------------------------------------
  // Initial message
  // ---------------------------------------------------

  lcd.setCursor(0, 0);
  lcd.print("Send a message");

  lcd.setCursor(0, 1);
  lcd.print("via UART");


  Serial.println("==============================");
  Serial.println(" DIGITAL NOTICE BOARD");
  Serial.println("==============================");
  Serial.println("USART1 ready.");
  Serial.println("Send a message.");
}


// =====================================================
// MAIN LOOP
// =====================================================

void loop()
{
  // ---------------------------------------------------
  // Check for UART data
  // ---------------------------------------------------

  while (Serial1.available())
  {
    char c = Serial1.read();


    // -------------------------------------------------
    // ENTER / NEWLINE
    // -------------------------------------------------

    if (c == '\n' || c == '\r')
    {
      if (bufferIndex > 0)
      {
        messageBuffer[bufferIndex] = '\0';

        messageReady = true;

        bufferIndex = 0;
      }
    }


    // -------------------------------------------------
    // NORMAL CHARACTER
    // -------------------------------------------------

    else
    {
      if (bufferIndex < BUFFER_SIZE - 1)
      {
        messageBuffer[bufferIndex++] = c;
      }
    }
  }


  // ---------------------------------------------------
  // Display received message
  // ---------------------------------------------------

  if (messageReady)
  {
    noInterrupts();

    char localMessage[BUFFER_SIZE];

    strcpy(localMessage, (const char*)messageBuffer);

    messageReady = false;

    interrupts();


    // Display message
    displayMessage(localMessage);


    // Print to PC monitor
    Serial.print("Received: ");
    Serial.println(localMessage);
  }


  delay(10);
}


// =====================================================
// DISPLAY MESSAGE ON LCD
// =====================================================

void displayMessage(const char* message)
{
  lcd.clear();


  // ---------------------------------------------------
  // First 16 characters
  // ---------------------------------------------------

  lcd.setCursor(0, 0);

  for (int i = 0; i < 16 && message[i] != '\0'; i++)
  {
    lcd.print(message[i]);
  }


  // ---------------------------------------------------
  // Remaining characters
  // ---------------------------------------------------

  if (strlen(message) > 16)
  {
    lcd.setCursor(0, 1);

    for (int i = 16; i < 32 && message[i] != '\0'; i++)
    {
      lcd.print(message[i]);
    }
  }
}