# MeshCore JB Network — New Node Operator Manual

**Johor Bahru, Malaysia | Community Off-Grid LoRa Mesh**
*Prepared: June 2026 | MCMC MTSFB TC T007:2020 Compliant*

---

## Table of Contents

1. [What is this network?](#1-what-is-this-network)
2. [MCMC Regulatory Compliance](#2-mcmc-regulatory-compliance)
3. [Radio Settings](#3-radio-settings)
4. [Node Types — Repeater vs Companion](#4-node-types--repeater-vs-companion)
5. [Building a Repeater Node](#5-building-a-repeater-node)
6. [Joining as a Companion](#6-joining-as-a-companion)
7. [Companion Apps](#7-companion-apps)
8. [DIY Coco Flowerpot Antenna](#8-diy-coco-flowerpot-antenna)
9. [New Node Onboarding Steps](#9-new-node-onboarding-steps)
10. [Current Network Coverage](#10-current-network-coverage)
11. [Network Scaling Roadmap](#11-network-scaling-roadmap)
12. [LetsMesh Observer & MQTT](#12-letsmesh-observer--mqtt)
13. [JMB Outreach Template (Bahasa Malaysia)](#13-jmb-outreach-template-bahasa-malaysia)
14. [Quick Reference Card](#14-quick-reference-card)

---

## 1. What is this network?

The **MeshCore JB Network** is a community-built, off-grid, licence-free mesh communication system for the Johor Bahru urban area. It requires **no internet, no cellular network, and no monthly fees**.

| Property | Value |
|---|---|
| Protocol | MeshCore (store-and-forward addressed routing) |
| Operating frequency | 919.800 MHz (MCMC Row 36 — ISM band, licence-free) |
| Network map | [map.meshcore.io](https://map.meshcore.io) |

**Primary use cases:**
- Community text messaging (no internet required)
- Emergency communication — flood alerts, distress calls
- Growing a resilient off-grid communication layer across JB

---

## 2. MCMC Regulatory Compliance

All nodes operate under **MCMC MTSFB TC T007:2020** (Short Range Devices — Specifications, Second Revision).

### Applicable frequency band

| Row | Band | Max Power | Restriction | Our use |
|---|---|---|---|---|
| 35 | 916–919 MHz | 25 mW EIRP | SRC | Not used — too low |
| **36** | **919–923 MHz** | **500 mW EIRP** | SRC + RFID | **✅ Our operating band** |
| 37 | 923–924 MHz | 500 mW EIRP | Duty cycle <1% OR freq hop OR LBT | Not used |

Our operating frequency **919.800 MHz** sits within Row 36 — no duty cycle restriction, 500 mW EIRP limit, SRC device classification. **Fully compliant.**

### TX power compliance

- Maximum TX power across all node types: **22 dBm**
- With a ~3 dBi antenna: resulting EIRP ≈ **315–500 mW** — within the 500 mW legal limit

> ⚠️ **WARNING — TX Power Cap:** Do NOT exceed **22 dBm** TX on any node. At 25–30 dBm with any antenna gain, EIRP exceeds 500 mW and violates MCMC Row 36. **TX is capped at 22 dBm across ALL nodes — repeater and companion alike. This is the MeshCore firmware maximum.**

### RFID coexistence

UHF RFID in Malaysia legally shares the 919–923 MHz band. RFID interrogators cluster around 920–921 MHz. Our chosen **919.800 MHz** provides ~200–1200 kHz separation from the RFID concentration zone.

---

## 3. Radio Settings

### Current production settings

```
Frequency        :  919.800 MHz
Bandwidth        :  125 kHz
Spreading Factor :  SF12
Coding Rate      :  CR5 (4/5)
TX Power         :  22 dBm  ← DO NOT EXCEED
Link Budget      :  ~157–158.5 dB
```

> 🔴 **All nodes — repeaters and companions — must use identical radio settings to communicate. Do not change these without coordinating a simultaneous network-wide update.**

### Parameter rationale

| Parameter | Value | Reason |
|---|---|---|
| 919.800 MHz | Row 36 | No duty cycle restriction. Clear of Meshtastic MY (922.875 / 921.125 MHz). Away from RFID concentration. |
| 125 kHz BW | Best sensitivity | Each BW doubling costs ~3 dB link budget. 125 kHz maximises range for sparse network. |
| SF12 | Max link budget | +158.5 dB. Critical for long urban gaps (e.g. Perling Rawa to Kempas Denai ~5.5 km). Same as Meshtastic LongSlow. |
| CR5 (4/5) | Efficient FEC | Sufficient error correction. Lower overhead vs CR8. Faster airtime without range penalty. |
| 22 dBm | MeshCore max | MeshCore firmware caps TX at 22 dBm. With ~3 dBi antenna, EIRP ≈ 315–400 mW — within the 500 mW legal limit. |

### Comparison with Meshtastic Malaysia settings

| Parameter | MeshCore JB (ours) | Meshtastic LongFast MY | Meshtastic MediumFast MY |
|---|---|---|---|
| Frequency | 919.800 MHz | 922.875 MHz | 921.125 MHz |
| Bandwidth | 125 kHz | 250 kHz | 250 kHz |
| SF | SF12 | SF11 | SF9 |
| CR | 4/5 | 4/5 | 4/5 |
| Link Budget | ~157 dB | 153 dB | 148 dB |
| Protocol | Store & Forward | Flooding | Flooding |
| Duty Cycle Rule | Row 36 — None | Row 37 issue | Row 36 — OK |
| Range | **Best** | Good | Shorter |

### Message delivery time at SF12

| Factor | Duration |
|---|---|
| Packet airtime (typical text) | 1.5 – 2.5 seconds |
| Store & forward queue delay | 0 – 2 seconds |
| Per additional hop | +1 – 2 seconds |
| **Typical 2-hop delivery total** | **5 – 8 seconds** |

For flood alerts and distress calls, 5–8 second delivery is entirely acceptable. The 3 dB link budget advantage of SF12 over SF11 is more valuable than saving 2–3 seconds in a sparse network with long inter-node gaps.

---

## 4. Node Types — Repeater vs Companion

There are two roles in this network. Understand which one you are taking on before proceeding.

| | Repeater Node | Companion Node |
|---|---|---|
| Role | Fixed infrastructure, always-on relay | Personal device you carry or use at home |
| Location | Rooftop or elevated outdoor position | Anywhere — handheld, desktop, mobile |
| Power | Solar / mains — must run continuously | USB, battery pack, or internal battery |
| Firmware | Community repeater firmware (e.g. EasySkyMesh) or any MeshCore-compatible repeater firmware | Official MeshCore from [meshcore.io](https://meshcore.io) |
| Flash tool | Firmware-specific (see Section 5) | [meshcore.io/flasher](https://meshcore.io/flasher) |
| Hardware | Any LoRa hardware capable of running MeshCore | Any MeshCore-supported device |
| Enclosure | Weatherproof outdoor box required | No enclosure requirement |
| CLI / config | Firmware-dependent CLI | MeshCore app or web interface |

> 📌 The hardware example in Section 5 (Seeed Xiao S3 + EasySkyMesh) is the **JB network's own reference build**. It is not a requirement. Any hardware and firmware combination that runs MeshCore with the correct radio settings will work as a repeater.

---

## 5. Building a Repeater Node

### Hardware — any compatible platform works

You are free to build your repeater on any LoRa hardware that supports MeshCore. The table below shows the JB network's current reference build as an example.

| Component | JB Reference Build | Notes |
|---|---|---|
| MCU/Radio | Seeed Studio Xiao S3 Wio + SX1262 | Max TX +22 dBm |
| Firmware | EasySkyMesh PowerSaving15 | 10–13 mA idle — optimised for solar |
| Connector | IPEX (U.FL) → 5cm pigtail → N-Type | Minimal feedline loss |
| Feedline | 15 cm RG-58 | Negligible loss at this length |
| Enclosure | Weatherproof outdoor box | Required for all outdoor repeaters |
| Solar Panel | 5W, 5V | Sufficient with powersaving firmware |
| Battery | 2× 18650 2400 mAh — **PARALLEL** (3.7V) | ~17.76 Wh total |
| Charger | CN3791 MPPT | 5V → 3.7V |

Other known-working platforms include T-Beam, HELTEC LoRa32, RAK WisBlock, and similar SX1262/SX1276-based boards. Check [meshcore.io](https://meshcore.io) for the current supported hardware list.

### Firmware options for repeaters

| Firmware | Source | Notes |
|---|---|---|
| EasySkyMesh PowerSaving15 | github.com/IoTThinks/EasySkyMesh/releases | Recommended for solar builds — ~10× lower idle current |
| Official MeshCore repeater builds | meshcore.io/flasher | Use the official flasher to select your board and role |
| Community firmware | Various | Any MeshCore-protocol-compatible repeater firmware |

### Power budget (EasySkyMesh reference build — JB conditions)

| Metric | Value |
|---|---|
| Idle current (powersaving mode) | 10–13 mA @ 3.3V |
| Power consumption | ~0.044 W |
| Daily energy use | ~1.06 Wh/day |
| Battery capacity (usable 80% DoD) | ~14.2 Wh |
| Battery-only runtime (no solar) | ~13–14 days |
| 5W panel on clear day (5h sun) | ~25 Wh generated |
| 5W panel on overcast monsoon day | ~2–4 Wh generated |
| Monsoon day surplus/deficit | **+1–3 Wh surplus — safe** |

> 📌 **If using EasySkyMesh:** Run `powersaving on` via CLI **after flashing and before configuring radio settings**. Verify with `powerlog`. This step is specific to EasySkyMesh — other firmware has its own configuration method.

### Minimum repeater requirements (any build)

Regardless of hardware or firmware choice, a repeater joining this network must:

- [ ] Operate on **919.800 MHz, 125 kHz BW, SF12, CR5**
- [ ] TX power set to **22 dBm maximum**
- [ ] Be mounted at an **elevated outdoor position** with a suitable antenna
- [ ] Run **continuously** (solar, mains, or large battery backup)
- [ ] Use a **weatherproof enclosure**

---

## 6. Joining as a Companion

Companion users join the network using a personal device — phone, handheld, laptop, or dedicated LoRa hardware. The easiest way to get started:

1. Go to **[meshcore.io](https://meshcore.io)** and check the list of supported devices
2. Flash your device using the official flasher at **[meshcore.io/flasher](https://meshcore.io/flasher)**
3. Set the radio parameters from Section 3 to match the JB network
4. You are on the mesh — no registration, no account needed

### Supported companion hardware

The official MeshCore site lists all currently supported devices. Common options include dedicated LoRa handhelds, T-Echo, T-Beam, and other SX1262-based boards. Check [meshcore.io](https://meshcore.io) for the up-to-date list — new devices are added regularly.

### TX power reminder for companions

Companion devices with high-gain antennas can also exceed 500 mW EIRP if TX is set too high. Keep TX at **22 dBm maximum** regardless of antenna type.

---

## 7. Companion Apps

There are two companion app options for joining the JB mesh network. Both speak the same on-air MeshCore protocol — users of either app share the same mesh seamlessly.

### Official MeshCore app

The primary app maintained by the MeshCore project. Get it via the official flasher and app store links at [meshcore.io](https://meshcore.io).

### KiekR — community toolbox

**[kiekr.app](https://kiekr.app)** is an independent, community-built MeshCore client for iOS and Android. It pairs over Bluetooth with the same MeshCore-compatible LoRa radio the official app uses — no separate hardware, no firmware change required. KiekR users and official-app users share one seamless mesh.

| Platform | Download |
|---|---|
| Android (Google Play open beta) | play.google.com/apps/testing/app.kiekr |
| Android (direct APK) | kiekr.app/download/android/latest |
| iOS (TestFlight beta) | testflight.apple.com/join/X8wnyh87 |

> KiekR is currently in open beta. Android users can install directly from the Play Store listing without a waitlist or sign-up.

#### What KiekR adds over the official app

KiekR is built for operators who want more visibility and control over the mesh. Key features:

**Messaging & routing**
- Direct messages with end-to-end encryption (X25519 + ChaCha20-Poly1305 + SHA-256)
- Public channel messaging with the standard `#channel` hashtag convention
- Manual path control — hand-pick the repeater hop sequence for a DM instead of relying on auto-learned paths
- Per-message region display — each message bubble shows which MeshCore region it came from
- Stale path recovery — trigger fresh path discovery when a learned repeater path goes dead

**Repeater management**
- Repeater ACL status display — shows whether your identity is in Group A (no ACL) or Group B (logged in) for each repeater
- ACL login workflow — send your repeater password and cache the result per identity
- Tunable neighbour fetch with adjustable retry count for lossy links
- Region admin commands for repeater operators with admin access

**Coverage & discovery**
- Online hop resolution — looks up unknown path hops against the kiekr.app or meshcorenetz.de directory
- Path visualisation — shows decoded hop chains colour-coded on a map
- Contribution to the live coverage map at [map.kiekr.app](https://map.kiekr.app) — opt-in, signed with a separate key, no message content uploaded

**MQTT bridge**
- Built-in MQTT bridge forwards raw LoRa packets from your radio to any broker (LetsMesh, meshcorenetz.de, or your own)
- Replaces the need for a separate Raspberry Pi MQTT bridge for companion nodes that have WiFi
- Configurable per broker: host, port, TLS, Wi-Fi only mode, IATA region tag

**Multi-identity & backup**
- Multi-identity history — KiekR remembers every radio identity you have ever connected, so switching nodes preserves DM history per identity
- Full backup/restore in stock-compatible MeshCore JSON format — importable into the official app
- KiekR-specific data (login cache, manual paths, MQTT config) exported separately so the stock-compat bundle stays clean

**Privacy**
- No analytics, no telemetry, no crash reporter phoning home
- Coverage map contributions are signed with a separate generated key — your MeshCore identity is never uploaded
- MQTT publishing is off by default — nothing leaves the phone unless you configure a broker

#### KiekR vs official app — quick comparison

| Feature | Official MeshCore app | KiekR |
|---|---|---|
| Messaging & DMs | Yes | Yes |
| Public channels | Yes | Yes |
| Manual path control | Yes | Yes |
| Per-message region display | No | Yes |
| Repeater ACL workflow | Basic | Full (Group A/B status, login cache) |
| MQTT bridge | No | Yes — built-in |
| Coverage map contribution | No | Yes — map.kiekr.app |
| Multi-identity history | No | Yes |
| Hop resolution & path map | No | Yes |
| Backup format | MeshCore JSON | MeshCore JSON + KiekR bundle |

---

## 8. DIY Coco Flowerpot Antenna

The JB network's active repeaters use a DIY Coco Flowerpot (collinear) antenna tuned directly to **919.800 MHz**. This is a simple, low-cost, high-performance antenna that anyone can build from coaxial cable.

### What is a Coco Flowerpot antenna?

A Coco (collinear) Flowerpot is a half-wave dipole-based collinear antenna built from a single piece of coax. It has ~3 dBi gain, omnidirectional pattern, and is well suited for fixed repeater use. No soldering required — it is built entirely from the coax itself using calculated velocity factor lengths.

### Velocity factor values used (RG-58)

| Section | Velocity Factor | Purpose |
|---|---|---|
| Upper radiator (foam core) | **VF 0.83** | Half-wave radiating element |
| Lower phasing stub (solid core) | **VF 0.66** | Quarter-wave phasing coil |

### Length calculator

The antenna dimensions are calculated from the target frequency and the velocity factor of each coax section. For **919.800 MHz** with RG-58:

**Formula:**

```
Half-wave length (mm) = (VF × 150,000) / frequency_MHz

Upper radiator  = (0.83 × 150,000) / 919.800  =  135.4 mm
Lower stub      = (0.66 × 150,000) / 919.800  =  107.6 mm
```

| Section | VF | Calculated length |
|---|---|---|
| Upper half-wave radiator | 0.83 | **135.4 mm** |
| Lower quarter-wave phasing stub | 0.66 | **107.6 mm** |

> These lengths are tuned to 919.800 MHz. If you change target frequency, recalculate both sections using the same formula. The interactive calculator below lets you adjust frequency and VF values.

### Build notes

- Use RG-58 throughout for both sections
- The phasing stub is formed by folding the coax back on itself (the braid acts as the return path)
- Weather-seal all joints with self-amalgamating tape
- Mount vertically — the antenna is omnidirectional only when vertical
- A 15 cm RG-58 pigtail to N-Type connector at the feedpoint is sufficient; cable loss at this length is negligible

---

## 9. New Node Onboarding Steps

### Companion users

1. Visit [meshcore.io](https://meshcore.io) — check supported devices
2. Flash using [meshcore.io/flasher](https://meshcore.io/flasher) — select your board and the companion role
3. Set radio parameters: **919.800 MHz · 125 kHz · SF12 · CR5 · 22 dBm max**
4. You are on the network — check [map.meshcore.io](https://map.meshcore.io) to see nearby nodes

### Repeater operators — full checklist

Follow all steps in order.

**Step 1 — Choose and prepare your hardware**

Select any MeshCore-compatible LoRa hardware. See Section 5 for the JB reference build and alternative platforms. Obtain a weatherproof outdoor enclosure.

**Step 2 — Flash firmware**

- For EasySkyMesh: download from `github.com/IoTThinks/EasySkyMesh/releases`
- For official MeshCore repeater build: use [meshcore.io/flasher](https://meshcore.io/flasher), select your board and repeater role
- For other community firmware: follow that firmware's flashing instructions

**Step 3 — Enable power saving (EasySkyMesh only)**

If using EasySkyMesh, connect via serial CLI and run:
```
powersaving on
```
Verify with:
```
powerlog
```
Do this **before** configuring radio settings. Skip this step if using other firmware.

**Step 4 — Configure radio settings**

Set on all firmware types:
```
Frequency    :  919.800 MHz
Bandwidth    :  125 kHz
SF           :  12
CR           :  4/5 (CR5)
TX Power     :  22 dBm  ← MeshCore firmware maximum
```

**Step 5 — Assemble and weatherproof**

- Mount board in weatherproof outdoor enclosure
- Connect antenna feedline (keep feedline as short as practical)
- If solar-powered: wire battery in **parallel** (not series), connect MPPT charger

**Step 6 — Choose location**

- Select rooftop or elevated position with clear sky and line-of-sight toward existing nodes
- Check [map.meshcore.io](https://map.meshcore.io) to identify nearest active repeaters
- For building rooftop access, use the **JMB outreach template** in Section 12

**Step 7 — Verify TX power compliance**

Before going live:
- [ ] TX is set to **22 dBm maximum**
- [ ] No external amplifiers in the RF path
- [ ] EIRP with antenna gain does not exceed 500 mW

**Step 8 — Report to the network**

- Share your node location, height, and antenna type in the network group
- Confirm the node appears on [map.meshcore.io](https://map.meshcore.io)
- Confirm connectivity with at least one neighbouring node before declaring active

---

## 10. Current Network Coverage

### Active and planned repeater nodes

| Node | Location | Zone | Height | Antenna | Status |
|---|---|---|---|---|---|
| Repeater 1 | Taman Perling Rawa | West | 7m roof + 3m pole = 10m | DIY Coco 919.800 MHz | ✅ Active |
| Repeater 2 | Kempas Denai | Central-West | 3-storey hillside + 10ft pole (~13m eff.) | DIY Coco 919.800 MHz | ✅ Active |
| Repeater 3 | Taman Setia Tropika | North-West | 7m rooftop | Gizont 5dBi fiberglass | ✅ Active |
| Repeater 4 | Bandar Baru Uda (APM Building) | South | TBC — elevated | TBC | 🔶 Upcoming |

### Target recruit areas — repeater nodes needed

| Area | Zone | Notes |
|---|---|---|
| Taman Daya | Central-East | Operator recruit needed |
| Taman Molek | East | Operator recruit needed |
| Taman Bestari Indah | North-East | Operator recruit needed |

If you live in or near these areas, your repeater node could be a critical link in the network.

---

## 11. Network Scaling Roadmap

As node count grows and channel congestion increases, radio settings must be progressively adjusted. **All nodes — repeaters and companions — must be updated simultaneously** when changing settings. A partial update will split the network.

### Stage progression

| Stage | Node Count | Frequency | BW | SF | CR | Expected Range/Hop |
|---|---|---|---|---|---|---|
| 1 — Current | <10 nodes | 919.800 MHz | 125 kHz | SF12 | CR5 | 8–12 km |
| 2 — Growing | 10–20 nodes | 919.800 MHz | 125 kHz | SF11 | CR5 | 6–9 km |
| 3 — Established | 20–50 nodes | 919.800 MHz | 250 kHz | SF10 | CR5 | 4–6 km |
| 4 — Dense Urban | 50+ nodes | 919.800 MHz | 250 kHz | SF9 | CR5 | 3–4 km |

> 📌 Frequency (919.800 MHz) and TX power (22 dBm) remain constant at ALL stages. Only SF and BW change. CR5 is sufficient at all stages.

### Trigger points — when to change settings

| Symptom | Action |
|---|---|
| Messages consistently taking >30 seconds | Drop SF by 1 step |
| Channel utilisation >30% | Increase BW: 125 → 250 kHz |
| Nodes missing each other despite good RSSI | Reduce SF |
| Store-and-forward queue constantly full | Change both SF and BW |

---

## 12. LetsMesh Observer & MQTT

A LetsMesh MQTT observer publishes the network's activity to the public map, making nodes and messages visible at [analyzer.letsmesh.net](https://analyzer.letsmesh.net). An observer requires a node with WiFi access and ideally mains power.

### Current observer candidates

| Node | WiFi | Observer Priority | Notes |
|---|---|---|---|
| Perling Rawa | Yes | Secondary | Solar-only — Pi Zero W possible |
| Kempas Denai | Yes | Secondary | Solar-only — Pi Zero W possible |
| Setia Tropika | Yes | Secondary | Solar-only — Pi Zero W possible |
| **BBU APM Building** | **TBC** | **PRIMARY** | Mains power + elevation — ideal first observer |

### Observer setup options

| Option | Method | Power Saving Kept | Complexity |
|---|---|---|---|
| A | Reflash with LetsMesh observer-uplink-native-dev firmware | Unknown | Low |
| **B ★** | **Raspberry Pi Zero W via USB running meshcoretomqtt script** | **Yes — EasySkyMesh intact** | **Medium** |
| C | Home Assistant MeshCore integration | Yes | Medium |

**Recommendation:** Option B (Raspberry Pi Zero W) at BBU APM Building (~RM45–50). Preserves EasySkyMesh powersaving on the Xiao S3 while adding LetsMesh upload via the Pi's WiFi.

Observer onboarding: [analyzer.letsmesh.net/observer/onboard](https://analyzer.letsmesh.net/observer/onboard)

---

## 13. JMB Outreach Template (Bahasa Malaysia)

Use this template when approaching building management (JMB) for rooftop repeater installation permission. Fill in `[Nama Bangunan]`, `[Nama]`, `[No. Telefon]`, and `[Tarikh]`.

---

**CADANGAN PEMASANGAN PERALATAN KOMUNIKASI KECEMASAN**

Kepada,
Jawatankuasa Pengurusan Bangunan (JMB),
[Nama Bangunan]

Assalamualaikum / Salam Sejahtera,

Saya [Nama], penduduk kawasan berdekatan, ingin mencadangkan pemasangan satu peralatan komunikasi kecemasan kecil di bumbung bangunan ini secara **percuma dan tanpa sebarang kos** kepada pihak pengurusan.

**Apakah peralatan ini?**
Sebuah kotak kecil bersaiz telefon, berkuasa solar, yang membolehkan komunikasi teks tanpa memerlukan internet, WiFi atau Celcom/Maxis/Digi. Ia beroperasi menggunakan teknologi radio jarak jauh yang digunakan di seluruh dunia untuk komunikasi kecemasan.

**Manfaat kepada penduduk:**
- Komunikasi tetap berfungsi semasa gangguan internet atau bekalan elektrik
- Berguna semasa banjir, kecemasan atau bencana
- Tiada kos bulanan atau yuran langganan
- Tiada penggunaan elektrik bangunan — berkuasa solar sepenuhnya
- Saiz kecil, kemas dan tidak mengganggu estetik bangunan

**Apa yang diperlukan:** Kebenaran untuk memasang peralatan kecil di bumbung dan akses sekali sekala untuk penyelenggaraan.

Saya sedia hadir untuk membuat demonstrasi langsung pada bila-bila masa yang sesuai.

Yang ikhlas,
[Nama] | [No. Telefon] | [Tarikh]

---

## 14. Quick Reference Card

### Radio settings — all nodes

| Parameter | Value |
|---|---|
| Frequency | **919.800 MHz** |
| Bandwidth | **125 kHz** |
| Spreading Factor | **SF12** |
| Coding Rate | **CR5 (4/5)** |
| TX Power | **22 dBm max — MeshCore firmware limit** |
| Regulation | MCMC Row 36 — 919–923 MHz, 500 mW EIRP |
| Protocol | MeshCore store-and-forward |

### Firmware

| Role | Firmware | Source |
|---|---|---|
| Companion | Official MeshCore | meshcore.io/flasher |
| Repeater (solar/powersave) | EasySkyMesh PowerSaving15 | github.com/IoTThinks/EasySkyMesh/releases |
| Repeater (other) | Official MeshCore repeater build | meshcore.io/flasher |

### CLI commands — EasySkyMesh repeater firmware only

| Command | Purpose |
|---|---|
| `powersaving on` | Enable power saving — run BEFORE radio config |
| `powersaving off` | Disable power saving (testing/config only) |
| `powerlog` | Check last reset reason and boot voltage |

### Coco Flowerpot antenna — 919.800 MHz lengths (RG-58)

| Section | VF | Length |
|---|---|---|
| Upper half-wave radiator | 0.83 | **135.4 mm** |
| Lower quarter-wave phasing stub | 0.66 | **107.6 mm** |

### Key links

| Resource | URL |
|---|---|
| MeshCore official site | meshcore.io |
| Official firmware flasher | meshcore.io/flasher |
| KiekR companion app | kiekr.app |
| KiekR coverage map | map.kiekr.app |
| MeshCore node map | map.meshcore.io |
| EasySkyMesh repeater firmware | github.com/IoTThinks/EasySkyMesh/releases |
| LetsMesh observer onboard | analyzer.letsmesh.net/observer/onboard |
| MCMC regulation | MCMC MTSFB TC T007:2020 — Table 1, Row 36 |

---

*MeshCore JB Network Documentation · June 2026 · For network operators and new node applicants*
