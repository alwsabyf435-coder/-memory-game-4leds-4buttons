
# -memory-game-4leds-4buttons
• لعبة التذكر بأربعة ليدات وبازر وأربعة أزرار
// لعبة سايمون - ESP32
// توصيل نهائي حسب طلبك
// الأزرار: أحمر 4 - أزرق 5 - أخضر 18 - أصفر 19
// الليدات: أحمر 13 - أزرق 12 - أخضر 14 - أصفر 27
// البازر: 25

const int ledPins[4] = {13, 12, 14, 27}; // أحمر, أزرق, أخضر, أصفر
const int btnPins[4] = {4, 5, 18, 19}; // أحمر, أزرق, أخضر, أصفر
const int buzzerPin = 25;

const int tones[4] = {262, 330, 392, 523}; // Do, Mi, Sol, Do عالي

int sequence[100];
int level = 0;
bool gameStarted = false;

void setup() {
  for (int i = 0; i < 4; i++) {
    pinMode(ledPins[i], OUTPUT);
    pinMode(btnPins[i], INPUT_PULLUP); // مهم جدا - بدون مقاومة خارجية
    digitalWrite(ledPins[i], LOW);
  }
  pinMode(buzzerPin, OUTPUT);
  randomSeed(analogRead(0));
  // ومضة بداية
  for(int i=0;i<4;i++){ digitalWrite(ledPins[i], HIGH); delay(100); digitalWrite(ledPins[i], LOW); }
}

void loop() {
  if (!gameStarted) {
    // انتظر أي زر ليبدأ
    for (int i = 0; i < 4; i++) {
      if (digitalRead(btnPins[i]) == LOW) {
        delay(200);
        gameStarted = true;
        level = 0;
        nextLevel();
        break;
      }
    }
  } else {
    // دور اللاعب
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
  // عرض التسلسل
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
        while (digitalRead(btnPins[i]) == LOW); // انتظر رفع اليد
        delay(50); // debouncing
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
  // صوت خطأ
  for(int i=0;i<3;i++){
    for(int j=0;j<4;j++) digitalWrite(ledPins[j], HIGH);
    tone(buzzerPin, 100, 200);
    delay(250);
    for(int j=0;j<4;j++) digitalWrite(ledPins[j], LOW);
    delay(150);
  }
  gameStarted = false;
  level = 0;
}
