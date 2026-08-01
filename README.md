/*************************************************
 SMART OFFICE V1.0
 ESP32 + Blynk + Relay 4 Channel
*************************************************/

#define BLYNK_TEMPLATE_ID "TMPL640DZu4lv"
#define BLYNK_TEMPLATE_NAME "control Perangkat Listrik"
#define BLYNK_AUTH_TOKEN "ADx7yX02hqUNF0awvsB1EAFj5O1ApOdp"

#include <WiFi.h>
#include <BlynkSimpleEsp32.h>
#include <PZEM004Tv30.h>
#include <DHT.h>

char ssid[] = "POCO1";
char pass[] = "12345678";

//===================== RELAY =====================
#define RELAY1 26
#define RELAY2 27
#define RELAY3 14
#define RELAY4 12

// DHT
#define DHTPIN 4
#define DHTTYPE DHT11

// MQ2
#define MQ2_PIN 34

// Buzzer
#define BUZZER 25

// PZEM
#define PZEM_RX 16
#define PZEM_TX 17

HardwareSerial PZEMSerial(2);
PZEM004Tv30 pzem(PZEMSerial, PZEM_RX, PZEM_TX);

DHT dht(DHTPIN, DHTTYPE);

BlynkTimer timer;

//===================== SETUP =====================
void setup()
{
  Serial.begin(115200);

  pinMode(RELAY1, OUTPUT);
  pinMode(RELAY2, OUTPUT);
  pinMode(RELAY3, OUTPUT);
  pinMode(RELAY4, OUTPUT);

  // Relay OFF (Active LOW)
  digitalWrite(RELAY1, HIGH);
  digitalWrite(RELAY2, HIGH);
  digitalWrite(RELAY3, HIGH);
  digitalWrite(RELAY4, HIGH);

  Serial.println("================================");
  Serial.println(" SMART OFFICE V1.0");
  Serial.println(" WiFi + Blynk + Relay");
  Serial.println("================================");

  Blynk.begin(BLYNK_AUTH_TOKEN, ssid, pass);

  Serial.println("Blynk Connected");
}

//===================== RELAY 1 =====================
BLYNK_WRITE(V7)
{
  int state = param.asInt();

  digitalWrite(RELAY1, !state);

  Serial.print("Relay 1 : ");
  Serial.println(state ? "ON" : "OFF");
}

//===================== RELAY 2 =====================
BLYNK_WRITE(V8)
{
  int state = param.asInt();

  digitalWrite(RELAY2, !state);

  Serial.print("Relay 2 : ");
  Serial.println(state ? "ON" : "OFF");
}

//===================== RELAY 3 =====================
BLYNK_WRITE(V9)
{
  int state = param.asInt();

  digitalWrite(RELAY3, !state);

  Serial.print("Relay 3 : ");
  Serial.println(state ? "ON" : "OFF");
}

//===================== RELAY 4 =====================
BLYNK_WRITE(V10)
{
  int state = param.asInt();

  digitalWrite(RELAY4, !state);

  Serial.print("Relay 4 : ");
  Serial.println(state ? "ON" : "OFF");
}
void readPZEM()
{
  float voltage = pzem.voltage();
  float current = pzem.current();
  float power = pzem.power();
  float energy = pzem.energy();
  float frequency = pzem.frequency();
  float pf = pzem.pf();

  if (!isnan(voltage))
  {
    Serial.println("========== PZEM ==========");
    Serial.print("Voltage   : ");
    Serial.print(voltage);
    Serial.println(" V");

    Serial.print("Current   : ");
    Serial.print(current);
    Serial.println(" A");

    Serial.print("Power     : ");
    Serial.print(power);
    Serial.println(" W");

    Serial.print("Energy    : ");
    Serial.print(energy);
    Serial.println(" kWh");

    Serial.print("Frequency : ");
    Serial.print(frequency);
    Serial.println(" Hz");

    Serial.print("PF        : ");
    Serial.println(pf);

    Serial.println("==========================");

    // Kirim ke Blynk
    Blynk.virtualWrite(V1, voltage);
    Blynk.virtualWrite(V2, current);
    Blynk.virtualWrite(V3, power);
    Blynk.virtualWrite(V4, energy);
    Blynk.virtualWrite(V11, frequency);
    Blynk.virtualWrite(V12, pf);
  }
  else
  {
    Serial.println("PZEM tidak terdeteksi");
  }
}
void readDHT()
{
  float suhu = dht.readTemperature();
  float kelembaban = dht.readHumidity();

  if (!isnan(suhu) && !isnan(kelembaban))
  {
    Serial.println("===== DHT11 =====");
    Serial.print("Suhu : ");
    Serial.print(suhu);
    Serial.println(" °C");

    Serial.print("Humidity : ");
    Serial.print(kelembaban);
    Serial.println(" %");

    Blynk.virtualWrite(V5, suhu);
    Blynk.virtualWrite(V13, kelembaban);
  }
  else
  {
    Serial.println("DHT11 Error");
  }
}
void readMQ2()
{
  int gas = analogRead(MQ2_PIN);

  Serial.println("===== MQ2 =====");
  Serial.print("Gas Value : ");
  Serial.println(gas);

  Blynk.virtualWrite(V6, gas);

  if (gas > 2000)
  {
    digitalWrite(BUZZER, HIGH);

    Serial.println("!!! GAS TERDETEKSI !!!");

    Blynk.logEvent("gas_alert", "Gas terdeteksi di ruangan!");
  }
  else
  {
    digitalWrite(BUZZER, LOW);
  }
}
//===================== LOOP =====================
void loop()
{
  Blynk.run();
  timer.run();
}
