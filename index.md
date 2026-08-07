# Gestured Controlled Robot
My project, the Gesture Controlled Robot, is a robot that moves from the tilt of the controller. In addition to that, I've added 2 modifications. One is an obstacle avoidance module (doesn't let it crash), and rear lights (turn signals, brake lights).

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Jonathan T | McNeil High School | Mechanical Engineering | Incoming Sophomore

![Robot, Hand module](Robot_Photo.jpeg)
  
# Final Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/EGdpX2PJOz0?si=yWJao6vWz5vQ0G73" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

### Summary
For my final milestone, I worked on adding modifications. There are 2 modifications. I have an obstacle avoidance module and rear lights. The way these work is the obstacle avoidance module uses an ultrasonic sensor and 2 ir sensors. The ultrasonic sensor acts as the main sensor. It works by sending soundwaves and seeing how long it takes for the soundwaves to be received back. Then we have the ir sensors. These act as the emergency sensor in case the ultrasonic sensor fails. During this whole project, my biggest challenge has been connecting the HC-05 bluetooth modules. This is because there are various different things needed to be formatted in a certain order and messing one up can make the whole thing not work. However, I continued to work on it and eventually got it working. I have learned persistence and become familiar with circuits and Arduinos. In the future, I hope to learn to build my own robot from scratch (Making my own pieces). I hope to use the knowledge I have gained to implement this to robotics as a field and compete on my school's robotics team.

# Second Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/-oUvwIYvIeA?si=aX1vO9CuitIUHga1&amp;start=1" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

### Summary
In my second milestone, the main thing I have accomplished is getting my accelerometer wired in and connecting my HC05 Bluetooth Modules together. Both of these are important for different reasons. The accelerometer is important because it is what is detecting movement. This can be tilt, acceleration, or vibration. In this case, I am using tilt. When the accelerometer is tilted in any direction, is sends a signal to my robot. The way it sends this signal is because of the connected HC05 Bluetooth Modules. One thing in this project that surprised is how the debugging of the code seems complicated but is a lot easier then it looks. A challenge is faced in this milestone is getting the Bluetooth Modules to connect and communicate with each other. I had this problem because I had installed the wrong file onto my Arduino IDE. So all I had to do was install the correct file. Now, I need to make the Hand Module wireless, and add some modifications.

# First Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/kWR7I4uDrG8?si=KUp_xfkqyDoa2PkN" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

### Summary
The overall goal of my first milestone was to fully build the robot and make it operational. When I built the robot, I first attached the 4 motors to the chassis' frame. Then I connected all 4 motors to a motor driver and 1 wheel onto each of the motors. The motor driver was then connected to an Arduino Uno each powered by a 9 volt battery. Then I connected a HC05 to the Arduino Uno through a breadboard. As I continue to work on this project, this would allow me to connect the hand module with the robot itself. A challenge I faced was being able to supply a sufficient power source. When I was trying to power the motor driver, I noticed that I didn't have a way to connect the battery to the motor driver. Because of this, I had to receive a new tool. This allowed me to make a connection between the battery and the motor driver. Another challenge I faced was keeping my materials safe. When I was attaching the HC05 to the Arduino Uno through the breadboard, the wiring on my breadboard was incorrect. This resulted in my TX pin to go through the voltage divider rather then the RX pin which needed its voltage to be decreased. This ended up frying my RX pin and requiring me to get a new HC05 with a new RX pin. To complete my project, I plan to get the HC05s communicating to each other, get the accelerometer working, and connect all the pieces together so the robot can move along with he movement of the hand.

# Schematics 
### Robot Chassis
![Robot Chassis](Robot_bb.png)

### Hand Module
![Hand Module](Hand_Module_bb.png)

### Rear Lights
![Rear Lights](Rear_Lights_bb.png)

### Obstacle Avoidance Module
![Obstacle Avoidance Module](Avoidance_Module_bb.png)

# Code
### UNO CODE
```c++

#include <SoftwareSerial.h>

SoftwareSerial BT_Serial(2, 3);
// RX, TX
// ==========================
// L298N MOTOR DRIVER
// ==========================
#define enA 10
#define in1 9
#define in2 8
#define in3 7
#define in4 6
#define enB 5
// ==========================
// HC-SR04
// ==========================
#define TRIG 4
#define ECHO 11
// ==========================
// LIGHTS
// ==========================
#define BRAKE_LIGHT A1
#define LEFT_SIGNAL A2
#define RIGHT_SIGNAL A3
// 10 inches ≈ 26 cm
#define SAFE_DISTANCE 26
char command = 's';
int Speed = 180;
void setup() {
  Serial.begin(9600);
  BT_Serial.begin(38400);
  // Motor pins
  pinMode(enA, OUTPUT);
  pinMode(enB, OUTPUT);
  pinMode(in1, OUTPUT);
  pinMode(in2, OUTPUT);
  pinMode(in3, OUTPUT);
  pinMode(in4, OUTPUT);
  // Ultrasonic
  pinMode(TRIG, OUTPUT);
  pinMode(ECHO, INPUT);
  // Lights
  pinMode(BRAKE_LIGHT, OUTPUT);
  pinMode(LEFT_SIGNAL, OUTPUT);
  pinMode(RIGHT_SIGNAL, OUTPUT);
  stopMotor();
  updateLights('s');
  Serial.println("Robot Ready");
}
void loop() {
  // ==========================
  // BLUETOOTH INPUT
  // ==========================
  if(BT_Serial.available()) {
    char incoming = BT_Serial.read();
    if(incoming == 'f' ||
       incoming == 'b' ||
       incoming == 'l' ||
       incoming == 'r' ||
       incoming == 's') {
      command = incoming;
    }
    Serial.print("Command: ");
    Serial.println(command);
  }
  // ==========================
  // ULTRASONIC
  // ==========================
  int distance = getDistance();
  Serial.print("Distance: ");
  Serial.print(distance);
  Serial.println(" cm");
  // ==========================
  // OBSTACLE MODE
  // ONLY BACKWARD ALLOWED
  // ==========================
  if(distance <= SAFE_DISTANCE && distance > 0) {
    Serial.println("OBSTACLE - REVERSE ONLY");
    if(command == 'l') {
      turnLeft();
      updateLights('l');
    }
    else {
      stopMotor();
      updateLights('s');
    }
    setSpeed();
    delay(50);
    return;
  }
  // ==========================
  // NORMAL MODE
  // ==========================
  switch(command) {
    case 'f':
      forward();
      break;
    case 'b':
      backward();
      break;
    case 'l':
      turnLeft();
      break;
    case 'r':
      turnRight();
      break;
    default:
      stopMotor();
      break;
  }
  updateLights(command);
  setSpeed();
  delay(50);
}
// ==========================
// MOTOR FUNCTIONS
// ==========================
void forward() {
  digitalWrite(in1,HIGH);
  digitalWrite(in2,LOW);
  digitalWrite(in3,LOW);
  digitalWrite(in4,HIGH);
}
void backward() {
  digitalWrite(in1,LOW);
  digitalWrite(in2,HIGH);
  digitalWrite(in3,HIGH);
  digitalWrite(in4,LOW);
}
void turnLeft() {
  digitalWrite(in1,LOW);
  digitalWrite(in2,HIGH);
  digitalWrite(in3,LOW);
  digitalWrite(in4,HIGH);
}
void turnRight() {
  digitalWrite(in1,HIGH);
  digitalWrite(in2,LOW);
  digitalWrite(in3,HIGH);
  digitalWrite(in4,LOW);
}
void stopMotor() {
  digitalWrite(in1,LOW);
  digitalWrite(in2,LOW);
  digitalWrite(in3,LOW);
  digitalWrite(in4,LOW);
}
// ==========================
// LIGHT CONTROL
// ==========================
void updateLights(char movement) {
  digitalWrite(BRAKE_LIGHT, LOW);
  digitalWrite(LEFT_SIGNAL, LOW);
  digitalWrite(RIGHT_SIGNAL, LOW);
  if(movement == 'l') {
    digitalWrite(BRAKE_LIGHT, HIGH);
  }
  else if(movement == 'f') {
    digitalWrite(LEFT_SIGNAL, HIGH);
  }
  else if(movement == 'b') {
    digitalWrite(RIGHT_SIGNAL, HIGH);
  }
}
void setSpeed() {
  analogWrite(enA, Speed);
  analogWrite(enB, Speed);
}
// ==========================
// HC-SR04 DISTANCE
// ==========================
int getDistance() {
  digitalWrite(TRIG,LOW);
  delayMicroseconds(5);
  digitalWrite(TRIG,HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG,LOW);
  long duration = pulseIn(ECHO,HIGH,30000);
  if(duration == 0) {
    return 999;
  }
  return duration / 58;
}
```
### Nano Code
```c++

#include <Arduino_BMI270_BMM150.h>
float x, y, z;
char lastCommand = 's';
void setup() {
  Serial.begin(9600);
  // HC-05
  Serial1.begin(38400);
  if (!IMU.begin()) {
    Serial.println("IMU failed!");
    while(1);
  }
  Serial.println("Gesture Ready");
}
void loop() {
  if(IMU.accelerationAvailable()) {
    IMU.readAcceleration(x, y, z);
    char command = 's'
    // Tilt forward
    if(x < -0.6) {
      command = 'f';
    }
    // Tilt backward
    else if(x > 0.6) {
      command = 'b';
    }
    // Tilt left
    else if(y > 0.6) {
      command = 'l';
    }
    // Tilt right
    else if(y < -0.6) {
      command = 'r';
    }
    // Only send when command changes
    if(command != lastCommand) {
      Serial1.write(command);
      Serial.print("Sent: ");
      Serial.println(command);
      lastCommand = command;
    }
  }
  delay(100);
}
```
# Bill of Materials

| **Part** | **Note** | **Price** | **Link** |
| :--- | :--- | :---: | :---: |
| Car Chassis Kit | Main Frame of Robot | $39.99 | [Link](https://amazon.com) |
| Screwdriver Kit | Tools Kit | $5.94 | [Link](https://amazon.com) |
| Arduino Uno Clone | Reads Inputs and Controls Outputs | $14.98 | [Link](https://amazon.com) |
| Electronics Kit | Electronic Parts | $14.00 | [Link](https://amazon.com) |
| Breadboard Kit | Breadboards to Connect Parts | $8.79 | [Link](https://amazon.com) |
| Arduino Nano 33 BLE Sense | Logic Control for Hand | $39.70 | [Link](https://amazon.com) |
| Micro USB Cable | Upload Code | $5.00 | [Link](https://amazon.com) |
| Accelerometer | Detects Acceleration | $9.00 | [Link](https://amazon.com) |
| HC05 | Connect Hand and Robot | $9.00 | [Link](https://amazon.com) |
| Breadboard Power Supply | Lets Breadboard Connect to Battery | $8.00 | [Link](https://amazon.com) |
| 9V Batteries | Power Components | $8.69 | [Link](https://amazon.com) |
| Velcro Tape | Connect Parts | $8.00 | [Link](https://amazon.com) |
| DMM | Debugging Tool | $9.99 | [Link](https://amazon.com) |
| Ultrasonic Sensor | Calculates Distance | $5.25 | [Link](https://sparkfun.com) |
| IR Obstacle Avoidance Module | Light Sensor | $5.75 | [Link](https://shillehtek.com) |
# Other Resources/Examples

1. <a href= "https://www.hackster.io/embeddedlab786/hand-gesture-control-robot-via-bluetooth-94b13d">Building Instructions</a>
2. <a href= "https://docs.google.com/document/d/1EpnEPulXQwPDSK-nKLohqPjpeXNteP2G/edit">Pairing Bluetooth Modules</a>
3. <a href= "https://www.youtube.com/watch?v=BXXAcFOTnBo">Pairing Bluetooth Modules Video</a>
4. <a href= "https://docs.sunfounder.com/projects/3in1-kit-v2/en/latest/car_project/car_auto.html">IR and Ultrasonic Sensor Wiring</a>
