# DHT11 / DHT22 - เซ็นเซอร์วัดอุณหภูมิและความชื้น

!!! info "พื้นฐานอิเล็กทรอนิกส์"
    การใช้เซ็นเซอร์ DHT จำเป็นต้องทราบเรื่อง **[ตัวต้านทานแบบ Pull-Up](https://ghostmicro.github.io/micro-electronic/pull_up_pull_down/)** เพื่อความเสถียรของสัญญาณ

![DHT Sensor](../assets/images/dht.jpg)

## 📋 ภาพรวม

DHT11 และ DHT22 เป็นเซ็นเซอร์วัดอุณหภูมิและความชื้นที่ได้รับความนิยมสูงในโปรเจค DIY เนื่องจากราคาถูก ใช้งานง่าย และมีความแม่นยำพอสมควร

### ความแตกต่างระหว่าง DHT11 และ DHT22

| คุณสมบัติ         | DHT11             | DHT22 (AM2302)        |
| -------------- | ----------------- | --------------------- |
| ช่วงอุณหภูมิ       | 0-50°C            | -40-80°C              |
| ความแม่นยำอุณหภูมิ  | ±2°C              | ±0.5°C                |
| ช่วงความชื้น      | 20-90%            | 0-100%                |
| ความแม่นยำความชื้น | ±5%               | ±2-5%                 |
| อัตราการอ่านค่า   | 1 Hz (1 ครั้ง/วินาที) | 0.5 Hz (1 ครั้ง/2 วินาที) |
| ราคา           | ถูกกว่า             | แพงกว่า                |

## 🔌 การต่อสาย

### พินของเซ็นเซอร์

```
DHT11/DHT22 (มอง จากด้านหน้า)
┌─────────────┐
│  ┌───┐      │
│  │ O │      │  Pin 1: VCC (3.3V - 5V)
│  └───┘      │  Pin 2: DATA
│             │  Pin 3: NC (ไม่ใช้)
└─────────────┘  Pin 4: GND
 │ │ │ │
 1 2 3 4
```

!!! note "โมดูล DHT"
    หากซื้อแบบโมดูลสำเร็จรูป มักจะมี 3 พินเท่านั้น (VCC, DATA, GND) และมี pull-up resistor ติดมาให้แล้ว

### ไดอะแกรมการต่อสาย

=== "Arduino"
    ```
    DHT Sensor          Arduino
    ──────────          ───────
    VCC (Pin 1)    →    5V
    DATA (Pin 2)   →    Digital Pin 2
    GND (Pin 4)    →    GND
    
    * ต่อ Resistor 10kΩ ระหว่าง VCC และ DATA (pull-up)
    ```

=== "ESP32"
    ```
    DHT Sensor          ESP32
    ──────────          ─────
    VCC (Pin 1)    →    3.3V
    DATA (Pin 2)   →    GPIO 4
    GND (Pin 4)    →    GND
    
    * ต่อ Resistor 10kΩ ระหว่าง VCC และ DATA (pull-up)
    ```

=== "STM32"
    ```
    DHT Sensor          STM32
    ──────────          ─────
    VCC (Pin 1)    →    3.3V
    DATA (Pin 2)   →    PA1
    GND (Pin 4)    →    GND
    
    * ต่อ Resistor 10kΩ ระหว่าง VCC และ DATA (pull-up)
    ```

## 💻 โค้ดตัวอย่าง

### ติดตั้ง Library

=== "Arduino IDE"
    ```
    Sketch → Include Library → Manage Libraries
    ค้นหา: "DHT sensor library" by Adafruit
    ติดตั้งทั้ง DHT sensor library และ Adafruit Unified Sensor
    ```

=== "PlatformIO"
    ```ini
    [env:myboard]
    lib_deps =
        adafruit/DHT sensor library
        adafruit/Adafruit Unified Sensor
    ```

### Arduino

```cpp
#include <DHT.h>

#define DHTPIN 2        // พินที่ต่อกับ DATA
#define DHTTYPE DHT22   // เปลี่ยนเป็น DHT11 ถ้าใช้ DHT11

DHT dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(9600);
  Serial.println("DHT Sensor Test");
  dht.begin();
}

void loop() {
  delay(2000);  // รอ 2 วินาที (DHT22 ต้องการเวลาอ่านค่า)
  
  float humidity = dht.readHumidity();
  float temperature = dht.readTemperature();
  
  // ตรวจสอบว่าอ่านค่าสำเร็จหรือไม่
  if (isnan(humidity) || isnan(temperature)) {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }
  
  Serial.print("Humidity: ");
  Serial.print(humidity);
  Serial.print("%  Temperature: ");
  Serial.print(temperature);
  Serial.println("°C");
}
```

### ESP32

```cpp
#include <DHT.h>

#define DHTPIN 4        // GPIO 4
#define DHTTYPE DHT22

DHT dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(115200);
  Serial.println("ESP32 DHT Sensor");
  dht.begin();
}

void loop() {
  delay(2000);
  
  float h = dht.readHumidity();
  float t = dht.readTemperature();
  float f = dht.readTemperature(true);  // Fahrenheit
  
  if (isnan(h) || isnan(t) || isnan(f)) {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }
  
  // คำนวณ Heat Index
  float hif = dht.computeHeatIndex(f, h);
  float hic = dht.computeHeatIndex(t, h, false);
  
  Serial.print("Humidity: ");
  Serial.print(h);
  Serial.print("%  Temp: ");
  Serial.print(t);
  Serial.print("°C  Heat index: ");
  Serial.print(hic);
  Serial.println("°C");
}
```

### STM32 (HAL Library)

```cpp
// ใช้ DHT library สำหรับ STM32
#include "dht.h"

DHT_DataTypedef DHT22_Data;

int main(void) {
  HAL_Init();
  SystemClock_Config();
  
  while (1) {
    DHT_GetData(&DHT22_Data);
    
    printf("Temperature: %.1f°C\n", DHT22_Data.Temperature);
    printf("Humidity: %.1f%%\n", DHT22_Data.Humidity);
    
    HAL_Delay(2000);
  }
}
```

## 🛠️ โปรเจค DIY

### 1. สถานีตรวจวัดอากาศ (Weather Station)

สร้างสถานีตรวจวัดอุณหภูมิและความชื้นพร้อมแสดงผลบนจอ LCD

**อุปกรณ์:**
- DHT22
- LCD 16x2 (I2C)
- Arduino/ESP32

**ฟีเจอร์:**
- แสดงอุณหภูมิและความชื้นแบบ real-time
- บันทึกค่าสูงสุด/ต่ำสุดของวัน
- แจ้งเตือนเมื่ออุณหภูมิสูงเกินกำหนด

### 2. ระบบควบคุมพัดลมอัตโนมัติ

เปิด-ปิดพัดลมอัตโนมัติตามอุณหภูมิ

**อุปกรณ์:**
- DHT11/DHT22
- Relay Module
- พัดลม 220V
- Arduino

**การทำงาน:**
```cpp
if (temperature > 30) {
  digitalWrite(RELAY_PIN, HIGH);  // เปิดพัดลม
} else if (temperature < 28) {
  digitalWrite(RELAY_PIN, LOW);   // ปิดพัดลม
}
```

### 3. IoT Temperature Monitor (ESP32)

ส่งข้อมูลอุณหภูมิและความชื้นขึ้น Cloud

**อุปกรณ์:**
- DHT22
- ESP32

**แพลตฟอร์ม:**
- Blynk
- ThingSpeak
- Firebase

## ⚠️ ข้อควรระวัง

!!! warning "ข้อจำกัดการใช้งาน"
    - **อย่าอ่านค่าบ่อยเกินไป**: DHT11 ต้องการเวลาอย่างน้อย 1 วินาที, DHT22 ต้องการ 2 วินาที
    - **ใช้ Pull-up Resistor**: ต้องมี resistor 10kΩ ต่อระหว่าง VCC และ DATA
    - **ความยาวสาย**: ไม่ควรเกิน 20 เมตร เพื่อป้องกันสัญญาณรบกวน

!!! tip "เคล็ดลับ"
    - ใช้ DHT22 สำหรับงานที่ต้องการความแม่นยำสูง
    - ใช้ DHT11 สำหรับงานทั่วไปที่ไม่ต้องการความแม่นยำมาก
    - วาง sensor ห่างจากแหล่งความร้อน (เช่น ตัวต้านทาน, ชิป) อย่างน้อย 10 ซม.

## 🔗 แหล่งข้อมูลเพิ่มเติม

- [DHT11 Datasheet](https://www.mouser.com/datasheet/2/758/DHT11-Technical-Data-Sheet-Translated-Version-1143054.pdf)
- [DHT22 Datasheet](https://www.sparkfun.com/datasheets/Sensors/Temperature/DHT22.pdf)
- [Adafruit DHT Library](https://github.com/adafruit/DHT-sensor-library)
