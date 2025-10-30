# Lab 1: Single Task vs Multitasking - ผลการทดลอง

## การวิเคราะห์โค้ด Single Task System

### ลำดับการทำงาน Single Task:
```
เวลา: 0ms     -> เริ่มอ่าน sensor (LED1 ON)
เวลา: 500ms   -> LED1 OFF 
เวลา: 1000ms  -> เริ่ม processing (การคำนวณหนัก)
เวลา: ~2000ms -> เริ่มควบคุม actuator (LED2 ON)  
เวลา: 2300ms  -> LED2 OFF
เวลา: 2600ms  -> ตรวจสอบปุ่ม
เวลา: 2600ms  -> กลับไปรอบใหม่
```

**วงจรหนึ่งใช้เวลา**: ประมาณ 2.6 วินาที

### ปัญหาที่พบใน Single Task:
1. **การตอบสนองปุ่มล่าช้า**: ถ้ากดปุ่มขณะทำ processing จะต้องรอจนกว่าจะจบรอบนั้น
2. **ไม่มีการทำงานแบบ concurrent**: LED ทำงานเป็นลำดับ ไม่พร้อมกัน
3. **CPU utilization**: 100% แต่ไม่มีประสิทธิภาพ เพราะต้องรอ delay

## การวิเคราะห์โค้ด Multitasking System

### Task Priorities:
- Emergency Task: Priority 5 (สูงสุด)
- Sensor Task: Priority 2
- Actuator Task: Priority 2  
- Processing Task: Priority 1 (ต่ำสุด)

### การทำงานแบบ Multitasking:
```
เวลา: 0-10ms    -> Emergency task ตรวจสอบปุ่มทุก 10ms
เวลา: 0-100ms   -> Sensor task (LED1 ON)  
เวลา: 0-200ms   -> Actuator task (LED2 ON)
เวลา: 0-?ms     -> Processing task ทำงานเมื่อ CPU ว่าง
```

## ผลการทดลองที่คาดหวัง

### 1. เวลาตอบสนองปุ่ม (Button Response Time)

| ระบบ | เวลาตอบสนองเฉลี่ย | เวลาตอบสนองสูงสุด | สาเหตุ |
|------|-------------------|-------------------|-------|
| Single Task | 1.3 วินาที | 2.6 วินาที | ต้องรอให้รอบปัจจุบันเสร็จ |
| Multitasking | < 10ms | 20ms | Emergency task มี priority สูงสุด |

### 2. การทำงานของ LED

#### Single Task:
- LED1 และ LED2 ทำงานเป็นลำดับ ไม่เคยทำงานพร้อมกัน
- LED1: ON 500ms, OFF 500ms, รอ 1600ms
- LED2: รอ 1000ms, ON 300ms, OFF 300ms, รอ 1000ms

#### Multitasking:
- LED1: กะพริบทุกวินาที (ON 100ms, OFF 900ms)
- LED2: กะพริบทุกวินาที (ON 200ms, OFF 800ms)  
- ทำงานพร้อมกัน independent จากกัน

### 3. CPU Utilization

#### Single Task:
- Processing: ~60% (การคำนวณหนัก)
- Sensor: ~15% (LED + delay)
- Actuator: ~10% (LED + delay)  
- Idle: ~15% (รอระหว่าง task)

#### Multitasking:
- Processing: ~30-40% (ลด priority ทำให้ถูก preempt)
- Sensor: ~5% (เร็วขึ้นเพราะไม่ต้องรอ)
- Actuator: ~5% (เร็วขึ้นเพราะไม่ต้องรอ)
- Emergency: ~1% (check ทุก 10ms)
- Idle: ~50-60% (เพราะ task ทำงานแบบ concurrent)

## คำตอบคำถามการวิเคราะห์

### 1. ความแตกต่างในการตอบสนองปุ่มระหว่างทั้งสองระบบคืออะไร?

**คำตอบ**: 
- **Single Task**: การตอบสนองล่าช้ามาก (1-3 วินาที) เพราะต้องรอให้งานปัจจุบันเสร็จก่อน โดยเฉพาะขณะทำ processing ที่ใช้เวลานาน
- **Multitasking**: การตอบสนองเร็วมาก (< 10ms) เพราะ emergency task มี priority สูงสุด สามารถ preempt งานอื่นได้ทันที

### 2. ใน Single Task System งานไหนที่ทำให้การตอบสนองล่าช้า?

**คำตอบ**: 
งาน **Processing Task** ที่มีการคำนวณหนัก (loop 1,000,000 ครั้ง) เป็นงานที่ทำให้การตอบสนองล่าช้าที่สุด เพราะ:
- ใช้เวลานานที่สุดในการทำงาน (~1 วินาที)
- ไม่มีการ yield ให้งานอื่น
- ถ้ากดปุ่มขณะ processing จะต้องรอจนกว่าจะเสร็จ

### 3. ข้อดีของ Multitasking System ที่สังเกตได้คืออะไร?

**คำตอบ**:
1. **Real-time Response**: ตอบสนองเหตุการณ์ฉุกเฉินได้ทันที
2. **Concurrency**: งานหลายอย่างทำพร้อมกัน LED กะพริบ independent
3. **Priority Management**: งานสำคัญทำก่อน งานไม่สำคัญทำหลัง
4. **Better Resource Utilization**: CPU ไม่ waste เวลารอ delay
5. **Responsiveness**: ระบบตอบสนองดีขึ้นโดยรวม

### 4. มีข้อเสียของ Multitasking System ที่สังเกตได้หรือไม่?

**คำตอบ**:
1. **Context Switching Overhead**: มีเวลาสูญเสียจากการเปลี่ยน task
2. **Memory Overhead**: แต่ละ task ต้องมี stack แยกกัน (2048 bytes/task)
3. **Complexity**: โค้ดซับซ้อนขึ้น ยากต่อการ debug
4. **Race Conditions**: อาจเกิดปัญหาถ้า task แชร์ resource
5. **Timing Uncertainty**: เวลาทำงานแต่ละ task ไม่แน่นอน (depends on scheduler)

## สรุปผลการทดลอง

Multitasking แสดงให้เห็นถึงข้อดีที่ชัดเจนในการปรับปรุง:
- **การตอบสนอง**: เร็วขึ้น 100-250 เท่า
- **ประสิทธิภาพ**: ใช้ทรัพยากรดีขึ้น
- **ความยืดหยุ่น**: สามารถปรับ priority ได้

แต่ก็มีต้นทุนในด้าน complexity และ memory overhead ที่ต้องพิจารณา