ใส่ไฟล์ 4 ไฟล์นี้ไว้ในโฟลเดอร์นี้ (ชื่อไฟล์ต้องตรงทุกตัวอักษร)
  FireAlarm.ino.bootloader.bin   <- จาก Sketch > Export Compiled Binary (โฟลเดอร์ build)
  FireAlarm.ino.partitions.bin   <- จากโฟลเดอร์ build
  FireAlarm.ino.bin              <- จากโฟลเดอร์ build
  boot_app0.bin                  <- จาก .../Arduino15/packages/esp32/hardware/esp32/<version>/tools/partitions/
⚠️ คอมไพล์ตอน secrets.h เป็นค่าว่างเท่านั้น
⚠️ ไม่ต้องใช้ FireAlarm.ino.merged.bin (ไฟล์นั้นจะลบค่าตั้งค่าในบอร์ดทิ้ง)
