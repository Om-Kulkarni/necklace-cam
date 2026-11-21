# Camera sensors and modules for the Pendant (ticket #3)

Status: research only. Tags: **[V]** verified in the cited primary source, **[I]** inferred from a primary source, **[E]** my estimate, **[U]** unverified. Untagged figures come from the numbered source given.

## Recommendation

**Camera Module 3 (IMX708) is not the wearable choice.** It is a good Raspberry Pi reference camera, but at 25 x 24 x 11.5 mm (12.4 mm for Wide) [1] it fills the whole 12 mm Pendant depth, and the project processor (RTL8735B, see `processors.md`) has no IMX708 support in its SDK (see "Fit with RTL8735B and pendant" below). The comparison further down is kept as a Raspberry Pi reference.

- **First AMB82-Mini dev-board test:** the camera that ships with the board (reported as JXF37P, SDK sensor `SENSOR_F37`). Realtek staff call it the officially tested default. [8][9]
- **Wearable v1 candidate:** a compact OV5647 autofocus module of about 8.5 x 8.5 x 5 mm (for example Hampo H7MA-OV5647), for fit, subject to the image-quality and driver risks in the new section. [10][11]
- A wide view (about 100 degrees horizontal) remains the safer match for a chest-level camera (see "Field of view").

## Comparison (Raspberry Pi reference modules)

| Module | Sensor / resolution | Size (mm) | Optics, FoV (H x V) | Focus | Video modes | Lanes / connector | Driver |
|---|---|---|---|---|---|---|---|
| **RPi Camera Module 3** | IMX708, 4608x2592, 1.4 um pixels [1][2] | ~25x24x11.5 [1][2] | f 4.74 mm, 66x41 deg [2] | PDAF + CDAF, ~10 cm to inf [2] | 1080p50 [1]; 2304x1296p56, 2304x1296p30 HDR, 1536x864p120 [2] | 2 lanes [4]; 15-pin 1 mm FPC, 200 mm cable [1] | RPi kernel + libcamera [1][4] |
| **RPi Camera Module 3 Wide** | IMX708 | ~25x24x12.4 [1][2] | f 2.75 mm, 102x67 deg [2] | PDAF + CDAF, ~5 cm to inf [2] | same [2] | same | same |
| RPi Camera Module 2 | IMX219, 3280x2464, 1.12 um pixels (page says IMX355 from Nov 2026) [2] | ~25x24x9 [2] | f 3.04 mm, 62.2x48.8 deg [2] | Fixed, adjustable [2] | 1080p47, 1640x1232p41 [2] | lanes [U] | Upstream `imx219.c` exists [5] |
| RPi HQ Camera | IMX477, 4056x3040, 1.55 um pixels [2] | 38x38x18.4 excl. lens [2] | Depends on C/CS or M12 lens [2] | Manual [2] | 2028x1080p50 [2] | lanes [U] | Not in upstream listing [5] |
| Waveshare IMX708-120 | IMX708 [6] | board size only in images [U] | f 3.08 mm, 120 deg diagonal, <3.5% distortion [6] | AF, ~15 cm to inf [6] | not listed [6] | connector not stated; "compatible with Camera Module 3 baseboard" [6] | libcamera stated by vendor [6] |
| Arducam IMX462 (1080p, low light) | IMX462, 1920x1080 | 24x25 from a search snippet only [U] | wide M12 lens [U] | [U] | [U] | [U] | vendor stack [U]; imx462 not in upstream listing [5] |

Notes:
- Arducam product pages returned HTTP 403 to my reader, so Arducam figures are [U] and not used in the recommendation.
- I found no open Sony datasheet for IMX708/IMX219/IMX477; sensor facts come from Raspberry Pi and Waveshare pages.

## Evidence by criterion

- **Size and lens height:** Module 3 PCB and mounting holes match Module 2; Z is several mm taller due to optics [1]. Numbers in [1][2].
- **Resolution and video:** 11.9 MP, 1080p50 listed as a common mode [1]. Driver modes: 4608x2592, 2304x1296 binned, 1536x864, plus one 2304x1296 HDR mode [4]. The 1080p-or-better clip target and 12 MP photos are met.
- **Low light:** Back-illuminated stacked sensor with high SNR per the vendor brief [1]; 1.4 um pixels versus 1.12 um on IMX219 [1][2]. No measured low-light data from a primary source [U]. Binned 2304x1296 should be cleaner than full resolution [E].
- **Autofocus:** Phase-detect AF supported by libcamera [1]; the driver handles PDAF data [4]. Behaviour while the wearer moves is untested [U].
- **Interface:** CSI-2; the driver rejects anything but 2 data lanes [4]. I2C fast mode and fast-mode-plus control, with the focus actuator also on 2-wire serial [1].
- **Power:** Driver requests four supplies (analog 2.8 V, analog 1.8 V, digital 1.1 V, interface 1.8 V) [4]; module current draw not given in [1][2] [U]. Operating temperature 0 to 50 deg C [1]; a body-worn pendant should stay inside that [E].
- **Availability:** Production until at least January 2030 [1]. Waveshare lists a variant at roughly USD 20 to 26, stock not shown [6].
- **Drivers:** `imx708.c` is GPL-2.0, Copyright Raspberry Pi Ltd, in the Raspberry Pi kernel tree [4]. When checked, it was absent from the upstream `drivers/media/i2c` listing, which does contain imx219, imx290, imx296, imx415, imx678 and ov5647 [5]. IMX708 therefore depends on the Raspberry Pi kernel fork. The lane limit is a hard check in source [4].

## Physical connection

- The module ships with a 200 mm ribbon and a 15 x 1 mm FPC connector [1]. Raspberry Pi hosts use a 15-pin connector (flagship models before Pi 5) or a 22-pin connector (Zero series, Compute Module IO boards, Pi 5 onward) [2]; Zero boards need an adapter FPC [1].
- The 22-pin 0.5 mm pitch is not stated on the pages read [U].
- The cable leaves the bottom edge of the board [1]. A 200 mm cable is far longer than a pendant needs; plan a shorter cable. Bend radius and connector height inside a 12 mm enclosure are not published [U].
- Factory-assembled constraint: the stock module avoids custom camera PCB assembly. A bespoke board using the IMX708 lens module would need a vendor [U].

## Field of view for a chest-level wearable [E]

No primary source covers this; this is my geometry.
- At 1.5 m, 66 x 41 degrees covers about 2.0 m wide by 1.1 m high; 102 x 67 degrees covers about 3.7 m by 2.0 m.
- A chest camera sits below eye level and tilts with posture, so the Wide variant's extra vertical coverage forgives aiming error and still catches faces.
- Costs of Wide: more edge distortion (the 120 deg Waveshare variant quotes <3.5% [6]), a taller module [1], and a nearer focus limit of about 5 cm [2].

## Fit with RTL8735B and pendant

Added after the processor choice (`docs/research/processors.md`). Question: which cameras does the RTL8735B SDK actually support, and which can fit the Pendant?

### What the SDK supports

- Realtek's `sensor.h` in the official `ameba-rtos-pro2` SDK lists about 90 sensor IDs. It does **not** list Sony IMX708, IMX219 or IMX477. [7] [V]
- Sony entries are IMX307, IMX327, IMX662, IMX471, IMX681 (plus 12 MP sequence variants) and IMX775. Other families: SmartSens SC2333/SC2336/SC231H/SC2310/SC301/SC3336/SC400AI/SC450AI/SC485SL/SC5356; GalaxyCore GC1084/GC2053/GC2083/GC2093/GC3003/GC4023/GC4103/GC4653/GC4663/GC4693/GC5035/GC05A2/GC8613; SOI F35/F37/F38/F51/F53/F55/F57/K05/K06A/K306P/K351; OmniVision OV2735/OV5647/OV5693/OV7251/OV9734/OS02H10/OS04A10/OV13B10/OV12890/OV50A40/OS04D10/OS04E10; PrimeSense PS5258 to PS5420. [7] [V]
- A table in `sensor.h` marks, per sensor, whether a normal driver, fast-camera-start (FCS) driver, ISP image-quality (IQ) tuning, HDR and bidirectional I2C exist. GC2053, GC4653, SC2336, F37 and GC5035 have normal driver + FCS + IQ. **OV5647 has a normal driver only: no FCS, no IQ, no HDR.** [7] [V]
- Default frame sizes in `sensor.h`: GC2053 1920x1080@30, SC2336 1920x1080@30, GC4653 2560x1440@24, GC5035 2592x1944@15, SC5356 2592x1944@15, OV5647 2592x1944@15, IMX327 1920x1080@24. `USE_SENSOR` defaults to GC2053. [7] [V]
- Sensor drivers ship mostly as prebuilt binaries (`sensor_*.bin`, `iq_*.bin`); only a few driver sources (GC4693_HD, K306P_HD, PS5270, PS5420) are in the tree. Sensor selection is a `sensor.h` edit plus a clean build. [7][12] [V]
- Chip: MIPI CSI-2 with four data lanes, up to 5 MP JPEG, H.264/H.265 encode. [15] [V] Realtek staff said AMB82-Mini "should work" with a 2-lane camera, not guaranteed. [14] [V, forum]
- The SDK has an `ai_glass` scenario template (recording, snapshot, Wi-Fi) with IQ files named for SC5356 fixed-focus lenses. I found no module part number or size for them. [19] [V for files, U for modules]
- Staff: JXF37 is officially tested and the default; other sensors work "if they appear in sensor.h"; new sensors are case by case. [8] [V, forum]. Another staff reply says the Arduino SDK can use GC5035. [18] [V, forum]

Wiring constraint: AMB82-Mini needs a **24-pin, 0.5 mm MIPI connector with a Realtek-specific pin map**; 22-pin and 15-pin Raspberry Pi cables are not compatible. Staff recommend FPC length under 100 mm. [12][13] [V, forum] The pin map is only in images I could not read [U]. A custom wearable board can choose its own connector.

### Buyable compact modules

Dimensions are as printed by the seller; none came from an SDK-vendor datasheet. Arducam pages returned HTTP 403, so they are not covered.

| Module | Sensor, SDK status | Size (mm) | Optics | Focus | Lanes / FPC | Power | Small-qty buy |
|---|---|---|---|---|---|---|---|
| **Hampo H7MA-OV5647** [10] | OV5647 5 MP; normal driver only, no IQ [7] | Intro says 8.5 x 8.5 x 5; spec table says lens 8.5 x 8.5 x 4.97 and module 15.6 x 8.5 (inconsistent, no TTL stated) | EFL 3.2 mm, F2.8, 70 D / 58.6 H / 45.3 V deg (full 5 MP frame) | Autofocus (SDK motor support [U]) | Lanes and FPC pins [U] | [U] | MOQ [U]; quote needed |
| "2MP GC5035 mini AF" (H&L Electrical SF-SX50350005A) [17] | GC5035; driver + FCS + IQ [7] | Not stated | EFL 2.27 mm, F2.2, 76 deg diagonal | Autofocus | [U] | [U] | MOQ 3, EUR 7.84 each (3 to 99); listing has errors (says "21MP") |
| Hampo MFA16-GC2053 [16] | GC2053; driver + FCS + IQ [7] | 40.34 x 22.30; lens body 13.65 x 13.65 x 18.57 | EFL 3.12 mm, F1.9, 149.8 D / 115.6 H / 57.6 V deg | Fixed | 2 lanes; 0.5 and 1 mm FPC, mating FH12-24S-0.5SH | [U] | MOQ [U]; samples 3 days |
| Dogoozx GC4653 4 MP via Oz Robotics [OzR] | GC4653; driver + FCS + IQ [7] | L 33 (customisable); height [U] | "Optional lenses" | [U] | [U] | [U] | USD 16.20; ships from China, USD 59.80 minimum shipping |
| AMB82-Mini shipped camera (Taiyee module, JXF37P) [8][9] | SOI F37; driver + FCS + IQ [7] | [U] | [U] | [U] | 24-pin to board [12] | [U] | Included with board; EUR 35.20, backorder at one retailer [RS] |
| Vision Components IMX327 module [13] | IMX327; driver + IQ, no FCS [7] | [U] | [U] | [U] | Staff: needs 24-pin pin-for-pin match [13] | [U] | [U] |

No module with a verified total depth of 6 mm or less and a published datasheet was found. The nearest is the OV5647 module: the seller's own text says about 5 mm. If that figure is lens plus board, it meets the 6 mm target [U].

### Pendant depth implied [E]

My stack estimate, not a drawing: front window about 1 mm, camera module 5 mm (seller claim), processor PCB plus 1.5 mm RTL8735B package [15] plus parts about 2.5 to 3 mm, rear shell about 1 mm. That is **roughly 9.5 to 10.5 mm** with the OV5647 module, leaving about 1.5 to 2.5 mm of margin in 12 mm. A 40 x 22 mm GC2053 module with an 18.57 mm lens body [16] would not fit. Camera Module 3 would need roughly 15 to 16 mm [E]. XY size of the GC5035 and GC4653 modules is unknown [U].

### Picks

1. **First AMB82-Mini test: the shipped JXF37P camera.** Officially tested, FCS + IQ in the SDK, no adapter, 1080p-class. It gives real power, thermal and file-writing numbers before any camera change. [7][8][9]
2. **Wearable v1 candidate: compact OV5647 AF module (about 8.5 x 8.5 x 5 mm)**, because it is the only SDK-listed sensor with a seller-stated depth near 5 mm. A candidate, not a decision.
3. **Image-quality alternative:** a factory-made GC5035, GC2053 or SC2336 module, since those have FCS and IQ files in the SDK. Ask for a drawing at 6 mm TTL or less.

### Risks specific to this fit

1. **OV5647 has no IQ tuning file** in the SDK, so exposure and white balance may run on a default or dummy IQ. Staff say a dummy IQ is typically used when none exists [21]. Image quality and low light may be poor [I].
2. **1080p path for OV5647:** the SDK default is 2592x1944@15 [7]. The Linux driver's 1080p30 mode is a centre crop of the full frame [11], which would narrow the quoted 58.6 deg horizontal view to roughly 44 deg [E]. Whether Realtek's binary does the same is unknown [U]. That may be too narrow for a chest camera.
3. **Autofocus:** I found no statement on voice-coil motor support in the SDK [U]. A fixed-focus variant avoids it.
4. **Pin map and connector:** the 24-pin Realtek map is required on the dev board; the Hampo FPC pinout is unpublished [U]. Plan an adapter, or an FPC designed for the custom board.
5. **Supply:** no stock level or minimum for Hampo [U].
6. **Wide view vs depth:** the only wide fixed-focus module with published optics (GC2053, 115.6 H deg) is far too large [16].
7. **Forum-only evidence** for 2-lane operation: staff said "should work" [14].

## Open risks (Raspberry Pi reference)

1. **Depth:** 12.4 mm module versus 12 mm pendant, leaving nothing for a PCB, battery interface or button. Biggest risk; a mechanical decision.
2. **Host coupling:** IMX708 needs the Raspberry Pi kernel stack and a 2-lane CSI host, which narrows the processor choice. Not upstream [5].
3. **Power and heat in sustained 1080p+ recording:** current and thermal data not published [U]; 50 deg C limit [1].
4. **Low light and autofocus while moving:** unmeasured [U].
5. **FPC mechanics:** cable length, connector height and bend in a 12 mm enclosure unspecified [U].
6. **Alternatives unverified:** Arducam data was blocked [U]; no compact MIPI alternative smaller than 25 x 24 mm was confirmed from a primary source.
7. **Product-page drift:** the Raspberry Pi page says Module 2's sensor changes to IMX355 in November 2026 [2]; check the variant before ordering.

## Sources

1. Raspberry Pi Camera Module 3 product brief, June 2024 (read via redirect from https://datasheets.raspberrypi.com/camera/camera-module-3-product-brief.pdf): https://pip-assets.raspberrypi.com/categories/786-raspberry-pi-camera-module-3/documents/RP-008151-DS-1-camera-module-3-product-brief.pdf
2. Raspberry Pi camera hardware documentation: https://www.raspberrypi.com/documentation/accessories/camera.html
3. (unused number reserved; see [1] page 3 for the production-lifetime statement)
4. Raspberry Pi Linux `imx708.c`, branch rpi-6.12.y: https://github.com/raspberrypi/linux/blob/rpi-6.12.y/drivers/media/i2c/imx708.c
5. Upstream Linux `drivers/media/i2c` directory listing, checked via the GitHub contents API: https://github.com/torvalds/linux/tree/master/drivers/media/i2c
6. Waveshare IMX708 module page: https://www.waveshare.com/product/raspberry-pi/boards-kits/imx708-120-camera-module.htm
7. Realtek `sensor.h` (ameba-rtos-pro2; also read `component/video/driver/RTL8735B/` and `scenario/ai_glass` in the same repo): https://github.com/Ameba-AIoT/ameba-rtos-pro2/blob/main/project/realtek_amebapro2_v0_example/inc/sensor.h
8. Realtek forum, tested modules (staff): https://forum.amebaiot.com/t/what-camera-modules-not-just-sensors-are-tested-and-recommended-for-amebapro2/4545
9. Realtek forum, onboard sensor identification (staff): https://forum.amebaiot.com/t/amb82-mini-onboard-camera-sensor-identification-and-voe-setup-for-wireless-video-and-i2c/4777
10. Hampo OV5647 AF MIPI module: https://www.hampotech.com/5mp-ov5647-sensor-auto-focus-mipi-camera-module-product/
11. Linux `ov5647.c` (GPL-2.0, 1080p30 centre-crop mode): https://github.com/torvalds/linux/blob/master/drivers/media/i2c/ov5647.c
12. Realtek forum guide, adding sensors (24-pin 0.5 mm, sensor.h, FPC under 100 mm): https://forum.amebaiot.com/t/guide-add-new-camera-sensor-module-options-to-amebapro2-amb82-mini/4640
13. Realtek forum, IMX327 compatibility (24-pin match): https://forum.amebaiot.com/t/imx327-compatiblity/4796
14. Realtek forum, 2-lane question (staff): https://forum.amebaiot.com/t/can-amb82-mini-work-with-imx681-using-only-2-mipi-lanes/4552
15. RTL8735B datasheet Rev 1.0 Lite (retailer-hosted copy; four-lane CSI-2, 5 MP JPEG, 10 x 10 x 1.5 mm): https://electronicscomp.com/image/catalog/Realtek-AMB82-Mini-DATASHEET.pdf
16. Hampo GC2053 micro fixed-focus MIPI module: https://www.hampotech.com/2mp-gc2053-micro-fixed-focus-mipi-camera-module-product/
17. Alibaba GC5035 mini AF listing: https://www.alibaba.co.uk/product-detail/2MP-GC5035-CMOS-Image-Sensor-MIPI_1601517083145.html
18. Realtek forum, 5 MP question (staff mention GC5035 for Arduino SDK): https://forum.amebaiot.com/t/want-to-connect-5mp-good-quality-camera-with-amb82-mini/3087
19. Realtek SDK `ai_glass` scenario: https://github.com/Ameba-AIoT/ameba-rtos-pro2/tree/main/project/realtek_amebapro2_v0_example/scenario/ai_glass
20. Realtek FAQ, MIPI connection (four-line CSI-2, F37 pinout reference): https://ameba-doc-arduino-sdk.readthedocs-hosted.com/en/latest/FAQ/ameba-pro2_mipi_connection.html
21. Realtek forum, SC231H thread (dummy IQ when no file): https://forum.amebaiot.com/t/rtl8735b-amebapro2-sc231hai-sensor-support-missing-iq-sc231h-bin-wdr-hdr-capability-and-hardware-reference-design/4980
- [OzR] Oz Robotics GC4653 module: https://ozrobotics.com/shop/hd-4mp-2k-mipi-camera-module/
- [RS] RobotShop AMB82-Mini listing: https://eu.robotshop.com/products/realtek-amb82-mini-iot-ai-camera-arduino-dev-1080p-sensor-multiple-io
