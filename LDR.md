| Component | Pin | STM32 |
|---|---|---|
| LDR | AO | **PA0** |
| LDR | VCC | 3.3V |
| LDR | GND | GND |
| Button | one side | **PA1** |
| Button | other side | GND |
| Relay | IN | **PB0** |
| Relay | VCC | 5V |
| Relay | GND | GND |
| Relay | COM | 5V |
| Relay | NO | LED anode |
| LED | cathode | resistor → GND |


// =====================================================
// STM32 AUTOMATIC LIGHT CONTROLLER
// STM32F103C8T6 Blue Pill
//
// LDR       -> PA0
// Pushbutton-> PA1
// Relay     -> PB0
// =====================================================


//components
{
  "version": 1,
  "author": "STM32 Automatic Light Controller",
  "editor": "wokwi",
  "parts": [
    {
      "type": "board-stm32-bluepill",
      "id": "stm32",
      "top": 0,
      "left": 0,
      "attrs": {}
    },
    {
      "type": "wokwi-photoresistor-sensor",
      "id": "ldr",
      "top": -120,
      "left": -280,
      "attrs": {
        "lux": "100"
      }
    },
    {
      "type": "wokwi-pushbutton",
      "id": "button",
      "top": 100,
      "left": -250,
      "attrs": {
        "color": "blue"
      }
    },
    {
      "type": "wokwi-relay-module",
      "id": "relay",
      "top": -20,
      "left": 300,
      "attrs": {
        "transistor": "npn"
      }
    },
    {
      "type": "wokwi-led",
      "id": "led",
      "top": 170,
      "left": 420,
      "attrs": {
        "color": "white"
      }
    },
    {
      "type": "wokwi-resistor",
      "id": "resistor",
      "top": 220,
      "left": 420,
      "attrs": {
        "value": "220"
      }
    },
    {
      "type": "wokwi-vcc",
      "id": "vcc",
      "top": -160,
      "left": 300,
      "attrs": {}
    },
    {
      "type": "wokwi-gnd",
      "id": "gnd",
      "top": 300,
      "left": 200,
      "attrs": {}
    }
  ],
  "connections": [
    ["ldr:VCC", "stm32:3V3", "red", []],
    ["ldr:GND", "stm32:GND.1", "black", []],
    ["ldr:AO", "stm32:PA0", "green", []],

    ["button:1.l", "stm32:PA1", "blue", []],
    ["button:2.l", "stm32:GND.1", "black", []],

    ["relay:VCC", "stm32:5V", "red", []],
    ["relay:GND", "stm32:GND.1", "black", []],
    ["relay:IN", "stm32:PB0", "orange", []],

    ["vcc:VCC", "relay:COM", "red", []],
    ["relay:NO", "led:A", "orange", []],
    ["led:C", "resistor:1", "black", []],
    ["resistor:2", "stm32:GND.1", "black", []]
  ]
}


//code
#define LDR_PIN     PA0
#define BUTTON_PIN  PA1
#define RELAY_PIN   PB0

// Adjust this after checking Serial Monitor
#define LDR_THRESHOLD 600

void setup()
{
  Serial.begin(115200);

  // LDR
  pinMode(LDR_PIN, INPUT);

  // Button uses internal pull-up
  pinMode(BUTTON_PIN, INPUT_PULLUP);

  // Relay
  pinMode(RELAY_PIN, OUTPUT);

  // Relay OFF initially
  digitalWrite(RELAY_PIN, LOW);

  Serial.println("================================");
  Serial.println(" STM32 AUTOMATIC LIGHT SYSTEM");
  Serial.println("================================");
  Serial.println("System started...");
}

void loop()
{
  // -------------------------------------------------
  // READ LDR
  // -------------------------------------------------

  int ldrValue = analogRead(LDR_PIN);

  // -------------------------------------------------
  // READ BUTTON
  // -------------------------------------------------

  bool buttonPressed = (digitalRead(BUTTON_PIN) == LOW);

  // -------------------------------------------------
  // DETERMINE WHETHER IT IS DARK
  // -------------------------------------------------

  bool dark = (ldrValue > LDR_THRESHOLD);

  // -------------------------------------------------
  // LIGHT CONTROL
  // -------------------------------------------------

  bool lightON = false;

  if (dark)
  {
    // Automatically turn light ON when dark
    lightON = true;
  }
  else if (buttonPressed)
  {
    // User can manually turn light ON
    lightON = true;
  }
  else
  {
    // Bright + button not pressed
    lightON = false;
  }

  // -------------------------------------------------
  // CONTROL RELAY
  // -------------------------------------------------

  if (lightON)
  {
    digitalWrite(RELAY_PIN, HIGH);
  }
  else
  {
    digitalWrite(RELAY_PIN, LOW);
  }

  // -------------------------------------------------
  // SERIAL MONITOR
  // -------------------------------------------------

  Serial.print("LDR = ");
  Serial.print(ldrValue);

  Serial.print(" | ");

  if (dark)
    Serial.print("DARK");
  else
    Serial.print("BRIGHT");

  Serial.print(" | Button = ");

  if (buttonPressed)
    Serial.print("PRESSED");
  else
    Serial.print("RELEASED");

  Serial.print(" | Light = ");

  if (lightON)
    Serial.println("ON");
  else
    Serial.println("OFF");

  delay(200);
}




