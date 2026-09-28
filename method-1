enum State { S_GREEN, S_YELLOW, S_RED, S_WARNING };

State st = S_GREEN;
unsigned long tStart = 0, dur = 0;
bool pedReq = false, emerg = false, night = false;
bool blinkOn = false;
unsigned long lastBlink = 0;
bool btnPrev = false;
unsigned long btnTime = 0;

void setOut(State s) {
  digitalWrite(13, s == S_RED);
  digitalWrite(12, s == S_YELLOW);
  digitalWrite(11, s == S_GREEN);
}

void goTo(State s, unsigned long d) {
  st = s;
  tStart = millis();
  dur = d;
  setOut(s);

  if (s == S_WARNING) {
    blinkOn = false;
    lastBlink = millis();
  }

  Serial.print(millis());
  Serial.print(" -> ");
  Serial.println(s);
}

void readInputs() {
  bool b = !digitalRead(2);

  if (b && !btnPrev) {
    btnTime = millis();
    btnPrev = true;
  }

  if (!b && btnPrev) {
    btnPrev = false;

    if (millis() - btnTime > 1500) {
      night = !night;
    } else {
      pedReq = true;
    }
  }

  emerg = !digitalRead(A0);
}

void setup() {
  Serial.begin(9600);

  pinMode(13, OUTPUT);
  pinMode(12, OUTPUT);
  pinMode(11, OUTPUT);
  pinMode(2, INPUT_PULLUP);
  pinMode(A0, INPUT_PULLUP);

  goTo(S_GREEN, 10000);
}

void loop() {
  readInputs();

  if (emerg && st != S_WARNING) {
    goTo(S_WARNING, 500);
  }

  if (!emerg && !night && st == S_WARNING) {
    goTo(S_GREEN, 10000);
  }

  if (night && st != S_WARNING) {
    goTo(S_WARNING, 500);
  }

  unsigned long e = millis() - tStart;

  switch (st) {

    case S_GREEN:
      if (e >= dur) {
        goTo(S_YELLOW, 3000);
      }
      break;

    case S_YELLOW:
      if (e >= dur) {
        if (pedReq) {
          pedReq = false;
          goTo(S_RED, 15000);
        } else {
          goTo(S_RED, 10000);
        }
      }
      break;

    case S_RED:
      if (e >= dur) {
        goTo(S_GREEN, 10000);
      }
      break;

    case S_WARNING:
      if (millis() - lastBlink >= 500) {
        lastBlink = millis();
        blinkOn = !blinkOn;

        digitalWrite(13, 0);
        digitalWrite(12, blinkOn);
        digitalWrite(11, 0);
      }
      break;
  }
}
