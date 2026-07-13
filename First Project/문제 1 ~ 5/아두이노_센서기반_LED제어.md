## 1. Connecting Various Sensors to Arduino

In this project, we read data from temperature, light, and distance sensors to control a NeoPixel (WS2812B) LED strip/ring based on specific conditions. The pin connection configuration for each component is as follows:

* **TMP36 (Analog Temperature Sensor)**
* **Power (Left Pin)**: Arduino 5V
* **Vout (Middle Pin)**: Arduino Analog A1 pin
* **Ground (Right Pin)**: Arduino GND


* **LDR (Light Dependent Resistor / Photoresistor)**
* Connect one terminal of the light sensor to 5V, and the other terminal to GND through a 10kΩ resistor (pull-down resistor).
* **Vout (between the sensor and resistor)**: Arduino Analog A0 pin


* **Ultrasonic Distance Sensor (3-pin PING model)**
* **5V**: Arduino 5V
* **GND**: Arduino GND
* **SIG**: Arduino Digital pin 9


* **WS2812B (NeoPixel LED)**
* **PWR / 5V**: Arduino 5V
* **GND**: Arduino GND
* **IN / DIN (Data In)**: Arduino Digital pin 6



---

## 2. Filtering Out Sensor Errors (Error Filtering)

Physical sensors (or simulators) can return incorrect values due to noise, communication delays, or hardware limitations. For system stability, we filter errors as follows:

1. **Preventing ADC Crosstalk**: To prevent voltage interference that occurs when consecutively reading multiple analog pins (A0, A1) using the Arduino's single ADC (Analog-to-Digital Converter), we perform a 'Dummy Read' and add a `delay(10)` before actually reading the pin to stabilize the internal capacitor.
2. **Removing Light Sensor (LDR) Noise**: To prevent the value from fluctuating due to minute flickering of light, we apply an **Exponentially Weighted Moving Average (Low-pass filter)** to smooth the changes in the form of `New Value = (Previous Value * 0.7) + (Current Value * 0.3)`.
3. **Optimizing Ultrasonic Sensor Response**: By setting a timeout (e.g., 30000us) for the `pulseIn()` function, we prevent the program from blocking while waiting for a response. Abnormal values outside the measurement range are replaced with the previous normal value to prevent visual flickering.

---

## 3. Reading Sensor Data and Designing the LED Control Algorithm

We design a flicker-free algorithm that synthesizes sensor data to provide intuitive feedback.

* **Brightness Control (Light-based)**: The darker the surroundings, the higher the overall LED brightness (Brightness Multiplier) to ensure visibility. Conversely, when it is bright, the brightness is lowered to save power.
* **Color Control (Temperature-based)**: Determines the base color of the LEDs according to the current temperature. (Below 20°C = Blue, 20~28°C = Green, Above 28°C = Red)
* **Display Area Control (Distance-based)**: As an object detected by the ultrasonic sensor gets closer (within the 5cm~45cm range), the number of active LED pixels increases, representing the proximity level as a visual gauge.

---

## 4. Integrated Program Source Code

```cpp
#include <Adafruit_NeoPixel.h>

// Pin Configuration
#define TMPPIN A1      
#define LDRPIN A0
#define PINGPIN 9      
#define LEDPIN 6
#define NUMPIXELS 12 // Number of NeoPixel LEDs to use

Adafruit_NeoPixel pixels(NUMPIXELS, LEDPIN, NEO_GRB + NEO_KHZ800);

// Variables to store previous states for error filtering
float lastTemp = 24.0;
long lastDist = 0;
int lastLdr = 0;

void setup() {
  Serial.begin(9600);
  
  pixels.begin();
  pixels.clear();
  pixels.show();
  
  lastLdr = analogRead(LDRPIN);
}

// Distance measurement function for 3-pin ultrasonic sensor
long getDistance() {
  pinMode(PINGPIN, OUTPUT);
  digitalWrite(PINGPIN, LOW);
  delayMicroseconds(2);
  digitalWrite(PINGPIN, HIGH);
  delayMicroseconds(5); 
  digitalWrite(PINGPIN, LOW);
  
  pinMode(PINGPIN, INPUT);
  long duration = pulseIn(PINGPIN, HIGH, 30000); 
  
  if (duration == 0) return -1; 
  return duration * 0.034 / 2;
}

void loop() {
  // [1] Read Temperature Data (Apply ADC Crosstalk Filtering)
  analogRead(TMPPIN); 
  delay(10);          
  int tmpRaw = analogRead(TMPPIN); 
  
  float voltage = tmpRaw * 5.0 / 1024.0;
  float t = (voltage - 0.5) * 100.0;
  
  if (isnan(t)) {
    t = lastTemp; 
  } else {
    lastTemp = t;
  }
  
  // [2] Read Light Data and Low-pass Filtering
  analogRead(LDRPIN); 
  delay(10);
  int rawLdr = analogRead(LDRPIN);
  
  int ldrVal = (lastLdr * 0.7) + (rawLdr * 0.3);
  lastLdr = ldrVal;
  
  // [3] Read Distance Data and Validate
  long distance = getDistance();
  if (distance < 0 || distance > 400) {
    distance = lastDist; 
  } else {
    lastDist = distance;
  }

  // --- Integrated Control Algorithm (Apply Anti-Flicker Logic) ---
  
  // A. Determine Base Color according to Temperature
  int baseR = 0, baseG = 0, baseB = 0;
  if (t < 20.0) {
    baseB = 255;      // Cold: Blue
  } else if (t >= 20.0 && t <= 28.0) {
    baseG = 255;      // Comfortable: Green
  } else if (t > 28.0) {
    baseR = 255;      // Hot: Red
  }

  // B. Calculate Brightness Multiplier according to light level (Darker = Brighter, 0.1 ~ 1.0)
  float brightnessMult = map(ldrVal, 0, 1023, 100, 10) / 100.0;
  
  int finalR = baseR * brightnessMult;
  int finalG = baseG * brightnessMult;
  int finalB = baseB * brightnessMult;
  
  // C. Calculate Number of Active LEDs according to Distance
  int activeLeds = map(distance, 5, 45, NUMPIXELS, 0);
  activeLeds = constrain(activeLeds, 0, NUMPIXELS);
  
  // Batch Update LED Status (Avoid using clear() function)
  for(int i = 0; i < NUMPIXELS; i++) {
    if(i < activeLeds) {
      pixels.setPixelColor(i, pixels.Color(finalR, finalG, finalB));
    } else {
      pixels.setPixelColor(i, pixels.Color(0, 0, 0)); 
    }
  }
  
  pixels.show();
  delay(100);
}

```

---

## 5. Integrated Explanation of Sensor Data Processing and LED Control Logic

The program operates as a one-way pipeline: **Input (Read) -> Refine (Error Filtering) -> Calculate (Algorithm Mapping) -> Output (Visualization)**.

1. **Input and Refinement**: Noise caused by the Arduino's hardware characteristics (single ADC) is refined using software delays (`delay`) and a dummy read technique. Additionally, minute LDR fluctuations are mathematically corrected to secure stable raw data.
2. **Calculation (Mapping Algorithm)**: The three types of refined data are converted into independent parameters for LED control (base color, brightness multiplier, number of active LEDs) using the `map()` and `constrain()` functions. To avoid internal conflicts with the `setBrightness()` function, we adopted a method of multiplying the color values directly by the multiplier.
3. **Output (Visualization)**: The calculated final RGB values are assigned using a `for` loop, and unnecessary pixels are forcibly overwritten to an off state (0,0,0). Then, `pixels.show()` is called just once to update the hardware state smoothly and without flickering.

---

## 6. Result Examples

**System Circuit Connection Diagram**

**Execution Result 1: Close Proximity and Low Temperature State (Blue LED ON)**

**Execution Result 2: Mid-Distance and High Temperature State (Red LED ON)**