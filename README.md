# Fingerprint-doorlock

## Project Overview
This project implements a secure and efficient Fingerprint Door Lock System using a fingerprint sensor and an Arduino microcontroller. The lock is activated only when an authorized fingerprint is scanned, ensuring a high level of security. A MOSFET is used to control the switching action, driving the locking mechanism based on the input from the fingerprint sensor. This project is designed for homes, offices, and other secure environments where controlled access is required.

## Key Features
- **Biometric Authentication:** The system uses a fingerprint sensor for biometric authentication, ensuring that only authorized users can unlock the door.
- **Arduino-Based Control:** The entire system is managed by an Arduino, which processes the fingerprint data and controls the locking mechanism.
- **MOSFET Switching:** A MOSFET is employed to drive the locking mechanism, allowing for efficient and reliable switching action.
- **Real-Time Access Control:** The fingerprint data is processed in real-time, enabling quick and secure access.
- **Power Efficiency:** With the use of a MOSFET, the system minimizes power consumption, making it ideal for long-term, low-power applications.
## Hardware Components
- **Fingerprint Sensor:** Captures the fingerprint data for identification.
- **Arduino:** Serves as the central processing unit, managing sensor data and controlling the lock.
- **MOSFET:** Used to control the current to the locking mechanism, enabling efficient switching.
- **Locking Mechanism:** Electric lock or servo motor to control the physical door lock.
- **Power Supply:** Provides necessary voltage for the Arduino, fingerprint sensor, and locking system.
## Circuit Description
The fingerprint sensor is connected to the Arduino to scan and verify fingerprints. Upon successful authentication, the Arduino sends a signal to the MOSFET, which controls the locking mechanism (e.g., electric lock or servo). The MOSFET ensures low-resistance switching to activate the lock, providing efficient and reliable operation.
![image](https://github.com/user-attachments/assets/4d17e3c6-2266-435d-9769-d8cb58b56d6a)

## Code
```
#include <Adafruit_Fingerprint.h>
#include <Servo.h>

// Define the pin connections for fingerprint sensor and solenoid lock
#define RST_PIN 5
#define FINGERPRINT_RX_PIN 2
#define FINGERPRINT_TX_PIN 3
#define MOSFET_PIN 8 // Pin to control the solenoid lock via MOSFET

// Create fingerprint sensor instance
Adafruit_Fingerprint finger = Adafruit_Fingerprint(&Serial1);

// Variables
int accessGranted = 0;
uint8_t id = 0;  // Fingerprint ID for enrolling

void setup()
{
  // Initialize serial communication
  Serial.begin(9600);
  Serial.println("Fingerprint sensor setup");

  // Set the MOSFET pin as OUTPUT
  pinMode(MOSFET_PIN, OUTPUT);
  
  // Initialize fingerprint sensor
  finger.begin(57600);
  if (finger.verifyPassword()) {
    Serial.println("Found fingerprint sensor!");
  } else {
    Serial.println("Did not find fingerprint sensor.");
    while (1) { delay(1); }
  }

  // Enroll new fingerprints (optional)
  Serial.println("Do you want to enroll new fingerprint? Press 'e' to enroll.");
  while (Serial.available() == 0);  // Wait for user input
  char userInput = Serial.read();
  if (userInput == 'e') {
    Serial.print("Enter Fingerprint ID (1-127): ");
    while (Serial.available() == 0);  // Wait for ID input
    id = Serial.parseInt();
    enrollFingerprint(id);
  }
}

void loop()
{
  // Check if fingerprint is matched
  accessGranted = getFingerprintID();
  
  if (accessGranted) {
    // Grant access by unlocking solenoid lock
    Serial.println("Access Granted!");
    unlockDoor();
    delay(5000); // Keep the door unlocked for 5 seconds
    lockDoor();
  }
}

// Function to enroll a new fingerprint
void enrollFingerprint(uint8_t id) {
  int p = -1;
  Serial.print("Enrolling ID #"); Serial.println(id);

  while (p != FINGERPRINT_OK) {
    p = finger.getImage();
    switch (p) {
      case FINGERPRINT_OK:
        Serial.println("Image taken");
        break;
      case FINGERPRINT_NOFINGER:
        Serial.println("No finger detected");
        break;
      case FINGERPRINT_PACKETRECIEVEERR:
        Serial.println("Communication error");
        break;
      case FINGERPRINT_IMAGEFAIL:
        Serial.println("Imaging error");
        break;
      default:
        return;
    }

    // Convert fingerprint image to template
    p = finger.image2Tz(1);
    if (p != FINGERPRINT_OK) {
      Serial.println("Error converting image");
      return;
    }

    Serial.println("Remove finger and place it again");

    // Wait for the finger to be removed
    delay(2000);
    while (finger.getImage() != FINGERPRINT_NOFINGER);

    // Get a second image
    p = finger.getImage();
    if (p != FINGERPRINT_OK) {
      Serial.println("Error capturing second image");
      return;
    }

    // Convert second image to template
    p = finger.image2Tz(2);
    if (p != FINGERPRINT_OK) {
      Serial.println("Error converting second image");
      return;
    }

    // Create the model from both images
    p = finger.createModel();
    if (p != FINGERPRINT_OK) {
      Serial.println("Error creating fingerprint model");
      return;
    }

    // Store the model into the sensor's flash memory
    p = finger.storeModel(id);
    if (p == FINGERPRINT_OK) {
      Serial.println("Fingerprint stored!");
    } else {
      Serial.println("Error storing fingerprint");
    }
  }
}

// Function to get the fingerprint ID
int getFingerprintID() {
  uint8_t p = finger.getImage();
  switch (p) {
    case FINGERPRINT_OK:
      Serial.println("Image taken");
      break;
    case FINGERPRINT_NOFINGER:
      return 0;
    case FINGERPRINT_PACKETRECIEVEERR:
      Serial.println("Communication error");
      return 0;
    case FINGERPRINT_IMAGEFAIL:
      Serial.println("Imaging error");
      return 0;
    default:
      return 0;
  }

  // Convert fingerprint image to template
  p = finger.image2Tz();
  if (p != FINGERPRINT_OK) return 0;

  // Search the sensor for a matching fingerprint
  p = finger.fingerFastSearch();
  if (p == FINGERPRINT_OK) {
    Serial.println("Fingerprint match found!");
    return finger.fingerID;
  } else {
    Serial.println("No match found");
    return 0;
  }
}

// Function to unlock the door
void unlockDoor() {
  digitalWrite(MOSFET_PIN, HIGH); // Turn on the MOSFET (open solenoid lock)
  Serial.println("Door unlocked");
}

// Function to lock the door
void lockDoor() {
  digitalWrite(MOSFET_PIN, LOW); // Turn off the MOSFET (close solenoid lock)
  Serial.println("Door locked");
}
```
## Steps to Enroll a New Fingerprint:
- Power the System and Open Serial Monitor: After uploading the code, open the serial monitor.
- Enter Enrollment Mode: Type 'e' in the serial monitor and hit Enter.
- Provide a Fingerprint ID: Enter a number between 1 and 127 to assign a unique ID for the new fingerprint.
- Place Finger Twice: Follow the prompts to place and remove the finger twice for accurate registration.
- Fingerprint Stored: Once stored, the fingerprint can be used to unlock the door.

## Advantages
- **High Security:** Biometric authentication ensures that only authorized personnel can access the locked area, significantly enhancing security.
- **Power Efficiency:**  The MOSFET enables low-power consumption, making the system more efficient compared to traditional mechanical locks.
- **Cost-Effective:** The use of Arduino and MOSFET makes the system both affordable and customizable for various applications.
- **Fast Response Time:** The real-time processing of fingerprints allows for quick unlocking without any delay.
- **Scalability:** The system can be easily expanded to support multiple users or integrated into larger security systems.
  
## Future Advancements
- **Wi-Fi and IoT Integration:** By adding a Wi-Fi module, the system could be controlled remotely via a smartphone or web interface, allowing users to monitor and control access from anywhere.
- **Mobile App:** Develop a mobile app for user management, allowing new fingerprints to be added or removed remotely.
- **Voice Command Integration:** Future versions of the system could include voice control, allowing the lock to be opened via smart assistants like Alexa or Google Assistant.
- **Face Recognition:** Incorporating face recognition in addition to fingerprint scanning for an even higher level of security.
- **Multi-factor Authentication:** Combining fingerprint scanning with PIN entry or RFID cards for dual-layered authentication, increasing security.
- **Battery Backup:** Adding an integrated battery backup to ensure the lock functions even during power outages.
- **Tamper Alerts:** Implementing a system that sends alerts to the user’s phone or security team if any tampering or forced entry is detected.
  
## Applications
- **Home Security:** Secure entry for homes, allowing only authorized individuals to unlock the door.
- **Office Security:** Can be used in offices for restricted access to certain rooms or facilities.
- **Industrial Access:**  Suitable for controlling access to sensitive areas in industrial settings.
  
## Conclusion
The Fingerprint Door Lock System provides a cutting-edge, secure, and efficient access control solution. With biometric authentication, low-power operation, and MOSFET-controlled switching, this project offers a robust, cost-effective system suitable for various applications. The potential for future advancements ensures that the system can be adapted to evolving security needs, making it a versatile solution for modern access control.

