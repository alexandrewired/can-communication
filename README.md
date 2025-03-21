# can-communication

CAN bus is a robust, standardized communication protocol originally developed for the automotive industry. It enables various electronic components—such as sensors, actuators, and control units—to communicate reliably even in harsh environments. This project explains the underlying principles and provides a practical example of setting up CAN communication between ESP32 boards.

CAN Bus differential Signaling - CAN bus uses two differential signal lines—CAN High (CAN-H) and CAN Low (CAN-L)—plus a common ground.

### Example of a transmitter node that sends "Hello":
```ruby

#include <Arduino.h>
#include "driver/twai.h"

// Define the CAN TX and RX pins (adjust these to your wiring)
#define CAN_TX_PIN 5
#define CAN_RX_PIN 4

// Function to initialize the TWAI driver
void setupCAN() {
  twai_general_config_t g_config = TWAI_GENERAL_CONFIG_DEFAULT(CAN_TX_PIN, CAN_RX_PIN, TWAI_MODE_NORMAL);
  twai_timing_config_t t_config = TWAI_TIMING_CONFIG_500KBITS();
  twai_filter_config_t f_config = TWAI_FILTER_CONFIG_ACCEPT_ALL();

  if (twai_driver_install(&g_config, &t_config, &f_config) == ESP_OK) {
    Serial.println("TWAI driver installed.");
  } else {
    Serial.println("Failed to install TWAI driver.");
  }

  if (twai_start() == ESP_OK) {
    Serial.println("TWAI driver started.");
  } else {
    Serial.println("Failed to start TWAI driver.");
  }
}

void setup() {
  Serial.begin(115200);
  setupCAN();
}

void loop() {
  // Prepare a message to send "Hello"
  twai_message_t tx_msg;
  tx_msg.identifier = 0x100;        // Standard CAN ID
  tx_msg.extd = 0;                  // Using standard (11-bit) identifier
  tx_msg.rtr = 0;                   // Data frame, not a remote frame
  tx_msg.data_length_code = 5;      // 5 bytes of data
  tx_msg.data[0] = 'H';
  tx_msg.data[1] = 'e';
  tx_msg.data[2] = 'l';
  tx_msg.data[3] = 'l';
  tx_msg.data[4] = 'o';

  // Transmit the message (timeout of 1000 ms)
  if (twai_transmit(&tx_msg, pdMS_TO_TICKS(1000)) == ESP_OK) {
    Serial.println("Sent: Hello");
  } else {
    Serial.println("Transmit error");
  }

  vTaskDelay(pdMS_TO_TICKS(1000));  // Send every second
}
```

### Example of a receiver node that receives "Hello":
```ruby

#include <Arduino.h>
#include "driver/twai.h"

// Define the CAN TX and RX pins (use the same wiring as on the transmitter)
#define CAN_TX_PIN 5
#define CAN_RX_PIN 4

// Function to initialize the TWAI driver
void setupCAN() {
  twai_general_config_t g_config = TWAI_GENERAL_CONFIG_DEFAULT(CAN_TX_PIN, CAN_RX_PIN, TWAI_MODE_NORMAL);
  twai_timing_config_t t_config = TWAI_TIMING_CONFIG_500KBITS();
  twai_filter_config_t f_config = TWAI_FILTER_CONFIG_ACCEPT_ALL();

  if (twai_driver_install(&g_config, &t_config, &f_config) == ESP_OK) {
    Serial.println("TWAI driver installed.");
  } else {
    Serial.println("Failed to install TWAI driver.");
  }

  if (twai_start() == ESP_OK) {
    Serial.println("TWAI driver started.");
  } else {
    Serial.println("Failed to start TWAI driver.");
  }
}

void setup() {
  Serial.begin(115200);
  setupCAN();
}

void loop() {
  twai_message_t rx_msg;

  // Wait up to 1000ms for a message to be received
  if (twai_receive(&rx_msg, pdMS_TO_TICKS(1000)) == ESP_OK) {
    Serial.print("Received [ID: 0x");
    Serial.print(rx_msg.identifier, HEX);
    Serial.print("]: ");
    for (int i = 0; i < rx_msg.data_length_code; i++) {
      Serial.print((char)rx_msg.data[i]);
    }
    Serial.println();
  } else {
    // No message received in the specified timeout
    // Optionally, add a debug message here.
  }

  vTaskDelay(pdMS_TO_TICKS(10));
}
```

## How It Works
  **TWAI (CAN) initialization:**
  
Both sketches initialize the ESP32’s built-in TWAI (CAN) driver using a configuration structure. The CAN timing is set to 500 kbps, and a filter that accepts all messages is used.

  **Transmitter Node:**

The transmitter creates a CAN message with standard identifier 0x100 and 5 bytes containing "Hello". It then transmits the message every second.
   

  **Receiver Node:**

The receiver waits for a message (with a timeout of 1000 ms), then prints the CAN identifier and converts the received bytes into a string to display the message.
