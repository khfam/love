#define DEBUG
#include "espConfig.h"
#include <BlynkSimpleEsp32.h>

int buzzer = 5;     // GPIO5
int relay = 4;      // GPIO4
int ledMode = 14;   // GPIO14
int button = 0;     // GPIO0
int mucCanhbao;
float mq2_value;
bool buttonState = HIGH;
bool runMode = 0;
bool canhbaoState = 0;
WidgetLED led(V0);
#define GAS V1
#define MUCCANHBAO V2
#define CANHBAO V3
#define CHEDO V4

BlynkTimer timer;
int timerID1, timerID2;

void setup() {
  Serial.begin(115200);
  pinMode(button, INPUT_PULLUP);
  pinMode(buzzer, OUTPUT);
  pinMode(relay, OUTPUT);
  pinMode(ledMode, OUTPUT);
  digitalWrite(buzzer, LOW);
  digitalWrite(relay, LOW);

  timerID1 = timer.setInterval(1000L, handleTimerID1);
  espConfig.begin();
}

void loop() {
  espConfig.run();
  app_loop();
}

void app_loop() {
  timer.run();
  if (digitalRead(button) == LOW) {
    if (buttonState == HIGH) {
      buttonState = LOW;
      runMode = !runMode;
      digitalWrite(ledMode, runMode);
      Serial.println("Run mode: " + String(runMode));
      Blynk.virtualWrite(CHEDO, runMode);
      delay(200);
    }
  } else {
    buttonState = HIGH;
  }
}

void handleTimerID1() {
  int mq2 = analogRead(34);  // Chuyển sang GPIO34 (ESP32 không có A0)
  float voltage = mq2 * (3.3 / 4095.0);  // ESP32 dùng 12-bit ADC và điện áp 3.3V
  float ratio = voltage / 1.4;
  mq2_value = 1000.0 * pow(10, ((log10(ratio) - 1.0278) / 0.6629));

  Serial.println("Gas: " + String(mq2_value, 0) + "ppm");
  Blynk.virtualWrite(GAS, mq2_value);

  led.setValue(!led.getValue()); // Nhấp nháy LED trạng thái trên Blynk

  if (runMode) {
    if (mq2_value > mucCanhbao) {
      if (!canhbaoState) {
        canhbaoState = 1;
        Blynk.logEvent("canhbao", "Cảnh báo! Khí gas = " + String(mq2_value) + "ppm vượt quá mức cho phép!");
        timerID2 = timer.setTimeout(60000L, handleTimerID2);
      }
      digitalWrite(buzzer, HIGH);
      digitalWrite(relay, HIGH);
      Blynk.virtualWrite(CANHBAO, HIGH);
      Serial.println("Đã bật cảnh báo!");
    } else {
      tatCanhBao();
    }
  } else {
    tatCanhBao();
  }
}

void tatCanhBao() {
  digitalWrite(buzzer, LOW);
  digitalWrite(relay, LOW);
  Blynk.virtualWrite(CANHBAO, LOW);
  Serial.println("Đã tắt cảnh báo!");
}

void handleTimerID2() {
  canhbaoState = 0;
}

BLYNK_CONNECTED() {
  Blynk.syncAll();
}

BLYNK_WRITE(MUCCANHBAO) {
  mucCanhbao = param.asInt();
}

BLYNK_WRITE(CHEDO) {
  runMode = param.asInt();
  digitalWrite(ledMode, runMode);
}
