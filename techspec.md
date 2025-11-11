# 🎧 Local Network In-Ear Monitor + Talkback System (Hybrid Web Control)

Version: 1.1
Author: Michael Sebastian
Last Updated: November 2025

---

## 📘 Overview

Sistem **Local Network In-Ear Monitor (IEM) + Talkback** ini dirancang untuk memungkinkan *real-time monitoring* audio dari **digital mixer ke musisi/singer** melalui jaringan lokal (*Wi-Fi 6 atau LAN*), dengan **latency ultra rendah (<20 ms)**.

Selain itu, sistem ini mendukung **push-to-talk (PTT)** dua arah untuk komunikasi musisi ↔ sound engineer menggunakan mic di earphone.
Semua kontrol dan status sistem diakses melalui **web dashboard**, tanpa GUI desktop.

---

## 🧠 System Goals

* Latency audio total: **< 20 ms (monitor)**, **< 40 ms (talkback full-duplex)**
* Beroperasi **tanpa internet**, hanya pada **jaringan lokal (Wi-Fi 6 / LAN)**
* Menggunakan **Opus codec (2.5 ms frame)** untuk efisiensi dan kualitas tinggi
* Server berbasis **JUCE (C++)** untuk performa real-time
* Control panel berbasis **Flutter Web Dashboard**
* Client (IEM app) berbasis **Flutter (mobile)** dengan audio core native (C++/FFI)

---

## 🧩 Architecture

### High-Level Diagram

```
[Mixer / Audio Interface]
         │  (USB multichannel audio)
         ▼
┌──────────────────────────────┐
│ SERVER (JUCE C++ Headless)   │
│ • Audio capture & routing     │
│ • Stereo mix per client       │
│ • Opus encode/decode          │
│ • UDP stream / talkback       │
│ • WebSocket/HTTP control API  │
└──────────────┬───────────────┘
               │
         Local Network (Wi-Fi 6 / LAN)
               │
 ┌─────────────────────────────┐
 │ CLIENT (Flutter Mobile)     │
 │ • Receive stereo monitor     │
 │ • Decode Opus → playback     │
 │ • Push-to-Talk mic (Opus)    │
 │ • UDP stream bidirectional   │
 └─────────────────────────────┘
               │
 ┌─────────────────────────────┐
 │ WEB DASHBOARD (Flutter Web) │
 │ • Client status & latency    │
 │ • Start/stop, volume, config │
 │ • Soundcard selection        │
 │ • Talkback from engineer     │
 └─────────────────────────────┘
```

---

## ⚙️ Tech Stack Summary

| Layer               | Technology                      | Function                        |
| ------------------- | ------------------------------- | ------------------------------- |
| Audio Engine        | **JUCE (C++)**                  | Real-time audio I/O & routing   |
| Codec               | **libOpus**                     | Low-latency audio compression   |
| Transport           | **UDP (boost::asio)**           | Real-time packet delivery       |
| Signaling / Control | **WebSocket++ / HTTP JSON API** | Dashboard integration           |
| Dashboard           | **Flutter Web**                 | Control, monitor, configuration |
| Mobile Client       | **Flutter + C++ (FFI)**         | Audio playback + PTT talkback   |
| Build System        | **CMake + Flutter FFI**         | Cross-platform build            |
| Network             | **Wi-Fi 6 Router (5 GHz)**      | Low jitter, local only          |

---

## 🔊 Audio Pipeline

### Monitor (Server → Client)

1. Server ambil input dari mixer (multichannel USB)
2. Routing & mixing → stereo per user
3. Encode menggunakan **Opus (2.5 ms frame, 128 kbps)**
4. Kirim via UDP unicast ke masing-masing client
5. Client decode dan playback dengan buffer kecil (≤64 samples)

### Talkback (Client → Server)

1. Tekan tombol PTT → aktifkan mic
2. Capture mono input → encode Opus (64 kbps mono)
3. Kirim via UDP ke server
4. Server decode dan route ke sound engineer monitor
5. Latency full-duplex: ±30–40 ms

---

## 🧭 Network Topology

| Component     | Description                                   |
| ------------- | --------------------------------------------- |
| **Router**    | Wi-Fi 6 / LAN dedicated (tanpa internet)      |
| **Subnet**    | 192.168.10.x (static preferred)               |
| **QoS**       | Prioritize UDP audio traffic (port 5000–5010) |
| **Discovery** | UDP broadcast / mDNS                          |
| **Sync**      | Simple NTP or timestamp correction            |

---

## 🖥️ Server (JUCE C++ Headless)

### Core Modules

| Module                        | Function                             |
| ----------------------------- | ------------------------------------ |
| `AudioInputManager`           | Detect & open mixer (ASIO/CoreAudio) |
| `MixEngine`                   | Mix multichannel → stereo per user   |
| `OpusEncoder` / `OpusDecoder` | Encode/decode low-latency streams    |
| `UDPServer`                   | Manage UDP sockets for clients       |
| `TalkbackRouter`              | Route incoming mic streams           |
| `ClientRegistry`              | Store connected clients & config     |
| `WebSocketServer`             | Serve dashboard connections          |
| `HttpApi`                     | Simple REST API for control          |
| `ConfigManager`               | Load/save JSON config                |

---

## 🌐 Web Dashboard (Flutter Web)

### Features

* Device discovery & connection status
* Per-client latency, signal level, and mix assignment
* Audio interface selector
* Master control: start/stop stream
* Soundman talkback button
* Config persistence (local JSON or REST PUT)

### API Endpoints (Server)

| Endpoint        | Method   | Description                                 |
| --------------- | -------- | ------------------------------------------- |
| `/api/status`   | GET      | Get system & client status                  |
| `/api/clients`  | GET      | List connected clients                      |
| `/api/mix/{id}` | POST     | Set mix or routing                          |
| `/api/audio`    | GET/POST | Get/set soundcard config                    |
| `/api/control`  | POST     | Start/Stop server, talkback                 |
| `/ws`           | WS       | Realtime events (client join/leave, meters) |

---

## 📱 Client (Flutter Mobile + Native Audio Core)

### Responsibilities

* Terima UDP stream stereo dari server
* Decode Opus → playback dengan buffer kecil
* Kirim talkback (mono) via UDP saat PTT aktif
* Auto-discover server via UDP broadcast
* UI sederhana: volume slider, status, PTT button

### FFI Bridge

| Function                | Purpose                     |
| ----------------------- | --------------------------- |
| `initAudioEngine()`     | Initialize native audio I/O |
| `startStream(ip, port)` | Start UDP receive           |
| `stopStream()`          | Stop audio stream           |
| `sendTalkback(buffer)`  | Push mic audio data         |
| `getStats()`            | Return jitter/latency       |

---

## ⚡ Latency Optimization

| Area    | Optimization                           |
| ------- | -------------------------------------- |
| Codec   | Opus 2.5 ms frame, bitrate 64–128 kbps |
| Network | UDP Unicast, no retransmit             |
| Buffer  | Playback ≤ 64 samples                  |
| Router  | Wi-Fi 6 dedicated, 5 GHz channel       |
| OS      | Disable Wi-Fi power save               |
| QoS     | Enable WMM audio priority              |
| Clock   | Local NTP sync or timestamp            |

**Target latency:**

* One-way (monitor): **10–20 ms**
* Full duplex (talkback): **25–40 ms**

---

## 🧰 Build & Deployment

### Build

```bash
# Server
cd server
mkdir build && cd build
cmake ..
make

# Client Native Core
cd client/native_audio_core
mkdir build && cd build
cmake ..
make

# Flutter (mobile & web)
cd client/flutter_app
flutter pub get
flutter build apk
flutter build web
```

### Run

```bash
# Run server (headless)
./iem_server --config ../resources/default-config.json

# Access web dashboard
http://192.168.10.10:8080

# Run client
flutter run
```

---

## 🗂️ Folder Structure

```
iem-system/
├── README.md
├── LICENSE
│
├── docs/
│   ├── architecture-diagram.png
│   ├── network-setup-guide.md
│   ├── api-reference.md
│   └── latency-tuning.md
│
├── server/
│   ├── CMakeLists.txt
│   ├── src/
│   │   ├── main.cpp
│   │   ├── AppConfig.h
│   │   ├── AudioEngine/
│   │   │   ├── AudioInputManager.cpp/h
│   │   │   ├── MixEngine.cpp/h
│   │   │   ├── OpusEncoder.cpp/h
│   │   │   ├── OpusDecoder.cpp/h
│   │   │   ├── TalkbackRouter.cpp/h
│   │   │   └── ConfigManager.cpp/h
│   │   ├── Network/
│   │   │   ├── UDPServer.cpp/h
│   │   │   ├── ClientRegistry.cpp/h
│   │   │   ├── DiscoveryService.cpp/h
│   │   │   ├── WebSocketServer.cpp/h
│   │   │   └── HttpApi.cpp/h
│   │   └── Utils/
│   │       ├── Logger.cpp/h
│   │       └── TimeSync.cpp/h
│   ├── resources/
│   │   ├── default-config.json
│   │   └── web-dashboard/   # Flutter Web build output
│   └── tests/
│       ├── opus_test.cpp
│       ├── network_test.cpp
│       └── latency_benchmark.cpp
│
├── client/
│   ├── flutter_app/
│   │   ├── lib/
│   │   │   ├── main.dart
│   │   │   ├── screens/
│   │   │   │   ├── monitor_screen.dart
│   │   │   │   ├── settings_screen.dart
│   │   │   │   └── about_screen.dart
│   │   │   ├── widgets/
│   │   │   │   ├── ptt_button.dart
│   │   │   │   ├── volume_slider.dart
│   │   │   │   └── connection_status.dart
│   │   │   ├── services/
│   │   │   │   ├── audio_service.dart
│   │   │   │   ├── network_service.dart
│   │   │   │   ├── discovery_service.dart
│   │   │   │   └── settings_service.dart
│   │   │   └── models/
│   │   │       ├── audio_config.dart
│   │   │       └── network_status.dart
│   │   └── web/
│   │       └── build/     # Web dashboard build
│   │
│   └── native_audio_core/
│       ├── CMakeLists.txt
│       ├── include/
│       │   ├── AudioEngine.h
│       │   ├── OpusWrapper.h
│       │   ├── UDPClient.h
│       │   └── CircularBuffer.h
│       ├── src/
│       │   ├── AudioEngine.cpp
│       │   ├── OpusWrapper.cpp
│       │   ├── UDPClient.cpp
│       │   ├── CircularBuffer.cpp
│       │   ├── platform/
│       │   │   ├── AndroidAudio.cpp
│       │   │   └── iOSAudio.mm
│       │   └── ffi_bindings.cpp
│       └── bindings/
│           ├── audio_bindings.dart
│           └── ffi_bridge.dart
│
├── shared/
│   ├── protocol/
│   │   ├── packet_definitions.h
│   │   ├── constants.h
│   │   └── message_types.h
│   ├── utils/
│   │   ├── checksum.cpp
│   │   ├── endian_utils.cpp
│   │   └── network_utils.cpp
│   └── docs/
│       └── opus_config_reference.txt
│
├── tools/
│   ├── scripts/
│   │   ├── build_all.sh
│   │   ├── run_server.sh
│   │   ├── network_test.py
│   │   └── client_discovery.py
│   └── configs/
│       ├── router_qos.txt
│       ├── ntp_config.txt
│       └── audio_profiles.json
│
└── third_party/
    ├── juce/
    ├── libopus/
    ├── boost/
    └── websocketpp/
```

---

## 🔗 Related projects & references

Kami juga mengacu pada proyek SonoBus sebagai referensi untuk beberapa komponen dan pengaturan jaringan/audio karena aplikasi tersebut merupakan implementasi nyata dari streaming audio low-latency menggunakan Opus dan teknologi terkait.

- Repository: https://github.com/sonosaurus/sonobus
- Ringkasan singkat: SonoBus adalah aplikasi cross-platform untuk streaming audio peer-to-peer ber-low-latency (menggunakan Opus), dengan kontrol kualitas/latency, discovery/connection server opsional, dan dukungan multiplatform (desktop & mobile). README dan repositori menunjukkan penggunaan JUCE, AOO, dan Opus.
- Relevansi untuk proyek ini: arsitektur jaringan low-latency, pengaturan Opus (bitrate/frame), discovery/connection server pattern, dan praktik build (CMake). Beberapa ide/komponen konfigurasi kami ambil/danujungi dari SonoBus (mis. parameter Opus, discovery, headless server design).
- Lisensi: SonoBus dilisensikan di bawah GNU GPLv3 (lihat LICENSE). Implikasi penting: jika kita menggunakan atau menyalin kode sumber dari SonoBus ke dalam repositori kita dan kemudian mendistribusikannya, hasilnya harus kompatibel dengan ketentuan GPLv3 (harus menyediakan source dan melisensikan kembali di bawah GPLv3). Jika hanya mengacu pada ide, arsitektur, atau parameter konfigurasi tanpa menyalin kode, tidak otomatis menimbulkan kewajiban GPL.

Tindakan disarankan:

1. Tandai setiap file/komponen yang diambil langsung dari SonoBus; dokumentasikan asalnya dan pertimbangan lisensi.
2. Jika ingin mengintegrasikan kode dari SonoBus, pertimbangkan opsi lisensi dan/atau gunakan alternatif permissive untuk komponen yang kritikal.

## ✅ Summary

| Aspect             | Implementation                                                                      |
| ------------------ | ----------------------------------------------------------------------------------- |
| **Server Type**    | Headless C++ (JUCE) with WebSocket/HTTP                                             |
| **Dashboard**      | Flutter Web (control & monitoring)                                                  |
| **Client**         | Flutter Mobile + C++ native audio                                                   |
| **Network**        | Wi-Fi 6 / LAN only                                                                  |
| **Codec**          | Opus 2.5 ms low-delay                                                               |
| **Target Latency** | 10–20 ms (monitor), 25–40 ms (talkback)                                             |
| **Architecture**   | Hybrid Web (no desktop GUI)                                                         |
| **Build System**   | CMake + Flutter FFI                                                                 |
| **Goal**           | Reliable, ultra-low-latency IEM + talkback system for musicians and sound engineers |

---

*End of Technical Documentation*
