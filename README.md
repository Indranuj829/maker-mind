# maker-mind
// Self-Voting Machine (Useless Voting System) - Final Improved Version

const int buttonA = 2;
const int buttonB = 3;
const int buttonC = 4;

const int ledA = 6;
const int ledB = 7;
const int ledC = 8;
const int ledWinner = 9;
const int buzzer = 10;

int voteCountA = 0;
int voteCountB = 0;
int voteCountC = 0;

bool systemActive = true;
bool awaitingRestart = false;
unsigned long lastResetTime = 0;
const unsigned long resetInterval = 10000; // 10 seconds

int buttonPressCount = 0;
bool wasPressedAll = false;

void setup() {
  pinMode(buttonA, INPUT_PULLUP);
  pinMode(buttonB, INPUT_PULLUP);
  pinMode(buttonC, INPUT_PULLUP);

  pinMode(ledA, OUTPUT);
  pinMode(ledB, OUTPUT);
  pinMode(ledC, OUTPUT);
  pinMode(ledWinner, OUTPUT);
  pinMode(buzzer, OUTPUT);

  Serial.begin(9600);
  randomSeed(analogRead(0));

  delay(5000);
  Serial.println("🗳️ Self-Voting Machine Initialized!");
  delay(5000);
}

void loop() {
  if (!systemActive) {
    checkRestartCondition();
    return;
  }

  bool buttonAState = digitalRead(buttonA) == LOW;
  bool buttonBState = digitalRead(buttonB) == LOW;
  bool buttonCState = digitalRead(buttonC) == LOW;

  if (buttonAState || buttonBState || buttonCState) {
    delay(5000); // 5 seconds voting delay

    if (random(0, 100) < 50) {
      if (buttonAState) {
        voteCountA++;
        Serial.println("🧑 Your vote counted for A!");
      } else if (buttonBState) {
        voteCountB++;
        Serial.println("🧑 Your vote counted for B!");
      } else if (buttonCState) {
        voteCountC++;
        Serial.println("🧑 Your vote counted for C!");
      }
    } else {
      int randomVote = random(0, 3);
      if (randomVote == 0) {
        voteCountA++;
        Serial.println("🤖 System randomly voted for A.");
      } else if (randomVote == 1) {
        voteCountB++;
        Serial.println("🤖 System randomly voted for B.");
      } else {
        voteCountC++;
        Serial.println("🤖 System randomly voted for C.");
      }
    }

    delay(5000);
    showResults();
    delay(10000); // 10 second reset time
    resetVoting();
  }

  checkStopCondition();
}

void showResults() {
  Serial.println("📊 Round Results:");
  delay(5000);
  Serial.print("🔴 A: "); Serial.println(voteCountA);
  delay(5000);
  Serial.print("🟢 B: "); Serial.println(voteCountB);
  delay(5000);
  Serial.print("🔵 C: "); Serial.println(voteCountC);
  delay(5000);

  int maxVotes = max(voteCountA, max(voteCountB, voteCountC));

  if (voteCountA == maxVotes) {
    Serial.println("🏆 Winner: A!");
    digitalWrite(ledA, HIGH);
  } else if (voteCountB == maxVotes) {
    Serial.println("🏆 Winner: B!");
    digitalWrite(ledB, HIGH);
  } else {
    Serial.println("🏆 Winner: C!");
    digitalWrite(ledC, HIGH);
  }

  blinkWinnerLED();
  playBeep();
}

void blinkWinnerLED() {
  for (int i = 0; i < 6; i++) {
    digitalWrite(ledWinner, HIGH);
    delay(300);
    digitalWrite(ledWinner, LOW);
    delay(300);
  }
}

void playBeep() {
  tone(buzzer, 1000);
  delay(500);
  noTone(buzzer);
}

void resetVoting() {
  voteCountA = 0;
  voteCountB = 0;
  voteCountC = 0;

  digitalWrite(ledA, LOW);
  digitalWrite(ledB, LOW);
  digitalWrite(ledC, LOW);
  digitalWrite(ledWinner, LOW);

  Serial.println("🔄 System reset for next round!");
  delay(5000);
}

void checkStopCondition() {
  static int consecutivePresses = 0;
  static unsigned long lastPressTime = 0;

  if (digitalRead(buttonA) == LOW && digitalRead(buttonB) == LOW && digitalRead(buttonC) == LOW) {
    if (millis() - lastPressTime > 500) {
      consecutivePresses++;
      lastPressTime = millis();
    }
  }

  if (consecutivePresses >= 2) {
    systemActive = false;
    Serial.println("🛑 System stopped! Press all buttons once to restart.");
    delay(5000);
    consecutivePresses = 0;
  }
}

void checkRestartCondition() {
  if (digitalRead(buttonA) == LOW && digitalRead(buttonB) == LOW && digitalRead(buttonC) == LOW) {
    systemActive = true;
    Serial.println("✅ System restarted! Ready to vote again.");
    delay(5000);
  }
}
