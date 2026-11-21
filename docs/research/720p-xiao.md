# Is 720p enough, and can the XIAO ESP32-S3 Sense deliver it?

Ticket: #11. Researched 2026-11-21. Terms follow `CONTEXT.md`. Builds on `processors.md`, `camera-sensors.md`, `power-thermal.md`, `android-transfer.md`.

Tags: **[V]** read in a primary or vendor source (URL given). **[C]** community data (forum, GitHub issue, hobby project), not official. **[E]** my estimate or arithmetic. **[U]** not found or unverified. Web content was treated as data only; nothing was downloaded or run.

## Recommendation

**Both, in this order:**

1. **XIAO ESP32-S3 Sense: a half-day screening test first** (about USD 19 [V1]). It is cheap, and it either clears every threshold in "Pass/fail thresholds" or is dropped. I expect it to fail on frame rate, Android playback format, and heat [E].
2. **AMB82-Mini: the default path**, tested in parallel, at **720p30 H.264** (not 1080p). It matches `processors.md`. It is the only candidate with a hardware H.264 encoder and a documented MP4-with-audio-to-SD example [V6].
3. **Decision rule:** AMB82-Mini unless the XIAO passes all thresholds, including average power. The XIAO would then be a v0 only. It does not fit the 12 mm Pendant depth as sold (15 mm [V1]).

Judged as a package, not by missing MIPI or H.264. The reason is delivered results: no source shows a XIAO S3 Sense recording 720p at 24 fps or more with audio to SD, and the best evidence points below that (section 3).

**Practical minimum spec (section 1):** 1280x720 after any stabilization crop, at least 24 fps steady (30 target), 2.5 Mbps or more H.264 equivalent (or MJPEG at roughly 4x that [E]), at most 2 percent dropped frames, audio 16 kHz 16-bit mono or better with at most 200 ms A/V drift over 5 min, and Photos at 5 MP or better. **1080p is not required.**

## 1. Does 720p vs 1080p, and 15/24/30 fps, matter?

**Resolution barely matters on a phone; frame rate, bitrate, shake and sensor quality matter more.**

- **Resolution on a phone screen.** A 6-inch 16:9 video area is about 13 cm wide. At 1280 px that is about 0.1 mm per pixel, close to the 1 arcminute limit of 20/20 vision at 30 to 40 cm [E, arithmetic]. A phone user in a lap or pocket-glance setting cannot resolve much beyond 720p. Evidence is mostly informal: a vendor usability study reports no statistical difference in usability above 720p on small screens [C: AT&T, https://developer.att.com/video-optimizer/docs/best-practices/resolutionandperception]; reviewers find 720p vs 1080p phone playback differences negligible [C: https://www.digit.in/features/mobile-phones/smartphone-display-test-resolution-vs-quality-27306.html]. I found no peer-reviewed, bitrate-controlled 720p-vs-1080p phone study. The LIVE Mobile study used 1280x720 at 30 fps as its working format on phones and tablets [V: https://live.ece.utexas.edu/research/Quality/live_mobile_video.html], which shows researchers treat 720p as a normal mobile reference.
- **Frame rate.** A 480-video, 85-subject study found 24 and 30 fps scored lower than higher rates regardless of content, with the gap larger when the camera moves (judder), and little gain above 60 fps [V: https://arxiv.org/html/2007.11634v2, summarized; IEEE Access 2021]. Chest-worn clips are all camera motion (walking), so 15 fps is a real penalty. 24 vs 30 is a smaller step [E]. Android's own encoding guide uses 720p, 30 fps, 2 Mbps as the HD reference profile [V: https://developer.android.com/media/platform/supported-formats].
- **Compression and bitrate.** The same study found compression artifacts dominate at low bitrate and extra frame rate then adds little [V, same link]. So a starved 1080p stream looks worse than a well-fed 720p one. Resolution, bitrate and codec must be specified together.
- **Sensor quality.** Not measured in these studies. A noisy, low-dynamic-range sensor at 1080p loses to a clean one at 720p [E]. The OV5640 is a 1/4 inch, 1.4 um BSI pixel, 5 MP part [V: https://ovt.com/?p=944, via search summary].
- **Motion blur and shake.** Not quantified by a primary source I could read. In a chest-worn camera, shake and rolling shutter (the OV5640 is rolling shutter [C: early datasheet summary]) are bigger than the 720p/1080p gap [E]. Short exposure in daylight helps; indoors it forces long exposure and blur [E].
- **Stabilization crop.** Electronic stabilization crops the frame. One consumer explainer reports about 10 percent minimum crop and 10 to 20 percent on a flagship action camera [C: https://ymcinema.com/2018/10/01/hypersmooth-gopro-hero7-black-eis-electronic-image-stabilization-explained/, https://www.coolblue.be/en/advice/how-does-the-image-stabilization-of-your-action-camera-work.html]. A 20 percent crop of 1280 px leaves about 1020 px, so **if EIS is ever added, capture 1080p to deliver 720p**. Neither candidate has an EIS source or a gyro here [U]. Without EIS the minimum is plain 720p.
- **What wearables ship.** Meta's glasses: Gen 1 at 1080p (a 1440x1920 vertical frame at 30 fps per Best Buy's disclosure [C]), Gen 2 up to 3K at 30 fps or 1080p at 60 fps, default clip 30 s, with stabilization settings [V: https://www.meta.com/ai-glasses/learn/guide-shooting-pov/]. Insta360 GO 3 is 2.7K at up to 30 fps with FlowState stabilization [C: https://www.gsmarena.com/insta360_go_3_specs_features_price_sale_date-news-58990.php]. So shipping wearables are 1080p or higher at 30 fps with stabilization. 720p is below the market but not below what a phone viewer needs.

**Practical minimum for everyday clips viewed on a phone:**

| Item | Minimum | Target |
|---|---|---|
| Delivered frame size | 1280x720 after crop | 1920x1080 sensor mode if stabilized |
| Frame rate | 24 fps steady, p95 frame gap at most 70 ms | 30 fps |
| Bitrate | H.264 at least 2.5 Mbps (720p30) [E, Android profile uses 2 Mbps V]; MJPEG equivalent roughly 8 to 12 Mbps [E] | H.264 4 to 6 Mbps |
| Dropped or duplicated frames | At most 2 percent | 0 |
| Audio | 16 kHz, 16-bit, mono, at most 200 ms drift in 5 min [E] | AAC, 24 kHz or more |
| Photo | 5 MP JPEG | 5 MP or better |
| Start latency, button to first frame | At most 1.5 s [E] | At most 0.5 s |
| Playable on Android | Natively, without transcoding | MP4 H.264 + AAC |

15 fps fails the minimum (judder with walking). 720p at 24 to 30 fps, well fed, passes.

## 2. What the XIAO ESP32-S3 Sense actually includes

| Item | Fact | Source |
|---|---|---|
| SoC | ESP32-S3R8, dual-core LX7 at up to 240 MHz, 2.4 GHz Wi-Fi + BLE 5.0 | [V1] |
| Memory | 8 MB PSRAM (on-chip), 8 MB flash | [V1] |
| Camera sensor | Store page says the board "now features" the OV5640 without autofocus, **but units may ship with OV5640 or OV3660 during the transition**; change log dated 2025/6/30 notes OV2640 to OV3660. Wiki says OV5640 (fixed focus, 2592x1944), OV3660 (2048x1536) or discontinued OV2640. **No sensor die revision is published** [U]. Check the chip ID at runtime (esp32-camera logs it) | [V1] https://www.seeedstudio.com/XIAO-ESP32S3-Sense-p-5639.html, [V2] https://wiki.seeedstudio.com/xiao_esp32s3_getting_started/ |
| Lens / FOV | **Not stated** on the store page, getting-started wiki or camera wiki [U]. Measure it | [V1][V2][V3] |
| Microphone | One digital PDM mic on the Sense board, GPIO41 data, GPIO42 clock; 16 kHz, 16-bit, mono only in the Arduino example; part number not named on the wiki [U] | [V4] https://wiki.seeedstudio.com/xiao_esp32s3_sense_mic/ |
| SD | microSD slot on the Sense board, FAT32 up to 32 GB; wired SPI-style (CS GPIO21, SCK 7, MISO 8, MOSI 9) per the pin page; community code also runs it as 1-bit SD_MMC (CLK 7, CMD 9, D0 8) | [V2], [V5] https://wiki.seeedstudio.com/xiao_esp32s3_pin_multiplexing/, [C] https://github.com/s60sc/ESP32-CAM_MJPEG2SD/issues/219 |
| Camera bus | 8-bit DVP: XCLK GPIO10, PCLK 13, VSYNC 38, HREF 47, 8 data lines, SCCB on GPIO39/40 | [V5] |
| Charger | Li-ion charge on board; the wiki charge-current table shows 50 mA and 100 mA fast-charge rows that the extract did not assign to the Sense variant [U] | [V2] |
| Size | 21 x 17.8 mm; **15 mm thick with the Sense board**. The Pendant target is 12 mm deep (`processors.md`) | [V1] |
| Power (official, Sense board) | Idle 38 mA at 5 V; Wi-Fi active 110 mA; webcam web app average about 140 mA, peak about 347 mA at 5 V; mic + SD writing average about 55 mA; deep sleep about 3 mA | [V2] |
| Operating temperature | -20 to 65 C (ambient rating, not skin limit) | [V2] |
| Price / stock | USD 18.90; stock not stated | [V1] |

Espressif's esp32-camera driver lists OV2640, OV3660 and OV5640, with PSRAM required except for CIF or smaller JPEG; `jpeg_quality` runs 0 to 63, lower is better; it recommends JPEG capture because YUV/RGB writes to PSRAM are slow [V: https://github.com/espressif/esp32-camera]. The OV5640 does 1080p at 30 fps and 720p at 60 fps **on the sensor** [V: https://ovt.com/?p=944, via search summary]; the sensor is not the limit, the ESP32-S3 DVP/PSRAM path is.

## 3. What it can actually record

**No official source gives sustained 720p MJPEG recording to SD with audio on this board.** What exists:

| Claim | Value | Class |
|---|---|---|
| Espressif: ESP32-S3 DVP JPEG, OV5640 | 1600x1200 at 25 fps (sensor/driver table; not SD, not audio) | [V] https://docs.espressif.com/projects/esp-techpedia/en/latest/esp-friends/solution-introduction/camera/dvp-mipi-csi-camera-solution.html |
| Espressif: esp-rtsp example on ESP32-S3, 720p MJPEG stream | "around 20 fps"; sensor, JPEG quality and PSRAM mode not stated; streaming, not recording | [V], same page |
| Espressif: ESP32-S3 has no hardware encoder; uses a software JPEG encoder for non-JPEG sensors | | [V], same page |
| Seeed forum: Camera Webserver on S3 Sense + OV5640 | Could not exceed about 25 fps; reply blames bus/PSRAM bandwidth and HTTP overhead; resolution unstated | [C] https://forum.seeedstudio.com/t/xiao-s3-sense-with-ov5640-framerate/295717 |
| MJPEG2SD (ESP32-S3, OV5640, recording to SD as AVI) | **FHD (1080p) records at 6 fps**; QXGA and larger 4 to 5 fps; the README has no HD (720p) row for the OV5640 | [C] https://github.com/s60sc/ESP32-CAM_MJPEG2SD |
| MJPEG2SD, OV2640 on original ESP32, 20 MHz | HD: camera max 12.5 fps, recorded 5 fps; S3 is about 2x faster; S3 with OV2640 records at sensor max with audio except UXGA at 10 fps | [C], same repo |
| Same project: audio | 16-bit mono PCM, 16 kHz; PDM mics work only on S3; OV5640 needs a heat sink or it overheats | [C], same repo |
| Config/start time at 1280x720 on an OV5640 + S3 | about 0.8 to 0.9 s to configure, VSYNC more than 600 ms after configuration | [C] https://github.com/espressif/esp32-camera/issues/651 |
| SD card speed | A no-name Class 6 card was 3x slower than a genuine one; 4-line SD mode about doubles speed on S3 (the Sense slot is wired for 1-bit or SPI) | [C], MJPEG2SD README |

**Reading:** at 720p the likely range is **about 10 to 20 fps recorded, with audio, to SD** [E, interpolated from 6 fps at 1080p, 5 fps at HD on a slower chip, and Espressif's about 20 fps stream figure]. That is below the 24 fps minimum. JPEG quality, bus load and SD hiccups set the frame drops, and none is published.

**SD write limits:** MJPEG at 12 to 24 Mbps needs 1.5 to 3 MB/s sustained [E]. A SPI or 1-bit SD interface plausibly sustains that on a good card, but it has no published margin. Frame stalls come from SD garbage collection, which buffering in PSRAM (as MJPEG2SD does) mitigates [C].

**Is esp_h264 usable on the S3?** Not for video. Espressif's own figure for the ESP32-S3 software encoder (openh264-based, baseline) is **320x240 at 11 fps**; its "use tips" blog gives the same ceiling, and the component README lists 320x192 I420 at 17.5 fps and 320x240 YUYV at 11.2 fps on ESP32-S3R8, with about 1 MB memory [V: https://developer.espressif.com/blog/2025/07/esp-h264-use-tips/ and https://raw.githubusercontent.com/espressif/esp-h264-component/master/esp_h264/README.md]. Scaling by pixel count, 1280x720 is about 12 to 15 times the pixels of 320x240, so **about 1 fps or less** [E]. Hardware H.264 exists only on the ESP32-P4 [V, same blog]. Conclusion: **MJPEG only**.

**Android playback is a hidden cost.** Android's platform media table lists H.264, H.265, VP8, VP9, AV1 and MPEG-4 SP video, and MP4/3GP/MKV/WebM containers. It does not list Motion JPEG or AVI [V: https://developer.android.com/media/platform/supported-formats]. Many phones play MJPEG anyway, but it is not guaranteed [E]. The app would need to remux or transcode, which adds phone-side work. Wire-format and player support would have to be tested.

## 4. File sizes, Transfer time, storage, battery

Assumptions [E]: 720p MJPEG frame 50 to 100 KB (Espressif cites 30 to 50 KB per 480x320 frame at 20 fps for a low-resolution case [V, techpedia page above]); H.264 720p30 at 2.5 Mbps, 1080p30 at 5 to 8 Mbps (the repo's earlier assumption is 8 to 16 Mbps for 1080p, `android-transfer.md`). Transfer rate 20 Mbps (the ESP32-S3 figure used in `android-transfer.md`); 35 Mbps is my guess for a dual-band AMB82-Mini [E]. Decimal MB.

| Format | Bitrate | MB per minute | Minutes per GB | Transfer s per min of Clip at 20 Mbps | at 35 Mbps |
|---|---|---|---|---|---|
| XIAO MJPEG 720p15, 50 KB/frame | 6 Mbps | 45 | 22 | 18 | 10 |
| XIAO MJPEG 720p15, 100 KB/frame | 12 Mbps | 90 | 11 | 36 | 21 |
| XIAO MJPEG 720p24, 70 KB/frame | 13 Mbps | 101 | 10 | 40 | 23 |
| XIAO MJPEG 720p24, 100 KB/frame | 19 Mbps | 144 | 7 | 58 | 33 |
| XIAO MJPEG 720p30, 100 KB/frame | 24 Mbps | 180 | 5.6 | 72 | 41 |
| AMB82 H.264 720p30 | 2.5 Mbps | 19 | 53 | 7.5 | 4.3 |
| AMB82 H.264 1080p30 | 5 Mbps | 38 | 27 | 15 | 8.6 |
| AMB82 H.264 1080p30 | 8 Mbps | 60 | 17 | 24 | 14 |

Reading: at the same viewing quality, MJPEG is 4 to 8 times the H.264 file [E, intra-only vs inter-frame]. A 32 GB FAT32 card (the XIAO limit [V2]) holds about 5 to 6 hours of 720p24 MJPEG vs about 28 hours of 720p30 H.264 [E]. Transfer of XIAO Clips takes 2 to 8 times longer, and each second of Transfer is a second of Wi-Fi heat.

**Battery per 1-minute Clip** (record + Transfer), 1000 mAh cell with 3.0 Wh usable (`power-thermal.md`) [E]:

| | Record | Transfer | Energy per Clip | Clips per charge |
|---|---|---|---|---|
| XIAO | 0.7 W x 60 s = 42 J | about 0.8 W x 40 s = 32 J | about 74 J | about 145 |
| AMB82-Mini | 1.5 W x 60 s = 90 J (unmeasured) | 2.5 W x 8 to 15 s = 20 to 38 J | about 110 to 130 J | about 85 to 100 |

XIAO draws less per second but transfers longer. Both give hours of battery, so **energy is not the limit; heat is**.

## 5. Power and heat

**Official / measured:**

- Seeed's own table for the Sense board: webcam web app averages about 140 mA at 5 V (about 0.7 W), peak about 347 mA (about 1.7 W); Wi-Fi active 110 mA; mic + SD writing about 55 mA average [V2]. The webcam test resolution is not stated and the OV5640 row of the camera-current table says TBD [V2].
- Seeed warns the board "will get very hot" in long streaming, the OV5640 can exceed 70 C in continuous use, and one register tweak reduces it to about 54 C with a risk of noise [V3: https://wiki.seeedstudio.com/xiao_esp32s3_camera_usage/]. These are board or sensor temperatures, not skin temperatures, but they are far over the 48 C skin limit.
- The community MJPEG2SD README says the OV5640's supply is too high for its internal regulator, so it overheats without a heat sink [C].
- AMB82-Mini: no recording power published. Official "Normal mode" is 53 to 84 mA for the board at 5 V or 3 V, with video and Wi-Fi state not defined [V: https://ameba-doc-arduino-sdk.readthedocs-hosted.com/en/latest/ameba_pro2/amb82-mini/Example_Guides/PowerMode/Deep%20Sleep%20Mode.html]. Treat as unknown [U]; `power-thermal.md` assumes 1 to 3 W for 1080p30.

**Time to 48 C**, using the lumped model in `power-thermal.md` (G = 0.021 W/K, C = 12 J/K, ambient 30 C at the chest, time constant about 580 s) [E]:

| Sustained dissipation | Steady temperature | Time to 43 C | Time to 48 C | Spreader doubling G to 0.042 W/K |
|---|---|---|---|---|
| 0.4 W | 49 C | 11 min | 28 min | stays at 39.5 C |
| 0.6 W (XIAO webcam average, low end) | 59 C | 6 min | 9.5 min | 44 C, never reaches 48 |
| 0.7 W (XIAO webcam average, my recording estimate) | 63 C | 4.7 min | 7.4 min | 46.7 C, never reaches 48, above 43 design target |
| 0.9 W (XIAO with heavy Wi-Fi or SD) | 73 C | 3.4 min | 5.2 min | 51 C, reaches 48 in 8.7 min |
| 1.5 W (AMB82-Mini, unmeasured) | about 100 C | about 2 min | about 3 min | about 66 C, reaches 48 in about 3.3 min |

(Steady and times for 1.5 W are extrapolated from the same formula.) **Single 30 s to 60 s Clips are safe on both**: 0.7 W for 60 s is about 42 J, a rise of about 3.5 K on a 12 J/K body [E]. The limit bites only for back-to-back or continuous recording, or a long Transfer.

**Does a heat spreader change the answer?** Only at the margin. A spreader that doubles effective heat-shedding area (about 20 cm2 more, thermally coupled to the chain) would let a 0.6 to 0.7 W XIAO run indefinitely below 48 C, but not below the 43 C design target, and it does nothing for a source over about 0.75 W [E]. It also assumes the chain and clothing really conduct heat away, which is untested [U]. For the AMB82-Mini the answer depends on its unmeasured recording power; at 1.5 W a spreader only delays the limit by about half a minute. Neither board passes the 0.3 to 0.4 W continuous budget without duty-cycling.

## 6. XIAO ESP32-S3 Sense vs AMB82-Mini

| Criterion | XIAO ESP32-S3 Sense | AMB82-Mini |
|---|---|---|
| Delivered video | MJPEG only; 720p about 10 to 20 fps [E], 1080p 6 fps [C]; no H.264 (esp_h264 is 320x240 at 11 fps) [V]; AVI/MJPEG not in Android's platform format list [V] | H.264/H.265/JPEG hardware encode, 1080p30 + 720p30, up to 1080p45 [V: `processors.md` S1]; MP4 with audio to SD example, default 1080p30, 30 s files [V: https://how2electronics.com/1080p-mp4-video-recorder-with-amb82-mini-iot-ai-camera/, third-party] |
| Photo quality | OV5640 up to 2592x1944 (5 MP) JPEG [V1][V]; or OV3660 3 MP; no ISP tuning | Shipped JXF37P camera (about 2 MP class); 5 MP OV5647 or GC5035 are possible but OV5647 lacks IQ tuning (`camera-sensors.md`) |
| Audio | One PDM mic, 16 kHz mono; recording with video unproven on official pages; works with MJPEG2SD [C] | One onboard mic with audio codec in the MP4 example [V third-party]; codec not confirmed [U] |
| Size | 21 x 17.8 x 15 mm [V1] | Board about 60 x 40 mm [C: retailer listing, e.g. https://www.desertcart.nz/products/628617210]; bare RTL8735B is 10 x 10 mm (`processors.md`) |
| Power | About 0.7 W webcam average [V2]; recording not published | Unmeasured [U]; assume 1 to 2 W [E] |
| Heat | Seeed: sensor above 70 C in long use [V3] | Unknown [U] |
| Transfer | Wi-Fi 2.4 GHz only, about 20 Mbps ceiling [V: Espressif]; MJPEG files 5x larger | Dual-band Wi-Fi + BLE 5.1 [V]; concurrent SoftAP + BLE unverified (`processors.md`) |
| SDK / openness | ESP-IDF, esp32-camera, Arduino; open source, huge community | Realtek SDK, non-OSI license, sensor drivers largely binary (`camera-sensors.md`) |
| Availability | USD 18.90 [V1]; widely stocked, OV5640 add-on sold by several retailers [C] | EUR 35.20 on backorder at one retailer on 2026-11-17 (`processors.md`) |
| Dev effort | Low to start; high to reach 720p24 reliably (tuning XCLK, JPEG quality, PSRAM, SD, drop handling) [E] | Medium: example gives MP4 recording; pendant needs power tuning, SoftAP/BLE test [E] |
| Path to custom wearable | Good: ESP32-S3 modules, 8-bit DVP, compact OV5640 modules; no SDK licensing issue [E] | Harder: bare SoC not sold in small quantity; module route [V: `processors.md`] |
| With processor in the Counterweight | **Poor**: parallel DVP bus (see below) | **Poor**: MIPI cable limit (see below) |

**Moving the processor to the Counterweight (ticket #10).** Both options are difficult; this favors keeping the processor in the Pendant.

- **XIAO / DVP.** The bus is 8 data lines + PCLK + HREF + VSYNC + XCLK + SCCB (SCL, SDA) [V5], about 14 signals, plus power, ground and PWDN/RESET, so about 18 or more conductors, with a 20 MHz XCLK and a pixel clock the S3 supports up to 40 MHz [V: Espressif FAQ, https://docs.espressif.com/projects/esp-faq/en/latest/application-solution/camera-application.html, via search summary]. A necklace cord runs about 0.5 m each way (`power-thermal.md`). Espressif gives no cable-length limit [U: techpedia page has none]. Community experience is that this bus misbehaves with long wires or weak supplies (artifacts at 20 MHz without a stable supply, noise reduced at 5 MHz and lower frame rate) [C: https://esp32.com/viewtopic.php?p=113565 via search summary; Seeed wiki lowers XCLK to 10 MHz to avoid errors [V3]]. A 18-conductor flexible cord against the neck is bulky and an EMI source; I rate it impractical [E]. Slowing the clock to cope costs frame rate, which the XIAO cannot spare [E].
- **AMB82-Mini / MIPI.** Realtek staff recommend an FPC under 100 mm for the MIPI link [C/V forum, `camera-sensors.md` source 12]. Carrying MIPI through the cord would need a serializer/deserializer pair, which adds power and parts [U, not researched].
- **Alternative that avoids both:** keep processor, sensor, mic, SD and radio in the Pendant, and send only battery power through the cord (as `power-thermal.md` assumes). Then heat is the question, not the cable. That is where the two boards differ least.

## Pass/fail thresholds for a cheap first test

XIAO (half a day, one board, one OV5640, one good 32 GB card, a USB power meter, a thermocouple on the board back). Test: MJPEG at 1280x720, XCLK 20 MHz, PSRAM buffers, SD recording with mic, Wi-Fi off, 5-minute indoor and one outdoor take, then a 20 s Transfer to a phone.

| Check | Pass | Fail |
|---|---|---|
| Average fps over 5 min | at least 24 | under 20 |
| Frame gaps, p95 | at most 70 ms | over 120 ms |
| Dropped/duplicated frames | at most 2 percent | over 5 percent |
| A/V drift in 5 min | at most 200 ms | over 500 ms |
| Start latency | at most 1.5 s | over 2.5 s |
| File size per minute | at most 100 MB | over 150 MB |
| Plays in Android's stock Gallery or an in-app player without transcoding | yes | no |
| Average power while recording (battery side) | at most 0.5 W | over 0.8 W |
| Board temperature after a 60 s Clip, from 25 C | at most +5 K | over +10 K |
| Photo | 2592x1944 JPEG, at most 1 s | not achievable |

Mark a **marginal** (20 to 24 fps, 0.5 to 0.8 W) as a fail for the Pendant, since there is no thermal headroom.

AMB82-Mini (same day): 720p30 H.264 + audio to SD for 5 min. Pass: 30 fps, 0 drops, file at most 25 MB/min, average power at most 1.0 W, board at most +10 K after a 60 s Clip, plays natively in Android, SoftAP Transfer at least 20 Mbps. Fail: power over 1.5 W or no concurrent SoftAP/BLE (`processors.md`).

## Evidence that would change the recommendation

- **Toward XIAO only:** the screening test passes every row above, **and** a Pendant mock with its measured power stays under 0.4 W average at a 10 Clips/hour duty cycle, **and** Android playback is solved cheaply (for example a remux step in the app).
- **Away from AMB82-Mini:** measured recording power above about 1.5 W, no stable concurrent SoftAP + BLE, or no stock at a usable quantity. Then test the ESP32-P4 option (hardware H.264 to 1080p30, `processors.md`) before accepting MJPEG.
- **Lower the spec to 15 fps:** only if a viewing test of real walking clips shows people accept it. The LIVE-YT-HFR finding on camera motion says they will not [V].
- **Raise the spec to 1080p:** if stabilization crop is added (capture at 1080p to deliver 720p) or if clips are meant for big screens.

## Open risks

1. **No official 720p SD recording benchmark** exists for this board. The 10 to 20 fps range is interpolated [E].
2. **Sensor revision and FOV** are not published; shipped units vary (OV5640 or OV3660) [V1].
3. **AMB82-Mini recording power and heat** are unmeasured. The comparison table's power gap is assumption-heavy.
4. **Thermal model** is the ticket-4 hand estimate (mass, h, area); real values need a mock. The 0.7 W recording figure is Seeed's streaming average, not a recording measurement.
5. **Perceptual evidence** is partly vendor or informal for resolution; the frame-rate evidence is peer-reviewed but uses streaming-video content, not chest-worn footage.
6. Several pages were read through a summarising fetcher; the Espressif FAQ and OmniVision claims came via search summaries, not full pages. The OmniVision product page itself returned 404.
7. Retail stock and price change daily.

## Sources

- [V1] Seeed store, XIAO ESP32S3 Sense: https://www.seeedstudio.com/XIAO-ESP32S3-Sense-p-5639.html
- [V2] Seeed wiki, getting started (specs, power, temperature): https://wiki.seeedstudio.com/xiao_esp32s3_getting_started/
- [V3] Seeed wiki, camera usage (heat warnings, formats): https://wiki.seeedstudio.com/xiao_esp32s3_camera_usage/
- [V4] Seeed wiki, microphone: https://wiki.seeedstudio.com/xiao_esp32s3_sense_mic/
- [V5] Seeed wiki, pin multiplexing (SD, DVP, mic pins): https://wiki.seeedstudio.com/xiao_esp32s3_pin_multiplexing/
- Espressif esp32-camera: https://github.com/espressif/esp32-camera
- Espressif esp32-camera OV5640 driver: https://github.com/espressif/esp32-camera/blob/master/sensors/ov5640.c
- Espressif DVP & MIPI-CSI camera solution: https://docs.espressif.com/projects/esp-techpedia/en/latest/esp-friends/solution-introduction/camera/dvp-mipi-csi-camera-solution.html
- Espressif camera FAQ: https://docs.espressif.com/projects/esp-faq/en/latest/application-solution/camera-application.html
- Espressif esp_h264 README: https://raw.githubusercontent.com/espressif/esp-h264-component/master/esp_h264/README.md
- Espressif esp_h264 use tips: https://developer.espressif.com/blog/2025/07/esp-h264-use-tips/
- OmniVision OV5640 launch note (OmniBSI, frame rates): https://ovt.com/?p=944
- Android supported media formats: https://developer.android.com/media/platform/supported-formats
- LIVE Mobile Video Quality Database: https://live.ece.utexas.edu/research/Quality/live_mobile_video.html
- LIVE-YT-HFR (Madhusudana et al., IEEE Access 2021): https://arxiv.org/abs/2007.11634
- Meta AI glasses video guide: https://www.meta.com/ai-glasses/learn/guide-shooting-pov/
- [V6] AMB82-Mini deep sleep and power docs: https://ameba-doc-arduino-sdk.readthedocs-hosted.com/en/latest/ameba_pro2/amb82-mini/Example_Guides/PowerMode/Deep%20Sleep%20Mode.html
- AMB82-Mini 1080p MP4 recording (third party): https://how2electronics.com/1080p-mp4-video-recorder-with-amb82-mini-iot-ai-camera/
- Community: MJPEG2SD https://github.com/s60sc/ESP32-CAM_MJPEG2SD and issue https://github.com/s60sc/ESP32-CAM_MJPEG2SD/issues/219; esp32-camera issue https://github.com/espressif/esp32-camera/issues/651; Seeed forum https://forum.seeedstudio.com/t/xiao-s3-sense-with-ov5640-framerate/295717; AT&T video optimizer https://developer.att.com/video-optimizer/docs/best-practices/resolutionandperception; phone display comparison https://www.digit.in/features/mobile-phones/smartphone-display-test-resolution-vs-quality-27306.html; Insta360 GO 3 coverage https://www.gsmarena.com/insta360_go_3_specs_features_price_sale_date-news-58990.php; EIS crop explainers https://ymcinema.com/2018/10/01/hypersmooth-gopro-hero7-black-eis-electronic-image-stabilization-explained/ and https://www.coolblue.be/en/advice/how-does-the-image-stabilization-of-your-action-camera-work.html; AMB82-Mini retail size https://www.desertcart.nz/products/628617210
