# Light Sensors - เซ็นเซอร์วัดแสง (LDR & BH1750)

ความเข้มแสงเป็นค่าพรรณนาที่สำคัญในงาน DIY มีโมดูลยอดนิยม 2 แบบที่ช่างและนักประดิษฐ์นิยมใช้

---

## 🌓 1. LDR (Photoresistor)

LDR เป็นตัวต้านทานที่เปลี่ยนค่าตามความเข้มแสง ให้สัญญานเป็น Analog

### การต่อสาย (Analog)
มักใช้ร่วมกับตัวต้านทาน 10kΩ เพื่อทำวงจร Voltage Divider

=== "Arduino"
    ```
    LDR + 10k Resistor    Arduino
    ──────────────────    ───────
    Point between LDR/R → Analog Pin A0
    VCC (5V)            → 5V
    GND                 → GND
    ```

### โค้ดตัวอย่าง
```cpp
void setup() {
  Serial.begin(9600);
}

void loop() {
  int lightVal = analogRead(A0);
  Serial.print("Light level: ");
  Serial.println(lightVal); // 0 (สว่างมาก) - 1023 (มืดมาก - ขึ้นอยู่กับการต่อ)
  delay(500);
}
```

---

## 💡 2. BH1750 (Digital Light Sensor)

เป็นเซ็นเซอร์ที่ให้ค่าความเข้มแสงเป็นหน่วย **Lux** ผ่านโปรโตคอล I2C มีความแม่นยำสูงกว่า LDR มาก

### การต่อสาย (I2C)

=== "Arduino"
    ```
    BH1750 (GY-30)     Arduino Uno
    ──────────────     ───────────
    VCC          →     5V หรือ 3.3V
    GND          →     GND
    SCL          →     A5
    SDA          →     A4
    ```

=== "ESP32"
    ```
    BH1750 (GY-30)     ESP32
    ──────────────     ─────
    VCC          →     3.3V
    GND          →     GND
    SCL          →     GPIO 22
    SDA          →     GPIO 21
    ```

### โค้ดตัวอย่าง (ต้องลง Library BH1750)
```cpp
#include <Wire.h>
#include <BH1750.h>

BH1750 lightMeter;

void setup(){
  Serial.begin(9600);
  Wire.begin();
  lightMeter.begin();
}

void loop() {
  float lux = lightMeter.readLightLevel();
  Serial.print("Light: ");
  Serial.print(lux);
  Serial.println(" lx");
  delay(1000);
}
```

## 🛠️ โปรเจค DIY

### 1. เครื่องวัดความเข้มแสงแบบพกพา (Lux Meter)
ใช้ BH1750 ร่วมกับจอ OLED เพื่อทำเครื่องวัดความเข้มแสงสำหรับตากล้องหรือวัดความสว่างในบ้าน

### 2. ระบบเปิด-ปิดไฟถนนอัตโนมัติ
ใช้ LDR ตรวจสอบเมื่อฟ้ามืดค่ำลง ให้เปิดรีเลย์คุมไฟหน้าบ้าน
