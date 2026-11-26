# BW Fire Alarm Notifier

ระบบแจ้งเตือนเสริม: จับเสียงตู้ Fire Alarm ที่ชั้นใต้ดิน แล้วส่งเสียงเตือนที่เคาน์เตอร์ชั้น 2 ผ่านอินเทอร์เน็ต (EMQX Cloud)

> ⚠️ ใช้แจ้งเตือนเสริมเท่านั้น เมื่อได้ยินเสียงเตือน ให้ยืนยันสถานะที่หน้าตู้ NOTIFIER ทุกครั้ง

```
ตัวส่ง (ESP32 + KY-037) ──► EMQX Cloud ──► ตัวรับ (ESP32 + Buzzer)
                                 └──────► หน้าเว็บสถานะ (GitHub Pages)
```

## โครงสร้างไฟล์

| ไฟล์ | ใช้ทำอะไร |
|---|---|
| `FireAlarm/FireAlarm.ino` | เฟิร์มแวร์ตัวเดียว ใช้ได้ทั้งตัวส่งและตัวรับ |
| `docs/index.html` | หน้าเว็บดูสถานะ (อัปโหลดขึ้น GitHub Pages) |
| `emqxsl-ca.crt` | CA Certificate ของ EMQX (ฝังไว้ในโค้ดแล้ว) |

## การต่อสาย

| อุปกรณ์ | ขาอุปกรณ์ | ESP32 |
|---|---|---|
| KY-037 (ตัวส่ง) | VCC / GND / DO | 3V3 / GND / GPIO27 |
| Active Buzzer (ตัวรับ) | VCC / GND / I/O | 3V3 / GND / GPIO26 |
| ปุ่ม | ใช้ปุ่ม **BOOT** บนบอร์ด (GPIO0) | — |

- KY-037: ไม่ต้องตั้งขั้วสัญญาณ บอร์ดวัดเองตอนเปิดเครื่อง (ช่วง 1 วินาทีแรกควรเงียบ)
- Buzzer ตั้งค่าเริ่มต้นเป็นแบบ low level trigger (`BUZZER_ACTIVE_HIGH = false`) ถ้าเป็นแบบ high trigger → แก้เป็น `true`

## คอมไพล์ด้วย Arduino IDE

| ค่า | ต้องเป็น |
|---|---|
| แพ็กเกจบอร์ด | esp32 by Espressif **3.3.12** (ห้าม 4.x) |
| Board | ESP32 Dev Module |
| Flash Frequency | **40MHz** |
| Flash Mode | DIO |
| Partition Scheme | Default 4MB with spiffs |
| Upload Speed | 115200 |

ถ้าแพ็กเกจหรือ Partition ผิด จะคอมไพล์ไม่ผ่านพร้อมข้อความบอกวิธีแก้ ถ้า Flash ไม่ใช่ 40MHz หน้าสถานะจะขึ้นเตือนในแถว "การคอมไพล์"

0. เปิดไฟล์ `FireAlarm/secrets.h` แล้วใส่รหัสผ่านของผู้ใช้ `sender` และ `receiver` ที่ตั้งไว้ใน EMQX
   ตอนตั้งค่าบอร์ดจะเหลือแค่เลือก Wi-Fi + เลือกตัวส่ง/ตัวรับ
   > ⚠️ **ห้ามอัปโหลด `secrets.h` ขึ้น GitHub** และ **ห้ามอัปโหลดไฟล์ .bin ที่คอมไพล์พร้อมรหัสขึ้น repo สาธารณะ** เพราะคนอื่นดึงรหัสออกจากไฟล์ .bin ได้
1. Library Manager → ติดตั้ง **PubSubClient** (by Nick O'Leary)
2. Tools → Board: **ESP32 Dev Module**
3. ⚠️ วางโฟลเดอร์โปรเจกต์ไว้ใน path ภาษาอังกฤษ ไม่มีเว้นวรรค เช่น `C:\Arduino\FireAlarm\`

## เปิดหน้าเว็บด้วย GitHub Pages

1. อัปโหลด**เฉพาะโฟลเดอร์ `docs` และ `README.md`** ขึ้น repo ของบัญชี GitHub ของระบบ
   (โค้ดใน `FireAlarm/` เก็บไว้ในเครื่อง เพราะมี `secrets.h`)
2. Settings → Pages → Source: **Deploy from a branch** → Branch: `main` / โฟลเดอร์ `/docs` → Save
3. รอ 1–2 นาที จะได้ลิงก์ `https://<username>.github.io/<repo>/`

## ตั้งค่าบอร์ดครั้งแรก

1. เปิดเครื่อง → LED กระพริบเร็ว = อยู่ในโหมดตั้งค่า
2. มือถือเชื่อม Wi-Fi **FireAlarm-Setup-xxxxxx** รหัส **bwfire2026**
3. หน้าตั้งค่าจะเด้งขึ้นมา (ถ้าไม่เด้งเปิด `192.168.4.1`)
4. เลือก Wi-Fi + ใส่รหัส Wi-Fi แล้วเลือก **ตัวส่ง** หรือ **ตัวรับ**
   (ส่วน MQTT ใช้ค่าที่ฝังจาก `secrets.h` อัตโนมัติ ถ้าไม่ได้ฝังไว้ต้องกรอก username/password เอง)
5. กดบันทึก → บอร์ดรีสตาร์ตแล้วเชื่อมต่อเอง

กดปุ่ม BOOT ค้าง **10 วินาที** = ล้างค่าทั้งหมด แล้วกลับเข้าโหมดตั้งค่า

## ความหมายของไฟและเสียง

**LED บนบอร์ด (ทั้งสองตัว)**

| LED | ความหมาย |
|---|---|
| กระพริบเร็วมาก | โหมดตั้งค่า (รอมือถือเชื่อมต่อ) / ตัวรับ: ตู้กำลังดัง |
| กระพริบช้า (1 วิ) | กำลังเชื่อมต่อ Wi-Fi |
| กระพริบ (0.5 วิ) | ต่อ Wi-Fi ได้แต่ยังต่อ broker ไม่ได้ / ตัวรับ: ขาดการติดต่อตัวส่ง |
| ติดค้าง | ทำงานปกติ |
| ดับวูบสั้น ๆ | ตัวส่ง: ได้ยินเสียงดัง (ใช้ตอนปรับความไว KY-037) |

**Buzzer (ตัวรับ)**

| เสียง | ความหมาย |
|---|---|
| ดังถี่ต่อเนื่อง | ตู้ Fire Alarm กำลังดัง (กดปุ่ม BOOT = ปิดเสียง) |
| บี๊บ 3 ครั้ง | มีคนกดปุ่มทดสอบที่ตัวส่ง |
| บี๊บสั้น 2 ครั้ง ทุก 10 วิ | ขาดการติดต่อกับตัวส่งเกิน 30 วินาที |
| บี๊บยาว 1 ครั้ง | เปิดเครื่องแล้ว พร้อมใช้งาน |

## ปรับความไวการจับเสียง

1. เปิดหน้าสถานะของตัวส่งจากมือถือในวง Wi-Fi เดียวกัน: `http://bw-firealarm.local` (ตัวรับ: `http://bw-firealarm-receiver.local`) ถ้าเปิดไม่ได้ให้ใช้ IP แทน
2. ดูค่า **ระดับเสียง (ครั้ง/วินาที)** ตอนเงียบ และตอนตู้ดัง
3. หมุนตัวปรับสีฟ้าบน KY-037 ให้ตอนเงียบค่าใกล้ 0 และตอนตู้ดังค่าสูงกว่า 20 ชัดเจน
4. ตัวส่งจะเตือนเมื่อเสียงบี๊บ ๆ ต่อเนื่องเกิน 5 วินาที ถ้าจำเป็นแก้ค่าในโค้ด: `LOUD_EDGES_PER_SEC`, `ON_SUSTAIN_SEC`, `ON_MIN_LOUD_PCT`, `EPISODE_GAP_SEC`, `OFF_QUIET_SEC`

## MQTT topics (prefix `bwfa`)

| Topic | ข้อมูล |
|---|---|
| `bwfa/sender/online` | `1`/`0` (retained, Last Will = `0`) |
| `bwfa/sender/alarm` | `1`/`0` (retained) |
| `bwfa/sender/event` | `{"type":"alarm_on"\|"alarm_off"\|"test"}` |
| `bwfa/sender/info` | JSON สถานะ ทุก 30 วิ (retained) |
| `bwfa/receiver/online` | `1`/`0` (retained) |
| `bwfa/receiver/event` | `{"type":"silenced"}` |
| `bwfa/receiver/info` | JSON สถานะ ทุก 30 วิ (retained) |

## ข้อมูลสำหรับส่งมอบ

- อีเมลระบบ: `bw.firealarm@gmail.com`
- Broker: EMQX Cloud Serverless, Spend Limit $0
  - Address: `bd343e82.ala.asia-southeast1.emqxsl.com` พอร์ต 8883 (บอร์ด) / 8084 (หน้าเว็บ)
  - ผู้ใช้: `sender`, `receiver`, `dashboard` (dashboard ถูกห้าม publish)
- CA Certificate: DigiCert Global Root G2 ใช้ได้ถึง 15 ม.ค. 2038
- รหัสผ่านทุกบัญชีเก็บแยกไว้ในเอกสารส่งมอบ (ไม่อยู่ในไฟล์นี้)

## อัปเดตเฟิร์มแวร์ผ่าน Wi-Fi (ไม่ต้องเสียบ USB)

1. แก้ `FW_VERSION` ในโค้ด แล้ว Sketch → **Export Compiled Binary**
2. ใช้คอมหรือมือถือที่อยู่ Wi-Fi วงเดียวกับบอร์ด เปิด
   - ตัวส่ง: `http://bw-firealarm.local/update`
   - ตัวรับ: `http://bw-firealarm-receiver.local/update`
   (เปิดชื่อ .local ไม่ได้ให้ใช้ `http://<IP>/update`)
3. ใส่ชื่อผู้ใช้ `admin` และรหัส = **รหัส MQTT ของบอร์ดนั้น** (sender หรือ receiver)
4. เลือกไฟล์ `build/esp32.esp32.esp32/FireAlarm.ino.bin` **ไฟล์เดียว** แล้วกดอัปโหลด
5. รอบอร์ดรีสตาร์ต แล้วเช็กเวอร์ชันในหน้าสถานะ (ค่า Wi-Fi และรหัสไม่หาย)

บอร์ดใหม่ที่ยังไม่มีโปรแกรมเลย: อัปโหลดครั้งแรกผ่าน USB ด้วย Arduino IDE
ห้ามเปลี่ยน Partition Scheme (ต้องเป็น Default 4MB with spiffs) ไม่อย่างนั้นอัปเดตผ่าน Wi-Fi ไม่ได้และค่าตั้งค่าจะหาย

## อัปเดตเฟิร์มแวร์จากที่ไหนก็ได้ (ผ่าน GitHub)

ตั้งค่าครั้งเดียว:
1. ใส่ลิงก์ GitHub Pages ใน `FW_MANIFEST_URL` (บนสุดของ FireAlarm.ino) เช่น `https://<username>.github.io/<repo>/firmware/firmware.json`
2. อัปโหลดเวอร์ชันนี้ลงบอร์ดทั้งสองตัวผ่าน USB 1 ครั้ง
3. ใน EMQX สร้างผู้ใช้ `admin` (ใช้กดปุ่ม "สั่งตรวจอัปเดต" ในหน้าเว็บ)

ทุกครั้งที่มีเวอร์ชันใหม่:
1. แก้ `FW_VERSION` ในโค้ด → ตรวจว่า `secrets.h` ว่าง → Sketch → Export Compiled Binary
2. อัปโหลด `FireAlarm.ino.bin` ไปที่ `docs/firmware/` ใน GitHub
3. แก้ `docs/firmware/firmware.json` ให้ `"version"` ตรงกับ `FW_VERSION`
4. รอประมาณ 5 นาที บอร์ดจะอัปเดตเองภายใน 1 ชั่วโมง หรือกดปุ่ม "สั่งตรวจอัปเดต" ในหน้าเว็บ (ใช้รหัส admin)

ความปลอดภัย:
- บอร์ดย้อนกลับเวอร์ชันเดิมเอง ถ้าเวอร์ชันใหม่ต่อ EMQX ไม่ได้ภายใน 3 รอบรีสตาร์ต และจะไม่ติดตั้งเวอร์ชันนั้นซ้ำ
- ใครแก้ไฟล์ใน repo ได้ = สั่งอัปเดตบอร์ดได้ ต้องเปิด 2FA ในบัญชี GitHub
- ไฟล์ .bin ต้องคอมไพล์ตอน `secrets.h` ว่างเสมอ

## ดูระดับเสียงสดในหน้าเว็บ (ตั้งสิทธิ์ใน EMQX ครั้งเดียว)

หน้าเว็บขอให้ตัวส่งส่งระดับเสียงทุก 1 วินาทีครั้งละ 2 นาที (ต่ออายุเองทุก 1 นาทีขณะเปิดหน้า) จึงต้องให้ผู้ใช้ `dashboard` ส่งข้อความได้ 1 topic:

EMQX → Access Control → Authorization → แท็บ Username → แก้ `dashboard`
1. เพิ่มกฎ: Topic `bwfa/cmd/live` · Action **Publish** · Permission **Allow**
2. กฎนี้ต้องอยู่ **ก่อน** กฎ `#` Publish Deny (ถ้าเรียงลำดับไม่ได้ ให้ลบกฎ Deny แล้วเพิ่มกลับเข้าไปใหม่ต่อท้าย)

## Wi-Fi ที่มีหน้า login (captive portal)

ESP32 กรอกหน้า login ของ Wi-Fi ไม่ได้ ให้ IT เพิ่ม **MAC address** ของบอร์ดในรายการยกเว้น (MAC bypass / whitelist)
- ดู MAC ได้ที่: หน้าตั้งค่า (192.168.4.1) ด้านบนสุด · หน้าสถานะของบอร์ด · Serial Monitor ตอนเปิดเครื่อง (`[boot] MAC (Wi-Fi)`)
- MAC ของ ESP32 คงที่ ไม่สุ่มเปลี่ยน
- บอร์ดต้องออกเน็ตได้ที่: EMQX `bd343e82.ala.asia-southeast1.emqxsl.com` พอร์ต 8883 · GitHub Pages พอร์ต 443 · NTP พอร์ต 123
- ถ้ายังติดหน้า login หน้าสถานะของบอร์ดจะขึ้น "⚠️ ออกเน็ตไม่ได้" พร้อม MAC ที่ต้องแจ้ง
