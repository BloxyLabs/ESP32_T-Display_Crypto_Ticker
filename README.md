# ESP32 T-Display Crypto Ticker

Check also our [YouTube](https://www.youtube.com/@bloxylabs "YouTube") channel for instructions and other related information.

If you had fun with the projects, please consider giving us a Super Thanks on YouTube or buying us a [cup of coffee](https://www.buymeacoffee.com/bloxylabs "cupofcoffee") ☕.

![Image](https://github.com/user-attachments/assets/17f229f1-0bb5-4b50-925c-797a9aa23661)

<h3><u>The basic code for the ESP32 T-Display Crypto Ticker</u></h3>

```
#include <WiFi.h>
#include <HTTPClient.h>
#include <ArduinoJson.h>
#include <TFT_eSPI.h>

#define DISPLAY_POWER_PIN 15 // Pin to control display power

// Wi-Fi credentials
const char* ssid = "YOUR_SSID";
const char* password = "YOUR_PASSWORD";

// API endpoints
const char* ethApi = "https://min-api.cryptocompare.com/data/price?fsym=ETH&tsyms=USD";
const char* solApi = "https://min-api.cryptocompare.com/data/price?fsym=SOL&tsyms=USD";
const char* btcApi = "https://min-api.cryptocompare.com/data/price?fsym=BTC&tsyms=USD";
const char* xrpApi = "https://min-api.cryptocompare.com/data/price?fsym=XRP&tsyms=USD";


// Data storage
float ethUsd = 0, prevEthUsd = 0;
float solUsd = 0, prevSolUsd = 0;
float btcUsd = 0, prevBtcUsd = 0;
float xrpUsd = 0, prevXrpUsd = 0;

TFT_eSPI tft = TFT_eSPI(); // Initialize display

void setup() {
    Serial.begin(115200);

    // Turn on display power
    pinMode(DISPLAY_POWER_PIN, OUTPUT);
    digitalWrite(DISPLAY_POWER_PIN, HIGH);
    delay(500); // Ensure display power is stable

    // Initialize display
    tft.init();
    tft.setRotation(3); // Landscape mode
    tft.fillScreen(TFT_BLACK);
    tft.setTextSize(2);

    // Display Wi-Fi connection status
    tft.setCursor(10, 10);
    tft.println("Connecting to Wi-Fi...");
    connectToWiFi();

    // Fetch and display data
    fetchAndDisplayData();
}

void loop() {
    // Refresh data every 30 seconds
    delay(30000);
    fetchAndDisplayData();
}

void connectToWiFi() {
    WiFi.disconnect(true); // Clear previous configurations
    WiFi.mode(WIFI_STA);   // Set Wi-Fi to station mode
    WiFi.begin(ssid, password);

    unsigned long startAttemptTime = millis();
    const unsigned long wifiTimeout = 30000; // Timeout after 30 seconds

    while (WiFi.status() != WL_CONNECTED && millis() - startAttemptTime < wifiTimeout) {
        delay(1000);
        Serial.println("Connecting to Wi-Fi...");
        tft.fillScreen(TFT_BLACK);
        tft.setCursor(10, 10);
        tft.println("Connecting to Wi-Fi...");
    }

    if (WiFi.status() == WL_CONNECTED) {
        Serial.println("Connected to Wi-Fi");
        tft.fillScreen(TFT_BLACK);
        tft.setCursor(10, 10);
        tft.println("Wi-Fi Connected!");
    } else {
        Serial.println("Failed to connect to Wi-Fi");
        tft.fillScreen(TFT_BLACK);
        tft.setCursor(10, 10);
        tft.println("Wi-Fi Connection Failed");
    }
}

void fetchAndDisplayData() {
    // Fetch cryptocurrency prices
    fetchCryptoPrice(ethApi, ethUsd, prevEthUsd);
    fetchCryptoPrice(solApi, solUsd, prevSolUsd);
    fetchCryptoPrice(btcApi, btcUsd, prevBtcUsd);
    fetchCryptoPrice(xrpApi, xrpUsd, prevXrpUsd);

    // Display prices
    displayCryptoPrices();
}

void fetchCryptoPrice(const char* apiUrl, float& usdPrice, float& prevUsdPrice) {
    if (WiFi.status() != WL_CONNECTED) {
        Serial.println("Wi-Fi not connected. Skipping API call.");
        return;
    }

    HTTPClient http;
    http.begin(apiUrl);
    int httpResponseCode = http.GET();

    if (httpResponseCode > 0) {
        String payload = http.getString();
        Serial.println(payload);

        // Parse JSON response
        DynamicJsonDocument doc(1024);
        deserializeJson(doc, payload);

        prevUsdPrice = usdPrice; // Store previous price
        usdPrice = doc["USD"].as<float>();
    } else {
        Serial.printf("Error fetching data: HTTP %d\n", httpResponseCode);
        usdPrice = -1; // Indicate an error
    }

    http.end();
}

void displayCryptoPrices() {
    tft.fillScreen(TFT_BLACK);

    // Display BTC price
    tft.setCursor(10, 10);
    tft.setTextColor(btcUsd > prevBtcUsd ? TFT_GREEN : (btcUsd < prevBtcUsd ? TFT_RED : TFT_WHITE), TFT_BLACK);
    tft.printf("BTC: $%.2f", btcUsd);
   
    // Display ETH price
    tft.setCursor(10, 40);
    tft.setTextColor(ethUsd > prevEthUsd ? TFT_GREEN : (ethUsd < prevEthUsd ? TFT_RED : TFT_WHITE), TFT_BLACK);
    tft.printf("ETH: $%.2f", ethUsd);

    // Display XRP price
    tft.setCursor(10, 70);
    tft.setTextColor(xrpUsd > prevXrpUsd ? TFT_GREEN : (xrpUsd < prevXrpUsd ? TFT_RED : TFT_WHITE), TFT_BLACK);
    tft.printf("XRP: $%.2f", xrpUsd);

    // Display SOL price
    tft.setCursor(10, 100);
    tft.setTextColor(solUsd > prevSolUsd ? TFT_GREEN : (solUsd < prevSolUsd ? TFT_RED : TFT_WHITE), TFT_BLACK);
    tft.printf("SOL: $%.2f", solUsd);


}
```
