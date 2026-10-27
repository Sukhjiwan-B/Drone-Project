# Drone-Project
This is a project I've been building with 2 of my friends. Our current goal is autonomous flight, to get the drone to ascend to a certain height, hover for a set time and descend back to the ground. 



## Our Process (Brief Summary)

We began by researching the requirements for building an autonomous quadcopter and dividing responsibilities among our three team members. I focused on developing the flight controller software, while my friends researched the electronics and frame design. We then collaborated to finalize our component selection and the drone's dimensions.

Once we received our components, we began constructing the frame, soldering electrical connections, and developing the flight controller software. After assembling our initial prototype, we created smaller test programs to verify individual subsystems and gather data, including determining the PWM signals required to generate sufficient thrust for lift.

During this stage, we also consulted an engineering mentor who provided valuable advice on improving our design and testing procedures.

Through iterative testing, troubleshooting, and redesign, we progressed from our initial LEGO prototype to a stronger plywood frame. We are currently refining our second prototype and tuning its feedback controller as we work toward autonomous flight.


## Part Selection

### Electronic Components
- **Arduino Uno R3:** Main flight controller
- **MPU-6050 IMU:** Accelerometer and gyroscope for measuring orientation and angular velocity
- **Ultrasonic sensor:** Measures altitude above the ground
- **Power Distribution Board (PDB):** Distributes battery power to the four ESCs and provides regulated 5V power for the Arduino
- **4 Electronic Speed Controllers (ESCs):** Control the four brushless motors using signals from the Arduino
- **4 A2212 1000KV brushless motors:** Generate thrust through the propellers
- **4 propellers:** 10 × 4.5 inches
- **11.1V LiPo battery:** Main power source
- **IR receiver and remote:** Provide a software-controlled motor shutdown
- **Jumper wires and connectors:** Connect electrical components

### Frame Materials
- **Prototype 1:** LEGO frame
- **Prototype 2:** Plywood, screws, and bolts

### Tools
- Multimeter
- Soldering iron
- Hand tools

### Why we choose these parts and their uses

Rather than immediately designing a custom PCB, we chose a modular approach that would allow us to understand, test, and troubleshoot each subsystem of the drone independently, such as its power distribution system using the PDB and orientation and position measurements through the MPU and sonar.

We selected a Power Distribution Board (PDB) with sufficient connections for four ESCs and an integrated 5V regulator to power the Arduino Uno. This allowed us to simplify our power distribution system without requiring a separate voltage regulator.

The Arduino Uno serves as our flight controller, processing measurements from the MPU-6050 IMU and ultrasonic sensor to calculate motor control signals.

The MPU-6050 combines an accelerometer and gyroscope, providing measurements used to estimate the drone's orientation and angular velocity. The ultrasonic sensor measures the distance to the ground, which will be used to manage altitude during autonomous flight.

Four A2212 1000KV brushless motors generate thrust, with each motor controlled by an ESC receiving signals from the Arduino.
  
<img width="1170" height="1408" alt="parts" src="https://github.com/user-attachments/assets/b567ac19-3100-4677-8b7a-43665f514507" />


## Programming

### PD Feedback Controller

During my initial research, I learned that maintaining stable flight would require a feedback control system. I initially implemented a Proportional (P) controller before progressing to a Proportional-Derivative (PD) controller to improve stability and reduce oscillations.

The controller uses measurements from the MPU to estimate the drone's orientation and calculate corrections to the motor commands.

**Proportional Control (P)**

The proportional component calculates a correction based on the difference between the drone's current orientation and its desired orientation (level).

For example, if the drone tilts 15° to the left, the controller calculates a correction proportional to this error. The correction is scaled by a constant, Kp, which determines how aggressively the motors respond.

A Kp value that is too low may produce a slow response, while a value that is too high can cause overshooting and oscillations. We therefore need to tune this through testing.

**Derivative Control (D)**

An issue I've ran into while testing the flight controller is the drone will level itself out and then begin oscillating. My assumption right now is that only having a P controller is causing the correction to over shoot, creating this oscillation of the drone. The goal is that this D term will stop the drone from overshooting its corrections, or at least reduce how often that happens compared to just the P controller.&#x20;

For example, if the drone initially has a 15° tilt and its error decreases to 10° after 0.5 seconds, the rate of change of the error is -10°/s.

By multiplying this rate by a second constant, Kd, the controller can account for the drone's rotational motion. When the drone is already correcting toward level, the derivative component can reduce the overall correction, helping prevent overshoot and oscillations.

These corrections are then applied to the motor control signals, adjusting the thrust generated by the motors to counteract changes in orientation.

We are currently testing and tuning Kp and Kd to achieve a stable response along a single axis before progressing to full flight.

### State Machine&#x20;

In addition to the PD controller, I implemented a finite state machine to manage the drone's autonomous flight sequence.

It consists of five states:

1. **Ground:** Motors remain at their initial operating condition before takeoff.
2. **Rise:** The drone increases altitude toward its target height.
3. **Hover:** The drone attempts to maintain its target altitude for a specified duration.
4. **Descend:** The drone decreases altitude in preparation for landing.
5. **Land:** The drone completes its landing sequence and shuts down the motors.

The ultrasonic sensor measures the drone's height above the ground, allowing the controller to use altitude measurements to determine when certain state transitions should occur.

Our goal is for the state machine and feedback controller to work together to achieve autonomous takeoff, hovering, and landing once the control system has been properly tuned and tested.


**code:** 


## Hardware

### Power Distribution

Our drone uses a Power Distribution Board (PDB) to distribute power from an 11.1V LiPo battery to four Electronic Speed Controllers (ESCs). Each ESC supplies three-phase power to its respective brushless motor, controlling its speed based on signals received from the Arduino.

The PDB also includes a 5V voltage regulator, which allows us to connect the Arduino to the PDB safely with out damaging it. This allows us to power the control electronics separately from the higher-voltage motor supply as these electronics receive their power from the Arduino.

<img width="3024" height="4032" alt="hardware" src="https://github.com/user-attachments/assets/c19a61a3-3763-456a-83ad-85e2bc3cdf1a" />


### Microcontroller and Sensors

The Arduino serves as the main controller, connected with the MPU, sonar sensor, and infrared (IR) receiver.

The MPU-6050 provides acceleration and angular velocity measurements used for stabilization, while the ultrasonic sensor measures the drone's distance from the ground. The IR receiver allows us to send commands to the Arduino, including an emergency motor shutdown.

The Arduino also generates control signals for the four ESCs, allowing us to adjust motor speeds independently.

### Common Ground and Wiring

One important consideration was ensuring that all electronic components shared a **common ground**.

Although the ESCs receive power directly from the PDB, their control signals are from the Arduino. Connecting their grounds ensures that the ESCs and Arduino interpret these signals relative to the same reference voltage.

Initially, we used a breadboard to connect the grounds of our components. However, this took up too much space on the frame.

We replaced the breadboard connections with **WAGO lever connectors** (the orange pieces), allowing multiple ground wires to connect at a single point. This made wiring all the components easier as there wasn't one fixed spot to put all the connections. It did make things messier, but it allowed us to have more space on the frame to comfortably hold all our electronics.&#x20;


<img width="700" height="782" alt="hardware" src="https://github.com/user-attachments/assets/b1ff0da9-9211-4cbb-8e39-f613146da55e" />


## Frame design
We have had two prototypes for our drone. Our initial idea was a 3D printed PLA frame suggested and designed by my friend as mentioned in the first summary, however we did not have access to a 3D printer so we had to improvise with common materials we had. The first frame I made was with extra Legos; we had made measurements about how large the center body had to be to fit all the components, this came out to 15X15cm. We also determined how long the arms would have to be to stop our 10(diameter inches)x4.5(pitch inches) propellors from hitting each other, our first design had the motors (also the center placement of the propellors) 19cm away from the body. Which was a generous amount of extra space. This frame was built and went through multiple tests unitl it broke. Our second frame was more compact and had the motors only 14cm away from the body which reduced the torque experienced at the connection joint, this frame was also made of plywood and used screws and bolts to put everything together. This frame is much stronger and has held up to all tests. One other thing I've implemented is attaching the power distribution system to the underside of the drone to open up more space on top for the electronics, Arduino and battery. 


<img width="3024" height="4032" alt="mockframe" src="https://github.com/user-attachments/assets/afa2ef0c-5d59-4121-9c53-b60da0f52523" />
<img width="2956" height="3546" alt="droneframe" src="https://github.com/user-attachments/assets/b0a6fcee-5858-49e4-96c7-4b7cc19e4e01" />
<img width="1170" height="1199" alt="PDBSYSTEM" src="https://github.com/user-attachments/assets/ffdfdac0-8d11-4010-bab9-d41d8868507d" />

## Mentor Advice
Once we were ready for running some tests to gather data about the drone we first went to a mentor I had met through the UBC mentorship program to ask for advice on our drone. What he had strongly recommended to us was to design a kill switch as well just incase anything ever went wrong in testing, and to make a physical switch that can turn the drone on and off rather then having to remove the battery connection every time as it can be quite a hassle, and even damage components. We were glad we talked to him before testing, with propellors on, and grateful for his advice. As it did end up saving us a couple times. The kill switch was installed right away. It was a software kill switch that involved an IR receiver and remote I had lying around in an electronics kit. The IR receiver would constantly check if the remote power button had been clicked every loop, and once clicked would send the motors to their minimum throttle(off) right away, and no other code would run again until the program was restarted. Allowing us to safely handle the drone and disconnect power with out worrying about the propellors spinning. The power switch we are still designing and finding a way to incorporate. 


https://github.com/user-attachments/assets/c825721a-ed55-435a-8f7f-889e638d65b6



## Prototyping and Testing

Throughout development, we created several smaller programs to verify individual components and gradually test the drone's integrated systems. Our testing process included six stages:

1. MPU-6050 testing
2. Ultrasonic sensor and state machine testing
3. ESC and motor testing
4. Combined MPU, ESC, and motor testing
5. Throttle testing and frame redesign
6. PD controller tuning (ongoing)

### 1. MPU-6050 Testing

Our first test involved the Arduino, MPU-6050, and Arduino Serial Monitor.

I manually rotated the MPU-6050 while monitoring its orientation measurements through the Serial Monitor. More importantly, I verified that the program calculated the correct PWM correction signals that would eventually be sent to the ESCs.

To validate these calculations, I compared the PWM outputs displayed in the Serial Monitor against manually calculated values using the same sensor measurements.

This allowed us to verify the sensor readings and motor correction calculations before connecting the motors.

### 2. Ultrasonic Sensor and State Machine Testing

The second test involved the Arduino, ultrasonic sensor, and our drone's flight-state machine.

I manually adjusted the ultrasonic sensor's height above the ground while monitoring the current flight state displayed in the Serial Monitor.

The purpose was to verify that the drone transitioned between the correct flight states at the intended height thresholds.

This helped confirm that the height measurements and state transition logic were functioning as expected.

### 3. ESC and Motor Testing

After successfully testing the sensors and control logic, we began testing the four ESCs and brushless motors.

Initially, we sent constant PWM signals to all four ESCs to verify that the motors could spin smoothly and consistently.

We also gradually adjusted the PWM signals to determine the minimum PWM required for motor rotation, which we found to be approximately **1075 microseconds**.

Like our previous tests, this was conducted without propellers attached.

We also tested software kill-switch feature by pressing the remote's power button and verifying that all four motors stopped.


### 4. Combined MPU, ESC, and Motor Testing

Our final test without propellers involved integrating the MPU-6050, ESCs, and motors on the drone frame.

We ran the motors at approximately their minimum spinning throttle of 1075 microseconds and manually tilted the drone along different axes.

As the drone tilted, the P controller increased or decreased the PWM signals sent to individual motors based on the measured orientation.

Because the motors were initially operating near their minimum spinning throttle, motors receiving sufficiently large negative corrections would stop spinning.

By observing which motors stopped for each tilt direction, we could verify that the MPU orientation and motor assignments in our code were correctly aligned.

This was an important step in confirming that the flight controller was working as intended.




https://github.com/user-attachments/assets/687f62e6-286e-448e-9bb9-912fa9815546




### 5. Throttle Testing and Frame Redesign

Following these tests, we returned to our simpler ESC and emergency shutdown program, this time with propellers attached.

Our goal was to gradually increase the PWM signals and estimate the throttle required to generate enough thrust for takeoff and hovering.

We determined the minimum PWM signal needed for lift of the lego frame to be 1275 microseconds. This was also where we encountered our first major hardware failure.

During one test, we increased the PWM signal by too large an increment, generating more thrust and mechanical loading than our original LEGO frame could withstand. The connections between the arms and the main body failed, causing the frame to break apart.

To address this issue, I redesigned the frame using plywood and shorter, stronger arms.

I moved the motors approximately **5 cm closer to the centre of the drone**, reducing the bending moment at the arm-to-body connections for a given motor thrust.&#x20;

The redesigned frame has successfully withstood the motor throttle levels we've tested so far, while remaining lightweight enough for continued flight testing.

Although the initial failure was unexpected, it demonstrated the importance of considering structural loads alongside electrical and software design.



### 6. PD Controller Tuning (Ongoing)

We are currently tuning our Proportional-Derivative (PD) stabilization controller.

To simplify the process, we are testing one rotational axis at a time while suspending the drone using strings. This allows us to observe its response to disturbances without attempting an unrestricted flight.

During testing, the motors operate at a throttle slightly below our estimated hovering throttle, while the controller adjusts motor speeds to correct the drone's orientation.

We manually tilt the drone along the active axis and observe how the controller responds as we adjust the proportional gain (Kp) and derivative gain (Kd).

Our objective is to find gain values that allow the drone to return to a level orientation after a disturbance without excessive overshooting or sustained oscillations.

A new test method I've been developing is a test rig that is sturdier and anchors the drone to a fixed point and only allows it to rotate about one axis. This will remove the issue of the strings swinging and messing with how the drone has to adjust. A drawing of a prototype has been attached below.&#x20;
 

<img width="522" height="722" alt="currentnew" src="https://github.com/user-attachments/assets/a8088c48-a035-4b3f-9fe7-e3f3c6d6dc89" />
<img width="3024" height="4032" alt="testrig" src="https://github.com/user-attachments/assets/7921d4da-09b6-4601-b859-abbdd63dfe9c" />



### Future goals:
Firstly to get the drone to hover and accomplish our initial goal.
Second to make the drone remote controlled, rather than autonomous flight.
Third to build a custom PCB  to reduce the amount of space taken up by wiring and separate electronics. 
