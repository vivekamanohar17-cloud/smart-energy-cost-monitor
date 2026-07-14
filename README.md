# smart-energy-cost-monitor
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64

Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);

// Pin Configuration
const int sensorPin = 34;      // ACS712 Output
const float voltage = 230.0;   // AC Voltage
const float sensitivity = 0.100; // ACS712-20A = 100mV/A
const float offset = 2.5;      // Zero Current Voltage

float current = 0;
float power = 0;
float energy = 0;

unsigned long previousMillis = 0;

void setup()
{
  Serial.begin(115200);

  Wire.begin(21,22);      // SDA, SCL

  if(!display.begin(SSD1306_SWITCHCAPVCC, 0x3C))
  {
    Serial.println("OLED Failed");
    while(true);
  }

  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(WHITE);

  display.setCursor(10,20);
  display.println("ElectroPulse");
  display.setCursor(25,35);
  display.println("Meter");
  display.display();

  delay(2000);
}

void loop()
{
  int adcValue = analogRead(sensorPin);

  float sensorVoltage = adcValue * (3.3 / 4095.0);

  current = abs((sensorVoltage - 2.5) / sensitivity);

  power = voltage * current;

  unsigned long currentMillis = millis();

  float hours = (currentMillis - previousMillis) / 3600000.0;

  energy += power * hours / 1000.0;

  previousMillis = currentMillis;

  Serial.print("Current : ");
  Serial.print(current);
  Serial.println(" A");

  Serial.print("Power : ");
  Serial.print(power);
  Serial.println(" W");

  Serial.print("Energy : ");
  Serial.print(energy);
  Serial.println(" kWh");

  display.clearDisplay();

  display.setTextSize(1);

  display.setCursor(0,0);
  display.println("ElectroPulse");

  display.setCursor(0,18);
  display.print("Current:");
  display.print(current,2);
  display.println("A");

  display.setCursor(0,32);
  display.print("Power:");
  display.print(power,1);
  display.println("W");

  display.setCursor(0,46);
  display.print("Energy:");
  display.print(energy,3);
  display.println("kWh");

  display.display();

  delay(1000);
}
