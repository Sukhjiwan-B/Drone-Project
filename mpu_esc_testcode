// libraries
#include <Wire.h>
#include <Adafruit_MPU6050.h>
#include <Adafruit_Sensor.h>
#include <Servo.h>
#include <IRremote.hpp>

/*
 * Quadcopter PD Stabilization Test
 *
 * Purpose:
 * Test roll and pitch stabilization using an MPU-6050
 * and a Proportional-Derivative (PD) controller.
 *
 * Features:
 * - Gyroscope calibration
 * - Complementary filter for orientation estimation
 * - PD feedback control
 * - Four-motor ESC mixing
 * - IR software kill switch
 *
 * This is an experimental stabilization controller.
 * Altitude sensing and the autonomous flight state
 * machine are implemented separately.
 *
 * Derivative term and its implmentation still needs to be tested with the serial moniter first to confirm
 * the term is adding to the equations correctly DO NOT test with propellor yet, confirm that code
 */
const int IR_RECEIVE_PIN = 2;

// Red power button command
const uint8_t KILL_BUTTON = 69;

// Once true, motors stay killed until reset
bool killed = false;

Servo esc1;
Servo esc2;
Servo esc3;
Servo esc4;

// Choose actual Arduino pins
const int esc1Pin = 3;
const int esc2Pin = 5;
const int esc3Pin = 6;
const int esc4Pin = 7;


Adafruit_MPU6050 mpu;

// Gyroscope calibration offsets
float gyroOffsetX = 0.0;
float gyroOffsetY = 0.0;
float rolld = 0.0;
float pitchd = 0.0; 
float pitchoff = 0.0;
float rolloff = 0.0; 

// Time of previous MPU reading
unsigned long previousMPUTime = 0;

// Serial print timing
unsigned long now = 0;
unsigned long lastPrintTime = 0;
const unsigned long printInterval = 500; // print twice per second

// Attitude variables
float roll = 0.0;
float pitch = 0.0;

float rollerror = 0.0;
float pitcherror = 0.0;

float rollcorrection = 0.0;
float pitchcorrection = 0.0;

// Motor command values

float esc1command = 0.0;
float esc2command = 0.0;
float esc3command = 0.0;
float esc4command = 0.0;


// Throttle values
const float basethrottle = 1000; //OFF

// For the mixer test, 1500 makes it easy to see
// commands increase AND decrease around the midpoint.
const float baseflightthrottle = 1325; //min PWM signal needed to produce lift 

const float minthrottle = 1075; //min PWM needed for motors to spin 
const float maxthrottle = 1500; //1500 abritrary maximum set for now

// Desired orientation
const float targetPitch = 0.0;
const float targetRoll = 0.0;

// P gains
const float kproll = 0.6; //values are being tested still
const float kppitch = 0.6; // values are being tested still
const float kpyaw = 1.0;// not yet intigrated, value is just a place holder 

const float kdroll = 1.0; //values are being tested still, also placeholder kd of 1 is probably too high 
const float kdpitch = 1.0;//values are being tested still
const float kdyaw = 1.0;// not yet intigrated, value is just a place holder 

const float kiroll = 1.0; //  not yet intigrated, value is just a place holder 
const float kipitch = 1.0; // not yet intigrated, value is just a place holder 
const float kiyaw = 1.0;//  not yet intigrated, value is just a place holder 


void setup() {

  Serial.begin(115200);

  // Start IR receiver
  IrReceiver.begin(
    IR_RECEIVE_PIN,
    ENABLE_LED_FEEDBACK
  );

  // Attach ESCs
  esc1.attach(esc1Pin);
  esc2.attach(esc2Pin);
  esc3.attach(esc3Pin);
  esc4.attach(esc4Pin);

  // Send minimum throttle so ESCs can arm
  esc1.writeMicroseconds((int)basethrottle);
  esc2.writeMicroseconds((int)basethrottle);
  esc3.writeMicroseconds((int)basethrottle);
  esc4.writeMicroseconds((int)basethrottle);
  

  delay(3000);

   // Initialize MPU
  initializeMPU();
  calibrateGyroscope();
  initializeAngles();

  //useful only if drone starts off level, will probably be taken out once mpu placement is screwed into the frame
  rolloff = roll; 
  pitchoff = pitch; 

  previousMPUTime = micros();
  
  Serial.println("Mixer test ready.");
  Serial.println("REMOVE PROPELLERS.");
  Serial.println();
}


void loop() {

  if (IrReceiver.decode()) {

    uint8_t command =
      IrReceiver.decodedIRData.command;


    Serial.print("IR command: ");
    Serial.println(command);


    if (command == KILL_BUTTON) {

      killed = true;

      Serial.println();
      Serial.println("KILL SWITCH ACTIVATED!");
      Serial.println("Motors locked at 1000 us.");
      Serial.println();

    }


    IrReceiver.resume();

  }


  
  //software killswitch, set motors to basethrottle(off) continuosly so that program has ot be restarted for the motors to start spinning again
  if (killed) {

    esc1.writeMicroseconds((int)basethrottle);
    esc2.writeMicroseconds((int)basethrottle);
    esc3.writeMicroseconds((int)basethrottle);
    esc4.writeMicroseconds((int)basethrottle);

    return;
  }


  now = millis();

  // Read MPU
  readmpu();

  // Calculate roll/pitch corrections
  Rollcorrection();
  Pitchcorrection();

  // Calculate motor commands
  escscommand();

  // for mpu testing through the serial moniter, actively tilt mpu, to see if the mpu is working as intended check corresponding readings on moniter 
  if (now - lastPrintTime >= printInterval) {

    lastPrintTime = now;

    Serial.println();
    Serial.println("========== DRONE STATUS ==========");

    // MPU-6050 orientation measurements
    Serial.println("[ORIENTATION]");

    Serial.print("Roll:         ");
    Serial.print(roll);
    Serial.println(" deg");

    Serial.print("Roll Rate:    ");
    Serial.print(rolld);
    Serial.println(" deg/s");

    Serial.print("Pitch:        ");
    Serial.print(pitch);
    Serial.println(" deg");

    Serial.print("Pitch Rate:   ");
    Serial.print(pitchd);
    Serial.println(" deg/s");

    // PD controller corrections
    Serial.println();
    Serial.println("[PD CORRECTIONS]");

    Serial.print("Roll:         ");
    Serial.println(rollcorrection);

    Serial.print("Pitch:        ");
    Serial.println(pitchcorrection);

    // Individual ESC commands
    Serial.println();
    Serial.println("[MOTOR COMMANDS]");

    Serial.print("Motor 1:      ");
    Serial.print(esc1command);
    Serial.println(" us");

    Serial.print("Motor 2:      ");
    Serial.print(esc2command);
    Serial.println(" us");

    Serial.print("Motor 3:      ");
    Serial.print(esc3command);
    Serial.println(" us");

    Serial.print("Motor 4:      ");
    Serial.print(esc4command);
    Serial.println(" us");

    Serial.println("==================================");
  }
}
// Read MPU data
void readmpu() {

  //structures that will hold different readings predefined by Adafruit Library 
  sensors_event_t acceleration;
  sensors_event_t rotation;
  sensors_event_t temperature;

  //put the readings into the structures 
  mpu.getEvent(
    &acceleration,
    &rotation,
    &temperature
  );

  // Time between MPU readings
  unsigned long currentMPUTime = micros();

  float deltaTime =
      (currentMPUTime - previousMPUTime) / 1000000.0;

  previousMPUTime = currentMPUTime;

  // Ignore bad timing intervals
  if (deltaTime <= 0.0 || deltaTime > 0.1) {
    return;
  }

  // look in the mpu readings, look specifically for accleration and apply x,y,z measuremtns to their respective variables 
  float ax = acceleration.acceleration.x;
  float ay = acceleration.acceleration.y;
  float az = acceleration.acceleration.z;

  // Roll/pitch estimates from gravity ( accelermoter reading)
  float accelRoll =
      atan2(ay, az) * 180.0 / PI;

  float accelPitch =
      atan2(
        -ax,
        sqrt(ay * ay + az * az)
      ) * 180.0 / PI;

  // Gyroscope readings
  float gyroRateX =
      (rotation.gyro.x - gyroOffsetX)
      * 180.0 / PI;

  float gyroRateY =
      (rotation.gyro.y - gyroOffsetY)
      * 180.0 / PI;

  // Complementary filter
  /* the gyroscope readings are more accurate for immidiate corrections so the overall weighted reading supplied to the controller will mostly come from that
     but the acceleromoter reading can help stabilize position over the longterm once the gyroscope readings die down and the drone is just trying to hover
     so make a small part of the reading supplied from the accel too 
  */
  roll =
      0.98 * (roll + gyroRateX * deltaTime)
      + 0.02 * accelRoll;

  pitch =
      0.98 * (pitch + gyroRateY * deltaTime)
      + 0.02 * accelPitch;

  //derivative terms 
  rolld = gyroRateX; 

  pitchd = gyroRateY; 
}

//mentioned earlier but pitchoff and rolloff are temporary variables for test will probably be discarded later 
// Pitch correction
void Pitchcorrection() {

  pitcherror = targetPitch - (pitch - pitchoff);

  pitchcorrection =
      kppitch * pitcherror - kdpitch*pitchd;
}


// Roll correction
void Rollcorrection() {

  rollerror = targetRoll - (roll - rolloff);

  rollcorrection =
      kproll * rollerror - kdroll*rolld;
}


// Calculate motor commands
void escscommand() {
  //the PWM signals sent to the motors are a combination of minimum signal needed for thrust and the correections 
  //top left and bottom right have same rotation and vice versa for the top right and bottom left 
  esc1command =
      baseflightthrottle
      + rollcorrection
      + pitchcorrection;

  esc2command =
      baseflightthrottle
      + rollcorrection
      - pitchcorrection;

  esc3command =
      baseflightthrottle
      - rollcorrection
      + pitchcorrection;

  esc4command =
      baseflightthrottle
      - rollcorrection
      - pitchcorrection;

  //limit the commands mainly just so the motors never receive a signal below the minimum needed to spin the motor, i doubt max constraint will come into play
  //but safe to have just incase
  esc1command =
      constrain(
        esc1command,
        minthrottle,
        maxthrottle
      );

  esc2command =
      constrain(
        esc2command,
        minthrottle,
        maxthrottle
      );

  esc3command =
      constrain(
        esc3command,
        minthrottle,
        maxthrottle
      );

  esc4command =
      constrain(
        esc4command,
        minthrottle,
        maxthrottle
      );

  
  //actually send the PWM signals 
  esc1.writeMicroseconds((int)esc1command);
  esc2.writeMicroseconds((int)esc2command);
  esc3.writeMicroseconds((int)esc3command);
  esc4.writeMicroseconds((int)esc4command);
  
}


// Initialize MPU
void initializeMPU() {
  //check if the mpu is connected properly 
  if (!mpu.begin()) {

    Serial.println("MPU-6050 not detected!");

    while (true) {
      delay(10);
    }
  }

  Serial.println("MPU-6050 connected.");

  mpu.setAccelerometerRange(
    MPU6050_RANGE_8_G
  );

  mpu.setGyroRange(
    MPU6050_RANGE_500_DEG
  );

  mpu.setFilterBandwidth(
    MPU6050_BAND_21_HZ
  );
}


// Gyroscope calibration
void calibrateGyroscope() {

  const int samples = 1000;

  float totalX = 0.0;
  float totalY = 0.0;

  Serial.println(
    "Keep the MPU completely still..."
  );

  delay(1000);

  for (int i = 0; i < samples; i++) {

    sensors_event_t acceleration;
    sensors_event_t rotation;
    sensors_event_t temperature;

    mpu.getEvent(
      &acceleration,
      &rotation,
      &temperature
    );

    totalX += rotation.gyro.x;
    totalY += rotation.gyro.y;

    delay(2);
  }

  gyroOffsetX =
      totalX / samples;

  gyroOffsetY =
      totalY / samples;

  Serial.println(
    "Gyroscope calibrated."
  );
}


// Initialize starting roll and pitch
void initializeAngles() {

  sensors_event_t acceleration;
  sensors_event_t rotation;
  sensors_event_t temperature;

  mpu.getEvent(
    &acceleration,
    &rotation,
    &temperature
  );

  float ax = acceleration.acceleration.x;
  float ay = acceleration.acceleration.y;
  float az = acceleration.acceleration.z;

  roll =
      atan2(ay, az)
      * 180.0 / PI;

  pitch =
      atan2(
        -ax,
        sqrt(ay * ay + az * az)
      ) * 180.0 / PI;

  Serial.print("Initial roll: ");
  Serial.println(roll);

  Serial.print("Initial pitch: ");
  Serial.println(pitch);
}
