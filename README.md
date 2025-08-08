# maker-mind
const int buttonA = 2;
const int buttonB = 3;
const int buttonC = 4;

const int ledA = 6;
const int ledB = 7;
const int ledC = 8;
const int buzzer = 9;
const int winnerLED = 10;

int voteCountA = 0;
int voteCountB = 0;
int voteCountC = 0;

unsigned long lastVoteTime = 0;
const unsigned long voteDuration = 20000; // 20 seconds
const unsigned long voteDelay = 5000; // 5 seconds
const unsigned long serialDelay = 4000; // 4 seconds

bool votingEnabled = true;

void setup() {
  pinMode(buttonA, INPUT_PULLUP);
  pinMode(buttonB, INPUT_PULLUP);
  pinMode(buttonC, INPUT_PULLUP);

  pinMode(ledA, OUTPUT);
  pinMode(ledB, OUTPUT);
  pinMode(ledC, OUTPUT);
  pinMode(buzzer, OUTPUT);
  pinMode(winnerLED, OUTPUT);

  Serial.begin(9600);
  randomSeed(analogRead(0));
  lastVoteTime = millis();

  Serial.println("Voting system started...");
  delay(serialDelay);
}

void loop() {
  if (!votingEnabled) {
    if (checkAllButtonsPressedOnce()) {
      votingEnabled = true;
      Serial.println("System started again!");
      delay(serialDelay);
    }
    return;
  }

  if (checkAllButtonsPressedTwice()) {
    votingEnabled = false;
    Serial.println("System stopped!");
    delay(serialDelay);
    return;
  }

  if (millis() - lastVoteTime >= voteDuration) {
    announceResult();
    resetVotes();
    lastVoteTime = millis();
  }

  // Check for vote
  if (digitalRead(buttonA) == LOW || digitalRead(buttonB) == LOW || digitalRead(buttonC) == LOW) {
    delay(50); // debounce
    int winner = -1;

    if (random(0, 2) == 0) {
      // System vote
      winner = random(0, 3);
      Serial.println("System voted...");
    } else {
      // Only register if button is pressed
      if (digitalRead(buttonA) == LOW) {
        winner = 0;
        Serial.println("User voted for A");
      } else if (digitalRead(buttonB) == LOW) {
        winner = 1;
        Serial.println("User voted for B");
      } else if (digitalRead(buttonC) == LOW) {
        winner = 2;
        Serial.println("User voted for C");
      }
    }

    delay(serialDelay);

    // Count the vote
    if (winner == 0) voteCountA++;
    else if (winner == 1) voteCountB++;
    else if (winner == 2) voteCountC++;

    updateLEDs();
    delay(voteDelay);
  }
}

void updateLEDs() {
  digitalWrite(ledA, voteCountA > 0 ? HIGH : LOW);
  digitalWrite(ledB, voteCountB > 0 ? HIGH : LOW);
  digitalWrite(ledC, voteCountC > 0 ? HIGH : LOW);
}

void announceResult() {
  Serial.println("Final Result:");
  delay(serialDelay);
  Serial.print("A: "); Serial.println(voteCountA); delay(serialDelay);
  Serial.print("B: "); Serial.println(voteCountB); delay(serialDelay);
  Serial.print("C: "); Serial.println(voteCountC); delay(serialDelay);

  int maxVote = max(voteCountA, max(voteCountB, voteCountC));
  int winner = -1;

  if (voteCountA == maxVote) winner = 0;
  else if (voteCountB == maxVote) winner = 1;
  else winner = 2;

  Serial.print("Winner is: ");
  if (winner == 0) Serial.println("A");
  else if (winner == 1) Serial.println("B");
  else Serial.println("C");

  delay(serialDelay);
  beepBuzzer();
  blinkWinnerLED();
}

void beepBuzzer() {
  digitalWrite(buzzer, HIGH);
  delay(500);
  digitalWrite(buzzer, LOW);
}

void blinkWinnerLED() {
  for (int i = 0; i < 6; i++) {
    digitalWrite(winnerLED, HIGH);
    delay(300);
    digitalWrite(winnerLED, LOW);
    delay(300);
  }
}

void resetVotes() {
  voteCountA = 0;
  voteCountB = 0;
  voteCountC = 0;

  digitalWrite(ledA, LOW);
  digitalWrite(ledB, LOW);
  digitalWrite(ledC, LOW);
  Serial.println("Votes reset");
  delay(serialDelay);
}

bool checkAllButtonsPressedTwice() {
  static int pressCount[3] = {0, 0, 0};
  static unsigned long lastCheck = 0;

  if (millis() - lastCheck > 200) {
    if (digitalRead(buttonA) == LOW) pressCount[0]++;
    if (digitalRead(buttonB) == LOW) pressCount[1]++;
    if (digitalRead(buttonC) == LOW) pressCount[2]++;
    lastCheck = millis();
  }

  if (pressCount[0] >= 2 && pressCount[1] >= 2 && pressCount[2] >= 2) {
    pressCount[0] = 0;
    pressCount[1] = 0;
    pressCount[2] = 0;
    return true;
  }

  return false;
}

bool checkAllButtonsPressedOnce() {
  static int pressCount[3] = {0, 0, 0};
  static unsigned long lastCheck = 0;

  if (millis() - lastCheck > 200) {
    if (digitalRead(buttonA) == LOW) pressCount[0]++;
    if (digitalRead(buttonB) == LOW) pressCount[1]++;
    if (digitalRead(buttonC) == LOW) pressCount[2]++;
    lastCheck = millis();
  }

  if (pressCount[0] >= 1 && pressCount[1] >= 1 && pressCount[2] >= 1) {
    pressCount[0] = 0;
    pressCount[1] = 0;
    pressCount[2] = 0;
    return true;
  }

  return false;
}
