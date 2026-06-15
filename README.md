#include "BluetoothSerial.h"

BluetoothSerial SerialBT;

// ── Pin Definitions ──
#define M_INDEX_IN1 25
#define M_INDEX_IN2 26
#define M_THUMB_IN1 27
#define M_THUMB_IN2 14

// ── Tune These ──
#define SPINS         1
#define RPM           80
#define SPEED_INDEX   50
#define SPEED_THUMB   60
#define RUN_TIME      2000

bool isClosed = false;

void stopAll() {
  analogWrite(M_INDEX_IN1, 0); analogWrite(M_INDEX_IN2, 0);
  analogWrite(M_THUMB_IN1, 0); analogWrite(M_THUMB_IN2, 0);
  digitalWrite(M_INDEX_IN1, LOW); digitalWrite(M_INDEX_IN2, LOW);
  digitalWrite(M_THUMB_IN1, LOW); digitalWrite(M_THUMB_IN2, LOW);
  Serial.println("STOPPED");
  SerialBT.println("STOPPED");
}

void forward() {
  analogWrite(M_INDEX_IN1, SPEED_INDEX); analogWrite(M_INDEX_IN2, 0);
  analogWrite(M_THUMB_IN1, 0);           analogWrite(M_THUMB_IN2, SPEED_THUMB);
  Serial.println("CLOSING...");
  SerialBT.println("CLOSING...");
}

void reverse() {
  analogWrite(M_INDEX_IN1, 0);           analogWrite(M_INDEX_IN2, SPEED_INDEX);
  analogWrite(M_THUMB_IN1, SPEED_THUMB); analogWrite(M_THUMB_IN2, 0);
  Serial.println("OPENING...");
  SerialBT.println("OPENING...");
}

void setup() {
  Serial.begin(115200);
  SerialBT.begin("BionicArm");

  pinMode(M_INDEX_IN1, OUTPUT); pinMode(M_INDEX_IN2, OUTPUT);
  pinMode(M_THUMB_IN1, OUTPUT); pinMode(M_THUMB_IN2, OUTPUT);

  stopAll();
  Serial.println("Ready — C=Close  O=Open");
  SerialBT.println("Ready — C=Close  O=Open");
}

void loop() {
  char cmd = 0;

  // Accept from both Serial Monitor and Bluetooth
  if (Serial.available())   cmd = toupper(Serial.read());
  if (SerialBT.available()) cmd = toupper(SerialBT.read());

  if (cmd == 'C' && !isClosed) {
    forward();
    delay(RUN_TIME);
    stopAll();
    isClosed = true;

  } else if (cmd == 'O' && isClosed) {
    reverse();
    delay(RUN_TIME);
    stopAll();
    isClosed = false;

  } else if (cmd == 'C' && isClosed) {
    Serial.println("ALREADY CLOSED");
    SerialBT.println("ALREADY CLOSED");

  } else if (cmd == 'O' && !isClosed) {
    Serial.println("ALREADY OPEN");
    SerialBT.println("ALREADY OPEN");
  }
}
