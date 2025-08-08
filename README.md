
// --- Arduino Voting System with User & System Voting, Tiebreak, and Auto Reset ---

const int buttonA = 2;
const int buttonB = 3;
const int buttonC = 4;
const int ledA = 5;
const int ledB = 6;
const int ledC = 7;
const int winnerLED = 8;
const int buzzer = 9;

int voteA = 0, voteB = 0, voteC = 0;
int prevA = HIGH, prevB = HIGH, prevC = HIGH;
int userVotes = 0;
bool systemActive = true;
bool tiebreak = false;

unsigned long lastResetTime = 0;
const unsigned long resetDelay = 20000; // 20 seconds in milliseconds
const unsigned long printDelay = 5000;  // 5 seconds between messages

unsigned long lastPrintTime = 0;
int printStep = 0;

void setup() {
  pinMode(buttonA, INPUT_PULLUP);
  pinMode(buttonB, INPUT_PULLUP);
  pinMode(buttonC, INPUT_PULLUP);
  pinMode(ledA, OUTPUT);
  pinMode(ledB, OUTPUT);
  pinMode(ledC, OUTPUT);
  pinMode(winnerLED, OUTPUT);
  pinMode(buzzer, OUTPUT);
  Serial.begin(9600);
  delay(1000);
  Serial.println("🟢 Voting System Initialized.");
  delay(1000);
  lastResetTime = millis();
}

void loop() {
  if (!systemActive) {
    checkRestart();
    return;
  }

  if (millis() - lastPrintTime >= printDelay) {
    switch (printStep) {
      case 0:
        Serial.println("🔄 New Voting Round Started...");
        break;
      case 1:
        Serial.println("🧑 Your turn to vote (Max 3 votes)...");
        break;
      case 2:
        Serial.println("🎲 Deciding whether to count your votes or system votes...");
        break;
      default:
        break;
    }
    printStep++;
    lastPrintTime = millis();
    if (printStep > 2) {
      printStep = 0;
      runVotingRound();
    }
  }
}

void runVotingRound() {
  voteA = voteB = voteC = 0;
  userVotes = 0;

  unsigned long voteStart = millis();
  while (millis() - voteStart < 10000 && userVotes < 3) {  // 10 seconds to vote
    if (digitalRead(buttonA) == LOW && prevA == HIGH) {
      voteA++;
      userVotes++;
      digitalWrite(ledA, HIGH);
      delay(100);
      digitalWrite(ledA, LOW);
      prevA = LOW;
    }
    if (digitalRead(buttonB) == LOW && prevB == HIGH) {
      voteB++;
      userVotes++;
      digitalWrite(ledB, HIGH);
      delay(100);
      digitalWrite(ledB, LOW);
      prevB = LOW;
    }
    if (digitalRead(buttonC) == LOW && prevC == HIGH) {
      voteC++;
      userVotes++;
      digitalWrite(ledC, HIGH);
      delay(100);
      digitalWrite(ledC, LOW);
      prevC = LOW;
    }
    if (digitalRead(buttonA) == HIGH) prevA = HIGH;
    if (digitalRead(buttonB) == HIGH) prevB = HIGH;
    if (digitalRead(buttonC) == HIGH) prevC = HIGH;
  }

  if (random(100) < 50) {
    Serial.println("✅ User votes accepted!");
  } else {
    Serial.println("❌ User votes ignored. System will vote.");
    voteA = random(0, 4);
    voteB = random(0, 4);
    voteC = random(0, 4);
  }

  displayResults();
  checkWinnerOrTie();
}

void displayResults() {
  Serial.print("📊 Vote A: "); Serial.println(voteA);
  delay(printDelay);
  Serial.print("📊 Vote B: "); Serial.println(voteB);
  delay(printDelay);
  Serial.print("📊 Vote C: "); Serial.println(voteC);
  delay(printDelay);
}

void checkWinnerOrTie() {
  if (voteA == voteB || voteA == voteC || voteB == voteC) {
    Serial.println("🤝 It's a Tie! Starting Tiebreaker Round...");
    delay(printDelay);
    tiebreak = true;
    runVotingRound();  // Tiebreaker
    tiebreak = false;
    checkWinnerOrTie();
  } else {
    declareWinner();
    systemActive = false;
    lastResetTime = millis();
  }
}

void declareWinner() {
  int winner = max(max(voteA, voteB), voteC);
  Serial.println("🏁 Final Result:");
  delay(printDelay);

  if (voteA == winner) {
    Serial.println("🏆 Winner: A");
    blinkLED(ledA);
  } else if (voteB == winner) {
    Serial.println("🏆 Winner: B");
    blinkLED(ledB);
  } else if (voteC == winner) {
    Serial.println("🏆 Winner: C");
    blinkLED(ledC);
  }
}

void blinkLED(int ledPin) {
  for (int i = 0; i < 5; i++) {
    digitalWrite(ledPin, HIGH);
    digitalWrite(winnerLED, HIGH);
    tone(buzzer, 1000, 200);
    delay(300);
    digitalWrite(ledPin, LOW);
    digitalWrite(winnerLED, LOW);
    delay(300);
  }
  noTone(buzzer);
}

void checkRestart() {
  if (digitalRead(buttonA) == LOW && digitalRead(buttonB) == LOW && digitalRead(buttonC) == LOW) {
    if (millis() - lastResetTime >= resetDelay) {
      Serial.println("🔁 Restarting System...");
      delay(printDelay);
      systemActive = true;
      voteA = voteB = voteC = 0;
      printStep = 0;
      lastResetTime = millis();
    }
  }
}
