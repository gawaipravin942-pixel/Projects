// ===== Motor Pins =====
#define PWM1 25
#define DIR1 26
#define PWM2 27
#define DIR2 14

// ===== Receiver Pins =====
#define THROTTLE_PIN 34
#define STEERING_PIN 35

const int freq = 1000;
const int resolution = 8;

void setup() {

  Serial.begin(115200);

  // Set outputs immediately LOW
  pinMode(PWM1, OUTPUT);
  pinMode(PWM2, OUTPUT);
  pinMode(DIR1, OUTPUT);
  pinMode(DIR2, OUTPUT);

  digitalWrite(PWM1, LOW);
  digitalWrite(PWM2, LOW);
  digitalWrite(DIR1, LOW);
  digitalWrite(DIR2, LOW);

  delay(200);

  // Attach PWM (ESP32 Core 3.x)
  ledcAttach(PWM1, freq, resolution);
  ledcAttach(PWM2, freq, resolution);

  ledcWrite(PWM1, 0);
  ledcWrite(PWM2, 0);

  pinMode(THROTTLE_PIN, INPUT);
  pinMode(STEERING_PIN, INPUT);
}

void loop() {

  int throttlePulse = pulseIn(THROTTLE_PIN, HIGH, 25000);
  int steeringPulse = pulseIn(STEERING_PIN, HIGH, 25000);

  if (!validSignal(throttlePulse) || !validSignal(steeringPulse)) {
    stopMotors();
    return;
  }

  int throttle = convertPulse(throttlePulse);
  int steering = convertPulse(steeringPulse);

  // Differential mixing (tank steering)
  int leftMotor  = throttle + steering;
  int rightMotor = throttle - steering;

  leftMotor  = constrain(leftMotor,  -255, 255);
  rightMotor = constrain(rightMotor, -255, 255);

  driveMotor(PWM1, DIR1, leftMotor);
  driveMotor(PWM2, DIR2, rightMotor);

  delay(10);
}

bool validSignal(int pulse) {
  return (pulse >= 1000 && pulse <= 2000);
}

int convertPulse(int pulse) {
  if (pulse > 1470 && pulse < 1530) return 0;   // Deadband
  return map(pulse, 1000, 2000, -255, 255);
}

void driveMotor(int pwmPin, int dirPin, int value) {

  if (value == 0) {
    ledcWrite(pwmPin, 0);
    return;
  }

  digitalWrite(dirPin, value > 0);
  ledcWrite(pwmPin, abs(value));
}

void stopMotors() {
  ledcWrite(PWM1, 0);
  ledcWrite(PWM2, 0);
}
