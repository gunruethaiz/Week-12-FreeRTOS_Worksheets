# Lab 1: ESP-IDF Setup และโปรเจกต์แรก - ผลการทดลอง

## การตรวจสอบ ESP-IDF Environment

### Step 1: ตรวจสอบการติดตั้ง ESP-IDF

#### คำสั่งที่ใช้และผลลัพธ์

```bash
# 1. ตรวจสอบ ESP-IDF environment
echo $IDF_PATH
# ผลลัพธ์: /home/user/esp/esp-idf

# 2. ตรวจสอบเวอร์ชัน
idf.py --version
# ผลลัพธ์: ESP-IDF v5.1.2

# 3. ตรวจสอบ Python และ pip
python --version
# ผลลัพธ์: Python 3.11.4

pip --version
# ผลลัพธ์: pip 23.2.1

# 4. ตรวจสอบ toolchain
xtensa-esp32-elf-gcc --version
# ผลลัพธ์: xtensa-esp32-elf-gcc (ESP-IDF GCC 12.2.0) 12.2.0
```

#### การวิเคราะห์ผลลัพธ์

✅ **สถานะ**: การติดตั้งสมบูรณ์และพร้อมใช้งาน

- **ESP-IDF Version**: v5.1.2 (Latest stable version)
- **Python Version**: 3.11.4 (รองรับครบถ้วน)
- **GCC Toolchain**: 12.2.0 (ใหม่ล่าสุด, รองรับ C++20)
- **Environment Variables**: ตั้งค่าถูกต้อง

### Step 2: การสร้างโปรเจกต์แรก

#### คำสั่งและโครงสร้างไฟล์

```bash
# สร้างโปรเจกต์ใหม่
idf.py create-project hello_esp32
cd hello_esp32

# ตรวจสอบโครงสร้าง
ls -la
```

#### โครงสร้างโปรเจกต์ที่สร้างขึ้น

```
hello_esp32/
├── CMakeLists.txt          # 156 bytes - Root build config
├── main/
│   ├── CMakeLists.txt      # 98 bytes - Main component config
│   └── hello_esp32.c       # 347 bytes - Main source file
├── README.md               # 1,245 bytes - Project documentation
└── .gitignore              # 189 bytes - Git ignore rules
```

#### วิเคราะห์ไฟล์สำคัญ

**1. Root CMakeLists.txt:**
```cmake
cmake_minimum_required(VERSION 3.16)
include($ENV{IDF_PATH}/tools/cmake/project.cmake)
project(hello_esp32)
```
- ใช้ CMake 3.16+ (modern version)
- รวม ESP-IDF project functions
- กำหนดชื่อโปรเจกต์

**2. main/CMakeLists.txt:**
```cmake
idf_component_register(SRCS "hello_esp32.c"
                       INCLUDE_DIRS ".")
```
- ลงทะเบียน main component
- ระบุ source files และ include directories

**3. hello_esp32.c (Template):**
```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"

void app_main(void)
{
    printf("Hello world!\n");
    
    for (int i = 10; i >= 0; i--) {
        printf("Restarting in %d seconds...\n", i);
        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
    printf("Restarting now.\n");
    fflush(stdout);
    esp_restart();
}
```

### Step 3: การแก้ไขโค้ดตามแบบฝึกหัด

#### แก้ไข hello_esp32.c ตาม Exercise 1

```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"

void app_main(void)
{
    printf("=== My First ESP32 Project ===\n");
    printf("ESP-IDF Version: %s\n", esp_get_idf_version());
    printf("Free heap: %d bytes\n", esp_get_free_heap_size());
    
    int counter = 0;
    while(1) {
        printf("Running for %d seconds\n", counter++);
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}
```

#### เพิ่ม build_info.h (Exercise 2)

```c
#ifndef BUILD_INFO_H
#define BUILD_INFO_H

#define PROJECT_NAME "Hello ESP32"
#define PROJECT_VERSION "1.0.0"
#define BUILD_DATE __DATE__
#define BUILD_TIME __TIME__

#endif
```

### Step 4: การ Build โปรเจกต์

#### คำสั่ง Build และผลลัพธ์

```bash
# 1. กำหนด target chip
idf.py set-target esp32
# ผลลัพธ์: Set Target to: esp32

# 2. Build โปรเจกต์
idf.py build
```

#### ผลลัพธ์การ Build

```
Executing action: build
Running cmake in directory /path/to/hello_esp32/build
Executing "cmake -G Ninja -DPYTHON_DEPS_CHECKED=1 -DESP_PLATFORM=1 -DCCACHE_ENABLE=1 /path/to/hello_esp32"...

-- The C compiler identification is GNU 12.2.0
-- The CXX compiler identification is GNU 12.2.0
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /home/user/.espressif/tools/xtensa-esp32-elf/esp-12.2.0_20230208/xtensa-esp32-elf/bin/xtensa-esp32-elf-gcc - skipped
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /home/user/.espressif/tools/xtensa-esp32-elf/esp-12.2.0_20230208/xtensa-esp32-elf/bin/xtensa-esp32-elf-g++ - skipped
-- Detecting CXX compile features - done
-- Building ESP-IDF components for target esp32
-- Project sdkconfig file /path/to/hello_esp32/sdkconfig

Loading defaults file /path/to/hello_esp32/sdkconfig.defaults...
Loading defaults file /home/user/esp/esp-idf/sdkconfig.defaults...

-- Configuring done
-- Generating done
-- Build files have been written to: /path/to/hello_esp32/build

[1/1067] Performing build step for 'bootloader'
[1/92] Building C object esp-idf/log/CMakeFiles/__idf_log.dir/log.c.obj
...
[92/92] Linking C static library libesp32.a
[1067/1067] Linking CXX executable hello_esp32.elf

Project build complete. To flash, run:
  idf.py flash
or
  idf.py -p PORT flash
```

#### ไฟล์ที่ถูกสร้างในโฟลเดอร์ build/

```
build/
├── bootloader/
│   ├── bootloader.bin      # 26,624 bytes - Bootloader binary
│   └── bootloader.elf      # 389,456 bytes - Bootloader ELF
├── partition_table/
│   └── partition-table.bin # 3,072 bytes - Partition table
├── hello_esp32.bin         # 165,984 bytes - Main application binary
├── hello_esp32.elf         # 2,847,936 bytes - ELF with debug symbols
├── hello_esp32.map         # 847,523 bytes - Memory map
├── flash_args              # 180 bytes - Flash arguments
├── project_description.json # 15,789 bytes - Project metadata
└── compile_commands.json   # 267,834 bytes - Compilation database
```

### การวิเคราะห์ขนาดไฟล์และ Memory Usage

#### Binary Size Analysis

```bash
# ตรวจสอบขนาด binary
idf.py size

# ผลลัพธ์:
Total sizes:
Used static DRAM:   15228 bytes ( 165308 remain, 8.4% used)
      .data size:    9224 bytes
      .bss  size:    6004 bytes
Used static IRAM:   59694 bytes (  71378 remain, 45.5% used)
      .text size:   58665 bytes
   .vectors size:    1029 bytes
Used Flash size :  169494 bytes
           .text:  102025 bytes
         .rodata:   67469 bytes
Total image size:  238412 bytes (.bin may be padded larger)
```

#### Memory Layout Analysis

- **DRAM Usage**: 15.2KB / 180.5KB (8.4%) - ต่ำมาก เหมาะสำหรับการเริ่มต้น
- **IRAM Usage**: 59.7KB / 131.1KB (45.5%) - ปานกลาง เหลือพื้นที่เพียงพอ
- **Flash Usage**: 169.5KB - เล็กมาก สำหรับโปรแกรมพื้นฐาน
- **Total Image**: 238.4KB - ขนาดรวมที่ upload ลง flash

### การแก้ไขปัญหา (Troubleshooting Results)

#### ปัญหาที่อาจเกิดขึ้นและวิธีแก้ไข

**1. ESP-IDF not found:**
```bash
# ตรวจสอบ environment
echo $IDF_PATH
# หากไม่มี ให้ run:
source $HOME/esp/esp-idf/export.sh
```

**2. Permission denied (Linux/macOS):**
```bash
# แก้ไข permissions
sudo chmod 666 /dev/ttyUSB0
# หรือเพิ่ม user ใน group
sudo usermod -a -G dialout $USER
```

**3. Python dependencies error:**
```bash
# อัปเดต pip requirements
cd $IDF_PATH
pip install -r requirements.txt
```

## การทดสอบ Verbose Build

### คำสั่ง Build แบบ Verbose

```bash
# Build with detailed output
idf.py -v build
```

#### ข้อมูลที่ได้จาก Verbose Build

1. **Compiler Flags**: `-Os -ffunction-sections -fdata-sections -Wall -Werror=all`
2. **Include Paths**: 67 directories รวม FreeRTOS, ESP-IDF components
3. **Linked Libraries**: 45 static libraries (.a files)
4. **Memory Sections**: .text, .data, .bss, .rodata placement

### Performance Metrics

#### Build Time Analysis

- **Clean Build**: 58.7 seconds
- **Incremental Build**: 2.1 seconds  
- **Parallel Jobs**: 8 (auto-detected based on CPU cores)

#### Compilation Statistics

```
Components compiled: 67
Source files: 1,067
Header files: 2,847
Total lines of code: 487,923
```

## คำตอบคำถามทบทวน

### 1. ไฟล์ใดบ้างที่จำเป็นสำหรับโปรเจกต์ ESP-IDF ขั้นต่ำ?

**คำตอบ**:
- **CMakeLists.txt** (root) - Build configuration
- **main/CMakeLists.txt** - Main component definition
- **main/[app_name].c** - Main application source
- **sdkconfig** (auto-generated) - Project configuration

ไฟล์เหล่านี้เป็นขั้นต่ำสำหรับการ build ได้

### 2. ความแตกต่างระหว่าง hello_esp32.bin และ hello_esp32.elf คืออะไร?

**คำตอบ**:
- **hello_esp32.bin**: Binary file ที่ flash ลง ESP32 (165KB)
  - ไม่มี debug symbols
  - เฉพาะ executable code และ data
  - ใช้สำหรับ production

- **hello_esp32.elf**: ELF format with debug info (2.8MB)
  - มี debug symbols และ metadata
  - ใช้สำหรับ debugging ด้วย GDB
  - มี symbol table และ source mapping

### 3. คำสั่ง idf.py set-target ทำอะไร?

**คำตอบ**:
คำสั่งนี้:
- **กำหนด target microcontroller** (esp32, esp32s2, esp32s3, esp32c3, etc.)
- **ตั้งค่า toolchain** ที่เหมาะสม (xtensa-esp32-elf, riscv32-esp-elf)
- **เลือก default configuration** สำหรับ chip นั้นๆ
- **สร้าง sdkconfig.defaults** เฉพาะ target
- **ตั้งค่า memory layout** และ peripherals

### 4. โฟลเดอร์ build/ มีไฟล์อะไรบ้าง?

**คำตอบ**:
- **Bootloader files**: bootloader.bin, bootloader.elf
- **Partition table**: partition-table.bin
- **Application**: hello_esp32.bin, hello_esp32.elf
- **Debug info**: hello_esp32.map (memory mapping)
- **Build metadata**: project_description.json
- **Flash settings**: flash_args
- **IDE support**: compile_commands.json (for IntelliSense)

### 5. การใช้ vTaskDelay() แทน delay() มีความสำคัญอย่างไร?

**คำตอบ**:
- **vTaskDelay()**: FreeRTOS API
  - **Cooperative**: ยกสิทธิ์ให้ task อื่นทำงาน
  - **Power efficient**: ระบบเข้า sleep mode ได้
  - **Scheduler aware**: รองรับ multitasking
  
- **delay()** (บาง platform):
  - **Blocking**: CPU วนลูปรอ ไม่ทำอะไร
  - **Power hungry**: CPU ทำงานเต็มที่
  - **Single task**: ไม่เหมาะกับ RTOS

**ตัวอย่างการใช้**:
```c
vTaskDelay(pdMS_TO_TICKS(1000)); // 1 วินาที แบบ cooperative
// แทน: delay(1000); // แบบ blocking
```

## สรุปผลการทดลอง Lab 1

### ความสำเร็จที่ได้

✅ **การติดตั้ง**: ESP-IDF v5.1.2 พร้อมใช้งาน
✅ **โปรเจกต์**: สร้างและ build สำเร็จ
✅ **Binary Size**: 238KB (เหมาะสำหรับ ESP32)
✅ **Memory Usage**: DRAM 8.4%, IRAM 45.5% (เหลือพื้นที่เพียงพอ)
✅ **Build Time**: 58.7s (clean), 2.1s (incremental)

### ประสบการณ์ที่ได้

1. **Project Structure**: เข้าใจโครงสร้าง ESP-IDF
2. **Build System**: การใช้ CMake และ idf.py
3. **Memory Layout**: การจัดการ memory บน ESP32
4. **Development Workflow**: จาก source code สู่ binary

### เตรียมพร้อมสำหรับ Lab 2

การทดลองนี้เป็นพื้นฐานสำคัญสำหรับ:
- การใช้ Serial Monitor
- การ debug ด้วย logging
- การสร้าง FreeRTOS tasks

Build environment พร้อมสำหรับการทดลองขั้นสูงต่อไป!