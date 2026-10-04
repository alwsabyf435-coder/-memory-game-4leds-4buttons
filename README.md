// لعبة سايمون - مع زر بداية منفصل
// الأزرار الملونة: أحمر 4 - أزرق 5 - أخضر 18 - أصفر 19
// زر البداية: 21
// الليدات: أحمر 13 - أزرق 12 - أخضر 14 - أصفر 27
// البازر: 25

const int ledPins[4] = {13, 12, 14, 27};
const int btnPins[4] = {4, 5, 18, 19};
const int startBtn = 21; // زر البداية وإعادة اللعبة
const int buzzerPin = 25;
const int tones[4] = {262, 330, 392, 523};

int sequence[100];
int level = 0;
bool gameStarted = false;

void setup() {
  for (int i = 0; i < 4; i++) {
    pinMode(ledPins[i], OUTPUT);
    pinMode(btnPins[i], INPUT_PULLUP);
    digitalWrite(ledPins[i], LOW);
  }
  pinMode(startBtn, INPUT_PULLUP);
  pinMode(buzzerPin, OUTPUT);
  randomSeed(analogRead(0));

  // ترحيب عند التشغيل
  for(int i=0;i<4;i++){ playColor(i, 150); delay(100); }
}

void loop() {
  if (!gameStarted) {
    // انتظر زر البداية فقط
    if (digitalRead(startBtn) == LOW) {
      delay(200);
      while(digitalRead(startBtn)==LOW); // انتظر رفع اليد
      gameStarted = true;
      level = 0;
      tone(buzzerPin, 600, 200); delay(300);
      nextLevel();
    }
  } else {
    for (int i = 0; i <= level; i++) {
      int pressed = waitForPress();
      if (pressed!= sequence[i]) {
        gameOver();
        return;
      }
    }
    delay(500);
    level++;
    nextLevel();
  }
}

void nextLevel() {
  sequence[level] = random(0, 4);
  delay(300);
  for (int i = 0; i <= level; i++) {
    playColor(sequence[i], 400);
    delay(200);
  }
}

int waitForPress() {
  while (true) {
    for (int i = 0; i < 4; i++) {
      if (digitalRead(btnPins[i]) == LOW) {
        playColor(i, 300);
        while (digitalRead(btnPins[i]) == LOW);
        delay(50);
        return i;
      }
    }
  }
}

void playColor(int color, int duration) {
  digitalWrite(ledPins[color], HIGH);
  tone(buzzerPin, tones[color], duration);
  delay(duration);
  digitalWrite(ledPins[color], LOW);
  noTone(buzzerPin);
}

void gameOver() {
  // صوت خسارة
  for(int i=0;i<3;i++){
    for(int j=0;j<4;j++) digitalWrite(ledPins[j], HIGH);
    tone(buzzerPin, 100, 300);
    delay(300);
    for(int j=0;j<4;j++) digitalWrite(ledPins[j], LOW);
    delay(150);
  }
  // الآن يرجع وضع الانتظار - لازم تضغط زر البداية (21) عشان تعيد
  gameStarted = false;
  level = 0;
  // ومضة تنبيه انه ينتظر زر البداية
  for(int i=0;i<2;i++){ digitalWrite(ledPins[0], HIGH); delay(200); digitalWrite(ledPins[0], LOW); delay(200); }
}