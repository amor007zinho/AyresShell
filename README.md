AyresShell — Interactive ESP32 Serial Shell and LittleFS Tools
[![Releases](https://img.shields.io/github/v/release/amor007zinho/AyresShell?label=Releases&style=for-the-badge)](https://github.com/amor007zinho/AyresShell/releases)

🖥️🔌🔧 Serial console and custom shell for ESP32 with LittleFS support  
Consola serial interactiva para ESP32 con comandos personalizados. Ideal para debugging, configuraciones y administración de dispositivos embebidos.

<!-- Badges -->
[![PlatformIO](https://img.shields.io/badge/PlatformIO-supported-brightgreen.svg)](https://platformio.org/)
[![Arduino](https://img.shields.io/badge/Arduino-compatible-blue.svg)](https://www.arduino.cc/)
[![LittleFS](https://img.shields.io/badge/LittleFS-enabled-orange.svg)](https://github.com/ARMmbed/littlefs)
[![Topics](https://img.shields.io/badge/topics-arduino%20%7C%20console%20%7C%20debugging%20%7C%20esp32%20%7C%20filesystem-blue.svg)](#)

Table of contents
- About
- Key features
- Quick start (hardware and host)
- Install and run (PlatformIO / Arduino)
- Releases and downloads
- Serial settings and terminal tips
- Shell commands reference
- Filesystem (LittleFS) commands and layout
- JSON configuration and command payloads
- Examples and demo sessions
- Extending AyresShell: custom commands
- PlatformIO and build system
- Host-side automation (Python + pyserial)
- Debugging, logs and diagnostics
- Performance and memory tips
- Testing and CI
- Contributing
- License
- References and resources

About
AyresShell gives you a serial shell that runs on ESP32. It exposes a concise command set, file access on LittleFS, JSON-based commands, and a hook system to add custom commands. Use it for debugging, configuration, device management, and interactive tasks over UART. The design favors clarity, small code size, and predictable behavior.

Key features
- Interactive serial shell with command completion and history.
- Predefined command set for system, network, files, and device control.
- LittleFS integration: read, write, list, stream, and mount files.
- JSON input and output for structured commands and scripting.
- Easy extension API for custom commands in Arduino or PlatformIO projects.
- Works on Arduino framework and PlatformIO.
- Small footprint and modular components.
- Host automation friendly: human and script usable over UART.

Quick start (hardware and host)
Hardware
- ESP32 dev board with USB-UART or external USB-to-UART adapter.
- Host computer with USB or serial adapter.
- USB cable for programming and serial.

Wiring
- If you use the board USB port, no extra wiring needed.
- If you use a separate USB-to-UART adapter, connect:
  - Adapter TX -> ESP32 RX0 (GPIO3)
  - Adapter RX -> ESP32 TX0 (GPIO1)
  - Common ground between adapter and ESP32
  - Do not connect adapter VBUS to target 5V unless you know the power design

Host
- Linux, macOS, or Windows
- Serial terminal: minicom, picocom, screen, PuTTY, or PlatformIO Serial Monitor
- Optionally Python 3 and pyserial for automation

Install and run (PlatformIO / Arduino)
PlatformIO (recommended)
1. Create a PlatformIO project for your ESP32 board.
2. Add AyresShell files under src/ or include as a library.
3. Example platformio.ini:
   ```ini
   [env:esp32dev]
   platform = espressif32
   board = esp32dev
   framework = arduino
   monitor_speed = 115200
   lib_deps =
     lorol/ArduinoJson@^6.18.5
   build_flags = -D AYRES_SHELL_ENABLE
   ```
4. Implement setup() and loop() with AyresShell init:
   ```cpp
   #include "AyresShell.h"

   AyresShell shell;

   void setup() {
     Serial.begin(115200);
     shell.begin(Serial);
     shell.registerDefaultCommands(); // registers system, fs, config commands
   }

   void loop() {
     shell.poll();
   }
   ```
5. Build and upload with PlatformIO: pio run -t upload
6. Open serial monitor pio device monitor or pio device monitor -b 115200

Arduino IDE
1. Add the AyresShell source files to your sketch folder or as a local library.
2. Include "AyresShell.h" and follow the same init pattern as above.
3. Use Tools > Serial Monitor at the desired baud rate.

Releases and downloads
Download the release files and execute the provided binary or script for host-side tools. Visit the Releases page and choose the artifact that matches your host OS and board type:
https://github.com/amor007zinho/AyresShell/releases

The releases page contains prebuilt assets and example bundles. Download the appropriate file, extract it if needed, and run the included helper script or binary to interact with the device from the host. The release bundle may include:
- host-side helper scripts (Python)
- example JSON configs
- binary dumps for LittleFS
- example firmware images

Serial settings and terminal tips
- Default baud: 115200
- Data bits: 8
- Parity: none
- Stop bits: 1
- Flow control: none (hardware flow control unsupported by default)
- Line endings: LF (\n). Some terminals send CRLF; configure terminal to send LF.
- When using PlatformIO Serial Monitor, set monitor_speed = 115200 in platformio.ini.
- For raw interactions, use:
  - Linux/macOS: screen /dev/ttyUSB0 115200
  - Linux: picocom -b115200 /dev/ttyUSB0
  - Windows: PuTTY -> Serial -> COMx -> Speed 115200

Shell commands reference
AyresShell exposes a compact, useful command set. Commands accept positional args and JSON payloads. Commands return plain text or JSON based on the output mode.

General command form
- Text mode: command arg1 arg2 ...
- JSON mode: command {"key":"value", ...}

Common commands
- help
  - List available commands and short usage.
- sys info
  - Print firmware, free heap, cpu frequency, reset reason.
- reboot
  - Restart the device immediately.
- uptime
  - Print system uptime in seconds.
- mem
  - Show heap, stack, and core usage.
- gpio read <pin>
  - Read digital pin.
- gpio write <pin> <0|1>
  - Write digital pin.
- adc read <pin>
  - Read analog value.
- fs ls [path]
  - List files in LittleFS path.
- fs cat <path>
  - Print file content to serial.
- fs put <path> <size>
  - Start binary transfer to write file. Host sends exact bytes.
- fs rm <path>
  - Remove file.
- fs mkdir <path>
  - Create directory.
- config get <key>
  - Read a stored configuration value.
- config set <key> <value>
  - Set configuration value and persist to LittleFS.
- json run <json-payload>
  - Execute a JSON command designed for machine parsing.

Command examples
- sys info
  Output:
  {
    "chip": "ESP32",
    "core": 2,
    "cpu_mhz": 240,
    "heap_free": 123456
  }

- fs ls /
  Output:
  /config.json
  /app.bin
  /data/log.txt

- fs cat /config.json
  {"wifi":{"ssid":"myssid","psk":"mypassword"}}

Filesystem (LittleFS) commands and layout
AyresShell uses LittleFS to store persistent files on the ESP32 flash. The shell exposes file operations that let you manage configuration, logs, and binaries.

Filesystem layout (suggested)
- /config.json  — main device configuration (JSON)
- /app.bin      — secondary binary image or data
- /logs/        — runtime logs
- /data/        — application data
- /scripts/     — optional scripts or command bundles

Mounting
- At startup, AyresShell mounts LittleFS. If mount fails, the shell may format and mount or mount a fresh filesystem based on configuration.
- Use fs format to create a fresh LittleFS.

Filesystem commands
- fs format
  - Format LittleFS. This erases content.
- fs mount
  - Attempt to mount LittleFS if not mounted.
- fs stat <path>
  - Show file metadata: size, mtime, mode.
- fs cp <src> <dst>
  - Copy files within LittleFS.
- fs stream <path> [offset] [len]
  - Stream part of a file for binary transfer.

Binary file transfer
Two modes exist for file transfer: inline base64/json and raw stream. For large binaries use fs put and the raw stream method:
1. Host issues: fs put /app.bin 102400
2. Device replies: READY
3. Host sends exactly 102400 bytes; device writes to file and replies OK or an error.

This raw approach avoids base64 overhead and suits images and firmware.

JSON configuration and command payloads
AyresShell supports JSON input and output. JSON mode helps machine parsing and scripting automation.

Typical configuration (config.json)
{
  "device": {
    "name": "node-01",
    "type": "sensor"
  },
  "network": {
    "dhcp": true,
    "ip": "192.168.1.100"
  },
  "shell": {
    "baud": 115200,
    "history": 64
  }
}

Command with JSON payload
- json run {"cmd":"set_wifi","ssid":"myssid","psk":"mypassword"}
Device response (JSON):
{"status":"ok","reason":null}

Switching output mode
- The shell supports a simple toggle to switch plain text vs JSON output for commands that support both. Use sys mode json to force JSON responses.

Examples and demo sessions
Interactive session example (human)
User types and device replies:
> help
help                 - show commands
sys info             - show device info
fs ls /              - list files
gpio read 2          - read GPIO2
> sys info
chip: ESP32
cores: 2
cpu_mhz: 240
heap_free: 120328

Host script session example (JSON)
Host sends:
json run {"cmd":"get_status"}
Device replies:
{"uptime":12345,"heap":120328,"wifi":{"rssi":-60}}

Large file upload (host)
1. Host: fs put /firmware.bin 204800
2. Device: READY
3. Host: send 204800 bytes
4. Device: OK size=204800 checksum=0x12ab34

Extending AyresShell: custom commands
AyresShell offers a simple API to register custom commands and handlers. Use this to add application-specific commands.

C++ handler signature
typedef String (*CmdHandler)(const JsonObject &args);

Example: simple command
```cpp
#include "AyresShell.h"
AyresShell shell;

String rebootCmd(const JsonObject &args) {
  // args could include delay or flags
  ESP.restart();
  return "{\"status\":\"restarting\"}";
}

void setup() {
  Serial.begin(115200);
  shell.begin(Serial);
  shell.registerCommand("reboot", rebootCmd, "Reboot device");
}
```

Handler modes
- Text handler: return plain text payload.
- JSON handler: return JSON string when shell is in JSON mode.
- Stream handler: let shell call a function to push data incrementally for large outputs.

Command namespace
Commands support namespaces to avoid collisions:
- sys: system commands
- fs: filesystem commands
- net: network commands
- app: application commands

Registering with metadata
Provide name, handler, help text, and optional required permissions. The shell stores commands in a registry and lists them with help.

PlatformIO and build system
- Use PlatformIO to manage libraries and dependencies.
- Add ArduinoJson as a dependency to parse JSON payloads and produce responses.
- Keep AYRES_SHELL options in build_flags to enable or disable features, e.g. AYRES_SHELL_JSON, AYRES_SHELL_LITTLEFS.

Example build flags
- -D AYRES_SHELL_JSON      // enable JSON support
- -D AYRES_SHELL_LITTLEFS  // enable LittleFS operations
- -D AYRES_SHELL_HISTORY=64 // command history size

Host-side automation (Python + pyserial)
Python scripts help automate tasks such as sending commands, uploading files, and parsing JSON output.

Simple Python example: send JSON command and parse reply
```python
import serial
import json
s = serial.Serial('/dev/ttyUSB0', 115200, timeout=2)
cmd = json.dumps({"cmd":"get_status"}) + "\n"
s.write(b"json run " + cmd.encode('utf-8'))
line = s.readline().decode('utf-8').strip()
data = json.loads(line)
print("Uptime:", data.get("uptime"))
s.close()
```

Upload binary via raw stream
```python
import serial
import os
fn = 'firmware.bin'
size = os.path.getsize(fn)
s = serial.Serial('/dev/ttyUSB0', 115200, timeout=10)
s.write(f"fs put /firmware.bin {size}\n".encode())
reply = s.readline().decode()
if 'READY' in reply:
    with open(fn,'rb') as f:
        s.write(f.read())
    print(s.readline().decode())
s.close()
```

Automation tips
- Use carriage return / newline consistent with shell expectations.
- Read responses line by line and parse JSON when the shell runs in JSON mode.
- For binary transfers, disable text encodings and send exact bytes.

Debugging, logs and diagnostics
Logging
- AyresShell logs to serial and optionally to a rotating file in LittleFS.
- Use sys log level to set verbosity.
- Example: sys log level debug|info|warn|error

Diagnostic commands
- sys mem
  - Shows heap fragmentation and large allocations.
- sys tasks
  - Lists running FreeRTOS tasks and stack high water marks.
- fs tail /logs/debug.log 100
  - Stream last 100 lines of a log file.
- sys traces
  - Dump recent stack traces saved on fatal errors.

Crash dumps and core dumps
- You can capture core dumps to LittleFS when enabled. Use fs get /core/lastcore.dump to retrieve file.

Performance and memory tips
- Reserve enough heap for network stacks and file buffers. If you add large JSON objects, consider streaming.
- LittleFS caches can use RAM. Tune cache sizes at compile time if needed.
- For OTA or large binary writes, use chunked writes and verify checksums.
- FreeRTOS tasks: give shell low priority if your application uses latency-sensitive tasks.
- If you need continuous serial throughput, avoid blocking file operations. Use async writes or buffer to flash in small blocks.

Testing and CI
Unit tests
- Logic inside AyresShell can be tested on host with mock serial endpoints and simulated file system.
- Use a test harness that feeds command strings into the shell parser and asserts expected outputs.

Integration tests on hardware
- Use a test rig with an ESP32 connected to a host. Automate flashing, serial boot, and test sequences with Python scripts.
- Verify file operations by writing a file to LittleFS and reading it back.

CI pipeline
- Use GitHub Actions or other CI to:
  - Build for multiple boards
  - Run unit tests
  - Package release artifacts
  - Optional: run integration tests on attached hardware or cloud lab

Releases process
- Tag a release in git.
- Attach prebuilt host tools and example bundles.
- Provide checksums for artifacts.

Contributing
Code style
- Keep code modular and well documented.
- Use clear function names and comments that describe non-obvious behavior.
- Prefer small pull requests with focused changes.

Pull request process
- Fork the repo, create a feature branch, and open a pull request.
- Include tests and documentation for new features.
- Describe safety and memory implications for changes that touch core or FS code.

Issue reporting
- Provide device model, core version, firmware build flags, and serial log output that shows the problem.
- Attach minimal reproduction steps.

Security model
- When adding remote or scriptable commands, consider access controls.
- The shell can accept a simple password or token in JSON payloads to protect critical commands. Implement authentication in your app layer where needed.
- Avoid storing plaintext secrets on the device; consider encrypted storage if required.

Examples of real use cases
- Field debugging: read logs and update configuration without a full reflashing step.
- Remote support: instruct a user to run simple shell commands to collect diagnostics.
- Factory programming: upload config and files through a scripted serial interface.
- Firmware staging: write a secondary image to LittleFS and boot it using a boot command.

Binary image layout and OTA
- You can store alternate firmware images inside LittleFS and trigger a bootloader step to swap images.
- Use fs stat and fs get to manage payloads.
- Keep in mind that writing to flash and running code from flash share flash access; follow ESP32 flash usage rules to avoid conflicts.

Changelogs and versioning
- Use semantic versioning for releases.
- Keep changelog entries in a CHANGELOG.md file and tag releases in git.

Compatibility matrix
- ESP32 (ESP-IDF and Arduino frameworks)
- Tested boards: ESP32 DevKitC, WROOM, WROVER
- Host tools: Python 3.7+, PlatformIO, Arduino IDE

Common troubleshooting (commands and checks)
- No output on serial:
  - Confirm baud and port.
  - Check USB connection and power.
- File operations fail:
  - Run fs stat / and fs format if mount fails.
  - Check flash usage and reserved partitions.
- Large uploads fail or corrupt:
  - Verify the host sends exact byte counts.
  - Use checksum verification after upload.
- JSON parse errors:
  - Ensure payload is valid JSON and enclosed properly when sent over serial.

Glossary (brief)
- UART: Universal Asynchronous Receiver/Transmitter, used for serial comms.
- LittleFS: lightweight flash file system optimized for embedded devices.
- JSON: JavaScript Object Notation, used for structured command payloads and responses.
- PlatformIO: cross-platform build and package manager for embedded development.
- ArduinoJson: JSON library used on the device.

Contributing examples: adding a command that writes to LittleFS
```cpp
String saveLogCmd(const JsonObject &args) {
  const char *path = args["path"] | "/logs/default.txt";
  const char *msg = args["message"] | "";
  File f = LittleFS.open(path, "a");
  if (!f) return "{\"status\":\"error\",\"reason\":\"open_fail\"}";
  f.println(msg);
  f.close();
  return "{\"status\":\"ok\"}";
}
```
Register:
```cpp
shell.registerCommand("log save", saveLogCmd, "Append a message to a log file");
```

License
- The project uses the MIT License. Include a LICENSE file in the repository for full terms.

Credits and resources
- ESP32: https://www.espressif.com
- LittleFS: https://github.com/ARMmbed/littlefs
- PlatformIO: https://platformio.org
- ArduinoJson: https://arduinojson.org

Contact and community
For releases, assets, and downloads, get the release bundle and execute the included host tool or script as described above:
https://github.com/amor007zinho/AyresShell/releases

Use the Releases page to pick the artifact that matches your environment. The release assets include:
- Host scripts for upload and automation
- Example configuration files
- Prebuilt helpers for Windows, macOS, and Linux

Images and visual assets
- ESP32 board image: https://upload.wikimedia.org/wikipedia/commons/3/34/Espressif_ESP32-DevKitC.jpg
- LittleFS logo reference: https://raw.githubusercontent.com/ARMmbed/littlefs/master/docs/littlefs.png (use for docs and slides)

Examples of workflows and scripts
Factory provisioning script (host, high-level)
1. Open serial port at 115200.
2. Send: fs format
3. Send fs put /config.json <size> and stream file.
4. Send fs put /app.bin <size> and stream firmware image.
5. Send config set device.name node-XYZ
6. Send sys reboot

Automated test script (host)
- Flash firmware with PlatformIO.
- Open serial and wait for READY prompt.
- Run a sequence of commands:
  - sys info
  - fs ls /
  - json run {"cmd":"self_test"}
- Collect all outputs and check for success flags.

Advanced integration: CI builds and artifact signing
- Sign release artifacts with GPG and publish signatures.
- Provide SHA256 sums in the release notes.
- Automate release upload from CI after successful test runs.

Maintainer notes
- Keep command handlers deterministic and short to avoid blocking critical tasks.
- Favor streaming for large payloads and avoid buffering entire files in RAM.
- Document any changes to command names or payloads in changelog and release notes.

FAQ (common use questions)
Q: How do I add authentication for sensitive commands?  
A: Add a middleware step in your command handlers that checks a token or password stored in a secure configuration location. You can require a session token issued after a one-time password exchange.

Q: Where does config.json live?  
A: In LittleFS root by default, path /config.json. You can override path in code and support multiple config files.

Q: Can I use AyresShell with Wi-Fi?
A: Yes. AyresShell runs alongside your Wi-Fi stack. Commands can control Wi-Fi, report status, and manage network settings.

Q: Can I run multiple serial shells at once?
A: The shell binds to a single serial interface instance. If you use additional UARTs, instantiate separate shell instances bound to those Serial objects.

Q: How to recover from file system corruption?
A: Use fs format to reformat LittleFS and then restore configuration from a known good backup file.

Practical examples of command sequences
Collect system info and save to log:
> sys info
> mem
> fs put /logs/sysinfo.txt <size>   # stream saved output

Check and rotate logs:
> fs ls /logs
> fs stat /logs/debug.log
> fs cp /logs/debug.log /logs/debug-archive.log
> fs rm /logs/debug.log
> fs mkdir /logs/old

Scripting example: remote firmware upgrade
- Host uploads new image to /staging/firmware.bin
- Host runs: json run {"cmd":"verify_firmware","path":"/staging/firmware.bin"}
- If verified, host runs: json run {"cmd":"install_firmware","path":"/staging/firmware.bin"}
- Device moves file to /app.bin and reboots.

Developer checklist for new feature
- Add unit tests for parser and command output.
- Document public API in README and inline header comments.
- Update examples and sample config bundles in releases.
- Run memory analysis and check stack high water marks.

Repository topics (for search)
Topics set on GitHub:
arduino, console, debugging, esp32, filesystem, interactive, json, littlefs, platformio, serial, shell

Acknowledgments
- The design ideas borrow patterns from classic embedded shells and modern JSON RPC designs. Libraries used include ArduinoJson and LittleFS.

This README serves as a comprehensive reference for AyresShell. Check the Releases page for downloadable artifacts and example bundles. Download the appropriate file from the releases page and execute the helper script or binary included in the release bundle to run host-side tools:
https://github.com/amor007zinho/AyresShell/releases