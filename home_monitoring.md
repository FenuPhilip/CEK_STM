| Device | STM32 Pin |
|---|---|
| Light virtual switch | PB0 |
| Fan virtual switch | PB1 |
| TV virtual switch | PB2 |
| Light LED | PA0 |
| Fan LED | PA1 |
| TV LED | PA2 |

                       STM32 BLUE PILL
                    ┌─────────────────────┐
                    │                     │
 Light Switch ─────►│ PB0                │
 Fan Switch ───────►│ PB1                │
 TV Switch ────────►│ PB2                │
                    │                     │
                    │                     │
 Light LED ◄────────│ PA0                │
 Fan LED ◄──────────│ PA1                │
 TV LED ◄───────────│ PA2                │
                    │                     │
                    │ GND                │
                    └─────────┬───────────┘
                              │
                              │
                    ┌─────────┼──────────┐
                    │         │          │
                    ▼         ▼          ▼
                  SWITCH    SWITCH     SWITCH
                    │         │          │
                   GND       GND        GND


              PA0 ──220Ω──► LED ──► GND
              PA1 ──220Ω──► LED ──► GND
              PA2 ──220Ω──► LED ──► GND


json
{
  "version": 1,
  "author": "STM32 IoT Smart Home",
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
      "id": "lightSwitch",
      "top": 30,
      "left": 350,
      "attrs": {
        "color": "yellow",
        "label": "LIGHT"
      }
    },

    {
      "type": "wokwi-pushbutton",
      "id": "fanSwitch",
      "top": 120,
      "left": 350,
      "attrs": {
        "color": "blue",
        "label": "FAN"
      }
    },

    {
      "type": "wokwi-pushbutton",
      "id": "tvSwitch",
      "top": 210,
      "left": 350,
      "attrs": {
        "color": "red",
        "label": "TV"
      }
    },

    {
      "type": "wokwi-led",
      "id": "lightLED",
      "top": 30,
      "left": 600,
      "attrs": {
        "color": "yellow"
      }
    },

    {
      "type": "wokwi-resistor",
      "id": "lightResistor",
      "top": 70,
      "left": 600,
      "attrs": {
        "value": "220"
      }
    },

    {
      "type": "wokwi-led",
      "id": "fanLED",
      "top": 130,
      "left": 600,
      "attrs": {
        "color": "blue"
      }
    },

    {
      "type": "wokwi-resistor",
      "id": "fanResistor",
      "top": 170,
      "left": 600,
      "attrs": {
        "value": "220"
      }
    },

    {
      "type": "wokwi-led",
      "id": "tvLED",
      "top": 230,
      "left": 600,
      "attrs": {
        "color": "red"
      }
    },

    {
      "type": "wokwi-resistor",
      "id": "tvResistor",
      "top": 270,
      "left": 600,
      "attrs": {
        "value": "220"
      }
    }
  ],

  "connections": [

    ["lightSwitch:1.l", "stm32:PB0", "green", []],
    ["lightSwitch:2.l", "stm32:GND.1", "black", []],

    ["fanSwitch:1.l", "stm32:PB1", "blue", []],
    ["fanSwitch:2.l", "stm32:GND.1", "black", []],

    ["tvSwitch:1.l", "stm32:PB2", "red", []],
    ["tvSwitch:2.l", "stm32:GND.1", "black", []],

    ["stm32:PA0", "lightResistor:1", "yellow", []],
    ["lightResistor:2", "lightLED:A", "yellow", []],
    ["lightLED:C", "stm32:GND.1", "black", []],

    ["stm32:PA1", "fanResistor:1", "blue", []],
    ["fanResistor:2", "fanLED:A", "blue", []],
    ["fanLED:C", "stm32:GND.1", "black", []],

    ["stm32:PA2", "tvResistor:1", "red", []],
    ["tvResistor:2", "tvLED:A", "red", []],
    ["tvLED:C", "stm32:GND.1", "black", []]
  ],

  "dependencies": {}
}


code
// =====================================================
// STM32 IoT SMART HOME MONITORING
// STM32F103C8T6 Blue Pill
//
// Virtual IoT switches:
// PB0 -> Light switch
// PB1 -> Fan switch
// PB2 -> TV switch
//
// Appliance outputs:
// PA0 -> Light LED
// PA1 -> Fan LED
// PA2 -> TV LED
// =====================================================


// -----------------------------
// Virtual IoT Inputs
// -----------------------------

#define LIGHT_SWITCH PB0
#define FAN_SWITCH   PB1
#define TV_SWITCH    PB2


// -----------------------------
// Appliance Outputs
// -----------------------------

#define LIGHT_LED PA0
#define FAN_LED   PA1
#define TV_LED    PA2


void setup()
{
  Serial.begin(115200);


  // ==========================================
  // VIRTUAL IoT SWITCHES
  // ==========================================

  pinMode(LIGHT_SWITCH, INPUT_PULLUP);
  pinMode(FAN_SWITCH, INPUT_PULLUP);
  pinMode(TV_SWITCH, INPUT_PULLUP);


  // ==========================================
  // APPLIANCE OUTPUTS
  // ==========================================

  pinMode(LIGHT_LED, OUTPUT);
  pinMode(FAN_LED, OUTPUT);
  pinMode(TV_LED, OUTPUT);


  // ==========================================
  // INITIAL STATE
  // ==========================================

  digitalWrite(LIGHT_LED, LOW);
  digitalWrite(FAN_LED, LOW);
  digitalWrite(TV_LED, LOW);


  Serial.println("==============================");
  Serial.println("  STM32 IoT SMART HOME");
  Serial.println("==============================");

  Serial.println("System Ready");
}


void loop()
{
  // ==========================================
  // READ VIRTUAL IoT COMMANDS
  // ==========================================

  bool lightCommand =
    (digitalRead(LIGHT_SWITCH) == LOW);

  bool fanCommand =
    (digitalRead(FAN_SWITCH) == LOW);

  bool tvCommand =
    (digitalRead(TV_SWITCH) == LOW);


  // ==========================================
  // CONTROL APPLIANCES
  // ==========================================

  digitalWrite(
    LIGHT_LED,
    lightCommand ? HIGH : LOW
  );

  digitalWrite(
    FAN_LED,
    fanCommand ? HIGH : LOW
  );

  digitalWrite(
    TV_LED,
    tvCommand ? HIGH : LOW
  );


  // ==========================================
  // DISPLAY STATUS IN SERIAL MONITOR
  // ==========================================

  Serial.print("Light: ");

  if (lightCommand)
    Serial.print("ON");
  else
    Serial.print("OFF");


  Serial.print(" | Fan: ");

  if (fanCommand)
    Serial.print("ON");
  else
    Serial.print("OFF");


  Serial.print(" | TV: ");

  if (tvCommand)
    Serial.println("ON");
  else
    Serial.println("OFF");


  delay(200);
}