# Which processors can record 1080p clips in a pendant?

Ticket: #2. Researched 2026-11-21. Scope: Clip recording (H.264/H.265 at 1080p30 or better), MIPI camera, audio, Onboard Storage, Wi-Fi for Transfer, fit in a ~30 x 25 x 12 mm Pendant.

Evidence tags: **[V]** read in a manufacturer document or official page (source ID given). **[E]** estimate or inference by me. **[U]** unverified.

## Recommendation

1. **Primary: Realtek RTL8735B (Ameba Pro2).** It is the only candidate with all of these in one 10 x 10 mm chip: H.264/H.265 encode, 4-lane MIPI CSI-2, Wi-Fi plus BLE, and a microamp-class deep sleep. Prototype firmware on the AMB82-Mini board (the Arduino SDK has an MP4-to-SD example), then move to a factory-assembled custom board or a module. [V: S1, S2, S3, S13]
2. **Fallback: Rockchip RV1106 (G2/G3).** Better ISP, Linux, and a real datasheet, but it needs an external Wi-Fi chip, and the 0.35 mm pitch package is hard to assemble. Pick it if Linux matters more than battery life and size. [V: S5, S6]
3. **Open-source-leaning alternative: ESP32-P4 plus an ESP32-C6 companion.** H.264 only (no H.265), 2-lane CSI, and the datasheet I could read was a v0.7 pre-release. Re-check the final documents before choosing it. [V: S7, S8]

Not chosen: Raspberry Pi Zero 2 W (board too big, idle power high), Kendryte K230 (works, but no integrated Wi-Fi, BGA with stacked or external LPDDR4, little power data), STM32N6 (no 1080p30 encode figure found).

## Chips that fail the requirement

| Chip | Why it fails | Source |
|---|---|---|
| ESP32-S3 (and ESP32-CAM boards) | Camera port is 8 to 16-bit parallel DVP only. No MIPI CSI. No H.264 encoder in the datasheet's feature list. | S9 |
| RP2350 / RP2040 (Pico family) | Product page lists no video encoder and no MIPI CSI interface. | S10 |
| Raspberry Pi 5 / CM5 | Docs say Pi 5 uses software video encoders, so no hardware encoder. Also far too big and hot for a pendant [E]. | S11 |
| Generic Wi-Fi MCUs (nRF, plain Cortex-M) | No hardware video encoder, so 1080p30 is not realistic. [E] general knowledge, not individually sourced. | none |

## Comparison

No vendor publishes current while recording 1080p30 and writing to storage. Treat all active power as unknown until measured.

| Candidate | Encode | MIPI CSI | Wi-Fi | Package / size | Sleep | Active | SDK | Buy small qty | Assembly |
|---|---|---|---|---|---|---|---|---|---|
| **RTL8735B** | H.264, H.265, JPEG; page gives 1080p30 + 720p30 and also 1080p45 [V S1] | 4-lane CSI-2, up to 5 MP [V S1] | On chip 802.11 b/g/n 2.4 GHz (DM variant dual-band), BLE [V S1] | QFN128, 10 x 10 mm; 128 MB DDR2 listed, in-package or not [U] [V S1] | Module deep sleep about 4.6 to 41 uA, Realtek pages disagree; dev board measured about 70 mA from board parts [V S3, S4] | [U] | FreeRTOS and Arduino SDKs on GitHub; license is a Realtek disclaimer PDF, not OSI; source vs binary for video/ISP [U] [V S12] | AMB82-Mini sold by retailers [V S13]; bare chip [U] | 10 mm QFN is ordinary; DDR layout is the risk unless a module is used [E] |
| **RV1106G2/G3** | H.265 Main + H.264 High; G2 5 MP@30, G3 8 MP@15 [V S5] | 2 x 2-lane CSI, 1.5 Gbps/lane [V S5] | None on chip; Luckfox sells Wi-Fi 6 variants [V S6] | QFN128, 12.3 x 12.3 mm, 0.35 mm pitch, DDR3L inside; Luckfox Core1106 module 30 x 30 mm [V S5, S6] | Not in datasheet [U] | Not in datasheet [U] | Buildroot Linux SDK [V S6]; openness [U] | Core1106 about $16 to $27 on Amazon/AliExpress [V S6] | 0.35 mm pitch is hard; the 30 x 30 module is larger than the pendant face [E] |
| **ESP32-P4 + C6** | H.264 baseline only, max 1080p30; no H.265 [V S7] | One interface, 2 lanes x 1.5 Gbps [V S7] | Only via external ESP32-C/S chip (ESP-Hosted) [V S8] | QFN104, 10 x 10 mm, plus companion [V S7] | Deep sleep 12 uA typ; light sleep 0.085 to 0.8 mA (v0.7 preliminary) [V S7] | [U] | ESP-IDF; H.264 driver maturity [U] [V S8] | [U] | Two chips, 10 mm QFN: easy [E] |
| **Kendryte K230** | H.264 up to High, H.265 Main; 8 MP@20 fps, which covers 1080p30 by arithmetic [E] [V S14, S15] | 3 interfaces (4+2 lanes or 3 x 2) [V S14] | None listed; companion needed [V S14] | BGA 0.65 mm pitch, 13 x 13 mm, or 11 x 11 mm with 128 MB LPDDR4 [V S14] | PMU deep sleep 20 uW or less [V S14] | [U] | RISC-V; SDK repos not checked [U] | [U] | BGA plus DRAM routing: harder [E] |
| **STM32N6** | Datasheet lists an H.264 encoder; 1080p30 limit not found [U] [V S16, S17] | 2-lane CSI-2 [V S16] | None | [U] | [U] | [U] | ST tooling; users call the reference manual vague on the encoder [V S18] | [U] | [U] |
| **RPi Zero 2 W** | H.264 encode 1080p30 [V S19] | CSI-2 connector [V S19] | On board 2.4 GHz plus BT 4.2 [V S19] | Board 65 x 30 mm, far over 30 x 25 [V S19] | No real sleep; idle about 100 to 120 mA (community figure) [V S20] | About 350 mA typical bare board [V S20] | Linux, very mature [E] | In production until at least Jan 2030 [V S19] | No pendant-sized form [U] |

## Open risks

1. **Power is unmeasured everywhere.** Battery size must come from prototype measurement, including storage writes and the Wi-Fi Transfer.
2. **RTL8735B documentation conflicts.** 1080p30 vs 1080p45 on one page, and deep-sleep figures that differ by about 10x between Realtek pages. [V S1, S3, S4]
3. **H.265 on RTL8735B is less proven.** A forum thread reports a freeze when switching from H.264 to H.265. Use H.264 first. [V S21]
4. **RTL8735B SDK is not under an open-source license.** I did not confirm whether the video pipeline ships as source or binary. Check the repo and license PDF. [V S12]
5. **Bare-chip buyability.** I found board and module availability, not distributor stock or minimums for the bare RTL8735B. A factory build probably needs a module or a vendor relationship. [U]
6. **Fit.** A 10 x 10 mm chip plus sensor, microphone, flash, PMIC and antenna inside 30 x 25 x 12 mm is plausible [E] but not checked against a layout.
7. **Not evaluated:** Allwinner, SigmaStar, Ingenic, Sophgo SG200x, Ambarella, i.MX. Sensor choice and the Recording Indicator are separate tickets.
8. **Fetch limits.** Several pages were read through a summarising fetcher, and the ESP32-P4 datasheet obtained was a v0.7 pre-release, so figures may differ in the final release. The STM32N6 datasheet itself could not be read.

## Sources

- S1 RTL8735B product page: https://aiot.realmcu.com/en/product/rtl8735b.html
- S2 AMB82-Mini 1080p MP4 recording guide (third party): https://how2electronics.com/1080p-mp4-video-recorder-with-amb82-mini-iot-ai-camera/
- S3 Ameba Deep Sleep docs: https://ameba-doc-arduino-sdk.readthedocs-hosted.com/en/latest/ameba_pro2/amb82-mini/Example_Guides/PowerMode/Deep%20Sleep%20Mode.html
- S4 Ameba forum, board deep-sleep measurement (user report): https://forum.amebaiot.com/t/deep-sleep-power-consumption-issue-on-amb82-mini/4535
- S5 RV1106 Datasheet Rev 1.8 (Rockchip, hosted copy): https://wmsc.lcsc.com/wmsc/upload/file/pdf/v2/lcsc/2405141706_Rockchip-RV1106G3_C5328706.pdf
- S6 Luckfox Core1106: https://wiki.luckfox.com/Core1106/Introduction/ and https://www.cnx-software.com/2025/01/21/solderable-rockchip-rv1106-system-on-module-features-112-castellated-pins-offers-wifi-6-and-bluetooth-5-2-connectivity/
- S7 ESP32-P4 datasheet (v0.7 pre-release as fetched): https://documentation.espressif.com/esp32-p4_datasheet_en.pdf
- S8 ESP32-P4 product page: https://www.espressif.com/en/products/socs/esp32-p4
- S9 ESP32-S3 datasheet: https://documentation.espressif.com/esp32-s3_datasheet_en.pdf
- S10 RP2350 product page: https://www.raspberrypi.com/products/rp2350/
- S11 Raspberry Pi camera software docs: https://www.raspberrypi.com/documentation/computers/camera_software.html
- S12 Ameba Pro2 SDK repo: https://github.com/Ameba-AIoT/ameba-rtos-pro2
- S13 Retail listing for AMB82-Mini: https://eu.robotshop.com/products/realtek-amb82-mini-iot-ai-camera-arduino-dev-1080p-sensor-multiple-io
- S14 K230 brief datasheet: https://www.kendryte.com/k230/en/dev/K230_brief_datasheet.html
- S15 K230 full datasheet: https://www.kendryte.com/k230/en/dev/00_hardware/K230_datasheet.html
- S16 STM32N6 datasheet (listed in search results, not read): https://www.st.com/resource/en/datasheet/stm32n645i0.pdf
- S17 ST hardware video encoding wiki (seen in search results only): https://wiki.st.com/stm32mcu/wiki/Introduction_to_Hardware_Video_Encoding_with_STM32
- S18 ST community thread: https://community.st.com/stm32-mcus-products-25/stm32n6-h264-video-encoder-asic-datasheet-manual-150790
- S19 Raspberry Pi Zero 2 W: https://www.raspberrypi.com/products/raspberry-pi-zero-2-w/
- S20 Pi power figures: https://www.raspberrypi.org/documentation/hardware/raspberrypi/power/README.md and https://pidramble.com/wiki/benchmarks/power-consumption (community)
- S21 Ameba forum, H.264 to H.265 freeze: https://forum.amebaiot.com/t/ameba-pro2-mp4-recording-freezes-when-switching-from-h-264-to-h-265/4833
