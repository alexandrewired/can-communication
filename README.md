# can-communication-network

CAN bus is a robust, standardized communication protocol originally developed for the automotive industry. It enables various electronic components—such as sensors, actuators, and control units—to communicate reliably even in harsh environments. This project explains the underlying principles and provides a practical example of setting up CAN communication between ESP32 boards.

CAN Bus differential Signaling - CAN bus uses two differential signal lines—CAN High (CAN-H) and CAN Low (CAN-L)—plus a common ground.

### Example of a transmitter node that sends "Hello":
```ruby

#include <SPI.h>
#include "mcp_can.h"

const int SPI_CS_PIN = 10;  // Change if needed
MCP_CAN CAN(SPI_CS_PIN);

void setup() {
  Serial.begin(115200);
  while (CAN_OK != CAN.begin(CAN_500KBPS)) {
    Serial.println("CAN init fail, retrying...");
    delay(100);
  }
  Serial.println("CAN init OK!");
}

void loop() {
  // Prepare the data bytes for "Hello"
  byte data[5] = {'H', 'e', 'l', 'l', 'o'};
  
  // Send the message with identifier 0x100
  if (CAN.sendMsgBuf(0x100, 0, 5, data) == CAN_OK) {
    Serial.println("Sent: Hello");
  } else {
    Serial.println("Send error");
  }
  
  delay(1000);  // Send every second
}
```

### Example of a receiver node that receives "Hello":
```ruby

#include <SPI.h>
#include "mcp_can.h"

const int SPI_CS_PIN = 10;  // Change if needed
MCP_CAN CAN(SPI_CS_PIN);

void setup() {
  Serial.begin(115200);
  while (CAN_OK != CAN.begin(CAN_500KBPS)) {
    Serial.println("CAN init fail, retrying...");
    delay(100);
  }
  Serial.println("CAN init OK!");
}

void loop() {
  if (CAN_MSGAVAIL == CAN.checkReceive()) {
    long unsigned int rxId;
    byte len = 0;
    byte rxBuf[8];
    
    CAN.readMsgBuf(&rxId, &len, rxBuf);
    
    Serial.print("Received [ID: 0x");
    Serial.print(rxId, HEX);
    Serial.print("]: ");
    
    // Convert received bytes into a string
    String message = "";
    for (int i = 0; i < len; i++) {
      message += (char)rxBuf[i];
    }
    Serial.println(message);
  }
}
```

## How It Works

  **Transmitter Node:**
  <ol>
  <li>Initializes the CAN bus at 500 kbps.</li>
  <li>Sends a 5-byte message containing the ASCII characters for "Hello" every second.</li>
  </ol> 
   

  **Receiver Node:**
  <ol>
  <li>Initializes the CAN bus at 500 kbps.</li>
  <li>Continuously checks for incoming messages.</li>
  <li>Reads the message, converts the received bytes into a string, and prints it to the serial monitor.</li>
  </ol> 
