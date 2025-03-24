# ESP32 Simple WiFi Server 🌐  

This project sets up an **ESP32 as a WiFi server**, allowing users to **control an LED** remotely through a web browser. The ESP32 connects to a **WiFi network**, listens for HTTP requests, and responds by **turning an LED ON, OFF, or enabling blink mode**.  

## Features  
- 🌍 Connects to an existing **WiFi network**.  
- 🖥️ Hosts a **web server on port 80**.  
- 💡 **Controls an LED** via a simple web request.  
- 🔄 **Supports Blink Mode** (LED blinks automatically).  

---

## 🛠️ Hardware Requirements  
- **ESP32 Dev Board**  
- **LED + Resistor** (or use the built-in LED on GPIO 2, which can be GPIO 5 or GPIO 22 on some ESP32 boards)  
- **USB cable & computer** (to upload code)  

---

## 📜 Code Overview: 

#include <WiFi.h>

const char *ssid = "Test123";      // WiFi SSID, you can change this based on your WiFi SSID
const char *password = "12345678"; // WiFi Password, you can change this based on your WiFi password

WiFiServer server(80);

int LEDPin = 2;                  // LED pin
bool blinkMode = false;          // Blinking mode
unsigned long previousMillis = 0;
const long interval = 500;
bool ledState = LOW;

void setup() {
  Serial.begin(115200);
  pinMode(LEDPin, OUTPUT);

  WiFi.begin(ssid, password);
  Serial.print("Connecting to WiFi");

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("\nWiFi connected!");
  Serial.print("IP Address: ");
  Serial.println(WiFi.localIP());

  server.begin();
}

void loop() {
  WiFiClient client = server.available(); // Check for incoming client
  
  if (client) {
    Serial.println("New Client Connected!");
    String currentLine = "";  
    unsigned long clientStartTime = millis(); // Track time for timeout

    while (client.connected()) {
      if (millis() - clientStartTime > 5000) {  // 5-second timeout
        Serial.println("Client timeout, closing connection.");
        break;
      }

      if (client.available()) {  
        char c = client.read();  
        Serial.write(c);        

        if (c == '\n') {  
          if (currentLine.length() == 0) {  // End of request
            client.println("HTTP/1.1 200 OK");
            client.println("Content-type:text/html");
            client.println();

            client.print("Click <a href=\"/H\">here</a> to make the LED on pin 2 turn on.<br>");
            client.print("Click <a href=\"/L\">here</a> to make the LED on pin 2 turn off.<br>");
            client.print("Click <a href=\"/B\">here</a> to make the LED on pin 2 blink.<br>");

            client.println();
            break;
          } else {
            currentLine = "";
          }
        } else if (c != '\r') {  
          currentLine += c;  
        }

        // Handle LED control
        if (currentLine.endsWith("GET /H")) {
          digitalWrite(LEDPin, HIGH);  
          blinkMode = false;  
        }
        if (currentLine.endsWith("GET /L")) {
          digitalWrite(LEDPin, LOW);  
          blinkMode = false;  
        }
        if (currentLine.endsWith("GET /B")) {  
          blinkMode = !blinkMode;
        }
      }
    }
    client.stop();
    Serial.println("Client Disconnected.");
  }

  // Ensure Serial output is always printing
  Serial.println("Waiting for clients...");

  // Non-blocking LED blinking logic
  if (blinkMode) {
    unsigned long currentMillis = millis();
    if (currentMillis - previousMillis >= interval) {
      previousMillis = currentMillis;
      ledState = !ledState;
      digitalWrite(LEDPin, ledState);
    }
  }

  delay(1000);
}