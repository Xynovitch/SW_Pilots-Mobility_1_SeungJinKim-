## 1. Types and Methods of Sensors (Input Devices)
* Sensors act as input devices that read physical data from the external environment in computer and microcontroller systems.
* **Converting analog sensor values to digital figures:** If a sensor outputs continuous voltage values, like a temperature sensor (TMP36) or a light sensor, it goes through the internal ADC (Analog-to-Digital Converter) of the Arduino to be converted and processed into a digital value between 0 and 1023.
* **Having a digital sensor from the beginning:** Modules like the temperature and humidity sensor (DHT11) or ultrasonic sensor have built-in chips, meaning they deliver perfect 0 and 1 data packets directly through I2C communication, etc.
* **Major sensors that measure physical values:** Temperature sensors, humidity sensors, light (photoresistor) sensors, ultrasonic (distance) sensors, soil moisture sensors, PIR motion sensors, etc.

---

## 2. Understanding Actuators (Output Devices)
* **Definition of an actuator:** Refers to all driving devices that use electricity to express final output values that users can perceive, such as physical movement, light, or sound.
* **Devices that process using light:** LED (Light Emitting Diode), NeoPixel (Smart LED), LCD display panels, etc.
* **Motor-based actuators:** DC motors (continuous rotation), Servo motors (precise angle control), Stepper motors (moving by a fixed step), Linear actuators (straight piston movement).
* **Other output devices:** Piezo buzzers (sound/notification output), Water pumps (generating water pressure), Relays and Solenoid valves (switching and blocking), etc.

---

## 3. Combinations of Sensors and Actuators (Implementing Embedded Devices)
* **A device that closes the window when it rains:** When a rain detection moisture sensor recognizes water drops, it operates a servo motor or a linear actuator to physically close the window.
* **A flower pot that automatically supplies water when moisture is low:** A soil moisture sensor monitors the dryness of the soil in real-time, and when it drops below a standard level, a water pump is activated to supply water.
* **Operating a cooling fan when the temperature rises:** A temperature sensor continuously monitors the indoor temperature, and when it exceeds the set temperature, it rotates a DC motor (fan) to generate wind.
* **Integrated sensor and actuator devices:** A Servo motor (a variable resistor sensor that reads the current angle and a driving DC motor are built together inside), Smartphone camera module (an image sensor combined with a micro-motor for autofocus).

---

## 4. Real-world Embedded Equipment Analysis
* **Streetlight (Automatic Lighting System):** An ambient light sensor (input) measures the amount of light to detect sunset, and the system evaluates this to supply power to high-output LED lamps (actuator).
* **Entrance sensor light:** When a PIR infrared motion sensor (input) detects the movement of a human body's heat source, it turns on an LED bulb (actuator) for a set time via a relay module.
* **Automatic door:** An ultrasonic or radar sensor (input) attached to the top determines the approach distance of a person, and rotates a large DC motor (actuator) pulling the sliding door to open it.

---

## 5. Sensor Error Handling and Timing Control 
* **How to handle when measured values bounce or cause errors:** You should not react immediately if the measured value deviates severely due to hardware noise (an outlier). You should buffer and record the number of times an abnormal value occurs, and if it is a definite error, apply a skip logic for a set number of times to prevent the motor from rattling and malfunctioning.
* **Matching the timing of actuator operation:** The calculation speed of the microcontroller is much faster than the mechanical movement of the motor. After the measured value comes in from the sensor, an appropriate delay is required so that the actuator can finish moving to the designated position.
* **Initialization wait time:** Right after power is applied, it is necessary to secure the initialization time of the sensor and actuator in the code (a booting wait through `delay`) until the hardware module normalizes.
* **Controlling multiple sensors and actuators:** When driving two or more linear actuators at the same time, twisting may occur due to mechanical imbalance. A complex control algorithm is required to continuously read the feedback sensors inside each actuator and synchronize their speeds.