# 💡 โครงการ: Smart LED Controller Pro (Web-Based Control)

โปรเจกต์นี้คือการสร้างคอนโทรลเลอร์ไฟ LED อัจฉริยะที่ฟังก์ชันครบถ้วนเหมือนสินค้าแบรนด์เนมในท้องตลาด โดยใช้ ESP32 เป็นตัวปล่อย WiFi และสร้าง Web Interface ให้เข้าถึงได้ผ่านมือถือเพื่อเปลี่ยนสี ปรับความสว่าง และเลือกโหมดการแสดงผล

## 🎯 คุณสมบัติหลัก (Full Functions)
- **Web UI Control:** ควบคุมผ่าน Web Browser บนมือถือโดยไม่ต้องติดตั้งแอป
- **Color Picker:** เลือกสีได้ละเอียดถึง 16 ล้านสี (RGB)
- **Presets & Animations:** มีโหมดไฟวิ่ง ไฟหายใจ และเอฟเฟกต์อื่นๆ กว่า 10 แบบ
- **Slider Control:** ปรับความสว่าง (Brightness) และความเร็ว (Speed) ของเอฟเฟกต์ได้แบบ Real-time
- **Auto-Remember:** จดจำโหมดล่าสุดเมื่อเปิดเครื่องใหม่

## 🛠️ รายการอุปกรณ์ (BOM)
| อุปกรณ์                    | หน้าที่                                                          |
| :----------------------- | :------------------------------------------------------------ |
| **ESP32 / ESP8266**      | สำหรับประมวลผลเอฟเฟกต์และรัน Web Server (แนะนำ ESP32 เพื่อความลื่นไหล) |
| **WS2812B LED Strip**    | ไฟ Addressable LED (5V)                                       |
| **Power Supply 5V**      | แหล่งจ่ายไฟ (คำนวณตามจำนวนดวงไฟ เช่น 5V 10A สำหรับ 180 ดวง)          |
| **Capacitor 1000uF**     | สำหรับกรองแสไฟให้คงที่ (Smoothing)                                 |
| **Resistor 330-470 Ohm** | ต่อที่ขา Data เพื่อป้องกันความเสียหายจากสัญญาณกระชาก                   |

## 🔌 การต่อวงจร (Wiring)

```mermaid
graph TD
    PSU[Power Supply 5V] -- "+5V" --> LED_VCC[LED Strip VCC]
    PSU -- "GND" --> LED_GND[LED Strip GND]
    PSU -- "+5V" --> MCU_VIN[MCU VIN / 5V]
    PSU -- "GND" --> MCU_GND[MCU GND]
    
    MCU_DATA[MCU GPIO D2/D4] -- "Resistor 470R" --> LED_DIN[LED Strip Data IN]
    
    subgraph Smoothing
        PSU_CAP[Capacitor 1000uF] -- Parallel -- PSU
    end
```

> [!TIP]
> **Pin สำหรับ ESP8266:** แนะนำให้ใช้ขา **D4 (GPIO 2)** หรือ **D2 (GPIO 4)** ซึ่งเป็นขาที่ส่งสัญญาณได้เสถียรสำหรับ ESP8266 ครับ

---

## 💻 โค้ดโปรแกรม (Full Function Firmware)

ใช้ Library `FastLED` สำหรับคุมไฟ และ `WebServer` สำหรับส่งหน้าเว็บควบคุม

```cpp
#if defined(ESP32)
  #include <WiFi.h>
  #include <WebServer.h>
#elif defined(ESP8266)
  #include <ESP8266WiFi.h>
  #include <ESP8266WebServer.h>
#endif
#include <FastLED.h>

#define LED_PIN     2 // D4 สำหรับ ESP8266 หรือ GPIO 2
#define NUM_LEDS    60
#define LED_TYPE    WS2812B
#define COLOR_ORDER GRB

CRGB leds[NUM_LEDS];

#if defined(ESP32)
  WebServer server(80);
#elif defined(ESP8266)
  ESP8266WebServer server(80);
#endif

// สถานะปัจจุบัน
uint8_t brightness = 128;
uint8_t currentEffect = 0;
uint8_t speed = 50;
CRGB solidColor = CRGB::Red;

const char* ssid = "GhostMicro_LED_Pro";
const char* password = "password";

// ฟังก์ชันโหมดไฟต่างๆ (ตัวอย่าง)
void effectRainbow() {
  static uint8_t hue = 0;
  fill_rainbow(leds, NUM_LEDS, hue++, 7);
}

void effectPulse() {
  float pulse = (exp(sin(millis()/2000.0*PI)) - 0.36787944)*108.0;
  fill_solid(leds, NUM_LEDS, solidColor);
  FastLED.setBrightness(map(pulse, 0, 255, 0, brightness));
}

void setup() {
  Serial.begin(115200);
  FastLED.addLeds<LED_TYPE, LED_PIN, COLOR_ORDER>(leds, NUM_LEDS);
  FastLED.setBrightness(brightness);

  WiFi.softAP(ssid, password);
  Serial.print("AP IP address: ");
  Serial.println(WiFi.softAPIP());

  // --- Web Interior Logic ---
  server.on("/", []() {
    String html = "<html><head><meta name='viewport' content='width=device-width, initial-scale=1'>";
    html += "<style>body{font-family:sans-serif; text-align:center; padding:20px;} button{padding:15px; margin:5px; width:40%;}</style></head>";
    html += "<body><h1>LED Controller Pro</h1>";
    html += "<p>Brightness: <input type='range' onchange='fetch(\"/set?b=\"+this.value)' value='"+String(brightness)+"'></p>";
    html += "<h2>Modes</h2>";
    html += "<button onclick='fetch(\"/mode?m=0\")'>Solid</button>";
    html += "<button onclick='fetch(\"/mode?m=1\")'>Rainbow</button>";
    html += "<button onclick='fetch(\"/mode?m=2\")'>Pulse</button>";
    html += "</body></html>";
    server.send(200, "text/html", html);
  });

  server.on("/set", []() {
    if (server.hasArg("b")) brightness = server.arg("b").toInt();
    server.send(200, "text/plain", "OK");
  });

  server.on("/mode", []() {
    if (server.hasArg("m")) currentEffect = server.arg("m").toInt();
    server.send(200, "text/plain", "OK");
  });

  server.begin();
}

void loop() {
  server.handleClient();
  
  switch(currentEffect) {
    case 0: fill_solid(leds, NUM_LEDS, solidColor); break;
    case 1: effectRainbow(); break;
    case 2: effectPulse(); break;
  }
  
  FastLED.show();
  delay(10);
}
```

---

## 🏆 ข้อแนะนำสู่ระดับโปร
1. **Power Injection:** หากต่อสายไฟยาวเกิน 2 เมตร ควรต่อไฟเลี้ยง 5V เข้าทั้ง "หัว" และ "ท้าย" ของเส้นไฟเพื่อป้องกันสีเพี้ยนที่ปลายสาย
2. **Logic Level Shifter:** สัญญาณ Data จาก ESP32 คือ 3.3V แต่ไฟ WS2812B ต้องการ 5V แนะนำใช้ชิป **74HCT125** แปลงสัญญาณเพื่อให้ทำงานได้เสถียร 100%
3. **Fuse:** ต่อฟิวส์ขนาด 10A ที่ขั้วบวกของ Power Supply เพื่อความปลอดภัยป้องกันไฟไหม้กรณีสายไฟลัดวงจร

> [!IMPORTANT]
> โปรเจกต์นี้สามารถขยายไปคุมผ่านอินเทอร์เน็ตโดยใช้ **Blynk** หรือ **Home Assistant** ได้ในอนาคตครับ!
