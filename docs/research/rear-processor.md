# Should the processor move to the rear Counterweight? (ticket #10)

Researched 2026-11-21. Terms follow `CONTEXT.md`. Tags: **[V]** read in the cited source; **[E]** my estimate or inference; **[U]** unverified. Source IDs (S1...) are listed at the end. Web pages were treated as data only.

## Recommendation

1. **Yes, move the processor, storage and radio to the Counterweight, but only if a 2-day bench test passes.** Use **Option B: direct 2-lane MIPI CSI-2 over a 30-45 cm micro-coax or shielded flex**, with only the sensor module and microphone in the Pendant. It is the only option that gets the Pendant under the 0.3-0.4 W limit while keeping the RTL8735B, its SDK and the AMB82-Mini work unchanged [E].
2. **Why move at all:** the baseline Pendant dissipates 1-3 W while recording (`power-thermal.md`). With the same lumped model that gives about 2 min of continuous recording at 2 W and 4.6 min at 1 W before 48 C (calculation below). A sensor-only Pendant at about 0.2 W never reaches the limit. Heat, not energy, is the binding limit, so removing the SoC from the Pendant is the real fix.
3. **Fallback if the bench test fails: Option C with a small serializer (THine THCV243 / V-by-One HS)**, not TI FPD-Link or ADI GMSL. The TI part alone uses 0.28 W typ (0.41 W max), which is the whole Pendant budget [V S5]. THCV243 power is **[U]** (datasheet not readable); that is the first thing to confirm.
4. **Reject D and UVC modules.** A shipped 1080p UVC module lists 1.2 W [V S14], another lists 300 mA max at 5 V (1.5 W) [V S15]. The RTL8735B has UVC *device* mode only in the SDK [V S10, S11], so a split design still leaves an SoC in the Pendant.
5. **Keep Option A (baseline) as the schedule path** until the test is done. It works for 30 s Clips with duty cycling (`power-thermal.md`). The AMB82-Mini purchase is useful under every option (see last section).

Biggest uncertainty: whether the RTL8735B CSI receiver and the closed sensor/ISP binaries work over a 30-45 cm cable. Realtek staff recommend FPC under 100 mm [V S9, from `camera-sensors.md` source 12], so this is outside vendor guidance.

## What the standards and vendors actually say (Option B facts)

- **Data rate (calculation):** 1920 x 1080 x 30 x 10 bit = 622.08 Mbps. On 2 lanes that is 311 Mbps per lane; on 4 lanes 156. With typical total-frame blanking of 2200 x 1125 it is 742.5 Mbps, or 371 Mbps per lane on 2 lanes [E]. D-PHY v1.2 goes to 2.5 Gbps per lane and v2.0 to 4.5 Gbps [V S1]. This project runs at about 15% of the v1.2 maximum.
- **Length:** MIPI's D-PHY page says D-PHY supports "interconnect lengths up to 4 meters" through the Alternate Low Power (ALP) feature, which uses only high-speed signal levels [V S1, S2]. ALP needs support in both sensor and receiver. I found no evidence the RTL8735B or its sensors support ALP **[U]**; assume they do not.
- **Classic D-PHY reach:** a MIPI blog says smartphone short-reach D-PHY/C-PHY "do not typically extend beyond approximately 25cm" at the highest data rates, and that integrators use proprietary bridges for longer runs [V S3]. The normative channel-loss numbers are in the members-only spec **[U]**.
- **Vendor products:** Kurokesu sells 4-lane CSI-2 slim FPC in 10/20/30/40 cm and an I-PEX micro-coax kit in 20 or 30 cm "for constant motion", with no data rate or flex-cycle figure [V S6]. These show 30-40 cm is sold, not that it is validated at any rate.
- **Why our rate helps [E]:** cable loss grows with frequency. If about 25 cm is the budget at 2.5 Gbps, the same loss at 0.4 Gbps allows roughly 3-6x the length (loss roughly proportional to frequency to the power 0.7-1). That suggests 45 cm is plausible on loss alone. The real risks are not loss: connector discontinuities, LP-mode (1.2 V single-ended) signalling over a long cable, skew, EMI, and the receiver's untested tolerance.
- **Control lines:** I2C to the sensor must run the same distance, at 100 kHz to be safe [E]. The SDK's closed sensor drivers must still initialise the sensor over that I2C; `sensor.h` marks which sensors have normal driver, fast camera start and IQ files [V S9].

## Options compared

Pendant power for B uses sensor-only power. Sensor figures: GC2053 98 mW at 1080p30 (module seller, not the sensor datasheet) [V S16]; OV2735 176 mW (seen in a search snippet of the vendor brief; the page I could fetch did not confirm it) [U S17]. I use 0.1-0.2 W for a sensor plus passives and 0.15-0.25 W with microphone and cable losses [E].

| | A. Baseline (all in Pendant) | B. Direct MIPI over micro-coax / flex | C. Serializer link (TI / ADI / THine) | D. Split: RTL8735B in Pendant, USB/SPI to rear | E. USB/UVC module |
|---|---|---|---|---|---|
| Pendant parts | SoC, DDR, flash, SD, Wi-Fi, antenna, sensor, mic | Sensor module, MIC, cable termination | Sensor, serializer (TI 5x5 mm; THCV243 2.1x2.9 mm), PoC parts | Sensor, RTL8735B, mic, link driver | UVC module (Arducam B0630: 19.5 x 15 x 17.3 mm) |
| Achievable Pendant size | 30 x 25 x 12 mm, tight [E] | About 20 x 15 x 8 mm, set by sensor module and cable strain relief [E] | B plus 25 mm2 (TI) or 6 mm2 (THine) [V S5, S7] | Nearly A, minus Wi-Fi, SD, antenna [E] | Module depth 17.3 mm already exceeds 12 mm [V S14] |
| Pendant power (continuous) | 1-3 W recording [power-thermal.md]; 2.5 W Transfer | **0.15-0.25 W [E]** | TI: 0.29 W typ, 0.41 W max serializer alone [V S5] + sensor = **0.4-0.6 W**. THine: [U], guess 0.1-0.25 W [E] | RTL8735B without Wi-Fi or SD: **0.5-1.0 W [E]**, unmeasured | **1.2 W [V S14]**; 1.5 W max [V S15] |
| Counterweight power | None (battery only) | SoC, SD, Wi-Fi: 2-3 W, same as A today [E] | Same, plus deserializer: TI page lists no power [U]; assume 0.3-0.5 W [E] | Host SoC plus Wi-Fi, SD: 1-2.5 W [E] | Host plus Wi-Fi, SD |
| Fits 0.3-0.4 W? | No (2 min at 2 W) | **Yes, margin 0.1-0.2 W [E]** | TI no; THine unknown | Probably no | No |
| Conductors, diameter | 2 power (power-thermal.md) | 3 pairs (CLK, 2 data) + I2C 2 + MCLK + power 2 + mic 2-3 + button 1-2: about **15-18 conductors**, about 3-4 mm round [E; no cable datasheet found] | TI: 1 coax (PoC) about 1.2 mm [E]. THine: 1 pair + 2 power + I2C if not tunneled: 4-6 conductors [E] | USB 2: 4 wires; SPI: 4-6 wires + power | USB 2: 4 wires |
| Flex life | Power wires only: easy | Micro-coax is rated for motion by vendors, no cycle counts found [V S6] **[U]**; stiff for the neck [E] | Thin coax or twisted pair: better than B [E] | Best (4 wires) | Best |
| Breakaway connector | 2 contacts, trivial | **Hard.** About 14+ contacts including 3 controlled-impedance pairs; no off-the-shelf rated part found. Custom spring-pin array [E] | Coax at 3-4 Gbps: custom too. Fewer contacts than B | **Easiest:** USB 2 magnetic connectors with 480 Mbit/s claims exist (Rosenberger, Amphenol listing) [V S12, S13] | Same as D |
| On disconnect | Power cut mid-write; MP4 may be corrupt [E] | Counterweight stays powered, closes file; sensor loss must not crash the CSI driver **[U]** | Same as B | Same as B | Same as B |
| EMI / radio coexistence | Radio and CPU together, as today | Cable carries 400-800 Mbps edges beside neck; Wi-Fi antenna 5-10 cm from the receiver in the Counterweight [E]; shield the cable, antenna on far face | Same, plus serializer harmonics | USB 2 480 Mbps is the same class as B [E] | Same |
| Antenna and body | Chest, outward facing; line of sight to a phone in front | **Behind the neck: body shadows a phone in front or pocket [E].** Keep 10 mm from skin [V S18]; a lossy block cost about 10 dB at 2.7 mm [V S19]; on-chest simulated efficiency 54% vs 85% free space [V S18] | Same as B | Same as B | Same as B |
| RTL8735B / SDK | Works today | Same SoC; needs 24-pin Realtek pin map adapter [V S9]; lane count on AMB82-Mini unverified, 2-lane likely OK [V S9] | Needs serdes register setup **before** sensor init; SDK sensor drivers are closed binaries, so this is a custom hack **[U]** | UVC **device** example `media_uvcd` exists [V S10, S11]. UVC **host** is not in the SDK [V S11]. Datasheet lists USB host [V S8] | Needs UVC **host**: absent from RTL8735B SDK [V S11]. Use a Linux or ESP host instead |
| PCB difficulty | Hard (fit) | Medium: two small boards, controlled-impedance cable | Medium to hard: BGA-like QFN, PoC filtering, PLL clocking | Medium | Easy (buy module) |
| Cost, personal build | Lowest | Low to medium; custom cable and connector are the cost **[U]** | Medium: TSER953/TDES954 orderable on ti.com (price and stock not shown) [V S5, S5b]; THCV243-B sampled via Digi-Key [V S7]; A-PHY (Valens VA7000) is engineering samples to automotive customers [V S4] | Medium | Low |
| Buildable personally? | Yes | **Yes, if connector solved** | TI/THine: with effort. A-PHY: no | Yes, but fails power | Yes, but fails power and fit |
| AMB82-Mini test still useful? | Fully | **Fully** (extend the camera FPC) | Yes for power, SD, Wi-Fi; serdes needs extra boards | Yes: UVC device test; power with Wi-Fi off | Partly: it becomes only the host-side power test |

### Option notes

**A. Baseline.** Lumped model from `power-thermal.md`: G = 0.021 W/K, tau = 580 s, ambient 30 C, so steady dissipation is G x 18 K = 0.378 W at 48 C and 0.273 W at 43 C. Time to 48 C is t = -tau ln(1 - 0.378/P): P = 2 W gives 122 s, 1 W gives 275 s, 0.7 W gives 450 s, 0.5 W gives 818 s, 0.4 W gives about 28 min. [E, hand model.]

**B. Direct MIPI.** Pendant power at 0.15-0.25 W stays below 0.378 W, so the model never reaches 48 C (steady state 7-12 K rise). The Counterweight takes the 2-3 W. Scaling its exposed area to a 60 x 35 x 15 mm shape (49 cm2 non-skin versus 20.7 cm2, 2.4x) gives G of about 0.05 W/K, so sustained about 0.9 W at 48 C and 0.65 W at 43 C, with 50 J/K heat capacity (tau 1000 s). At 2 W it reaches 48 C in about 10 min (versus 2 min for the Pendant) [E]. Continuous recording is therefore still thermally limited, only longer. Treat the Counterweight as a heat problem too: it sits against the neck and holds the Li-polymer cell. Firmware throttling and an NTC remain required.

**C. Serializers.**
- TI DS90UB953-Q1 / TSER953: 4.16 Gbps forward channel, up to 4 CSI-2 lanes at 832 Mbps, coax or STP, power-over-coax, 5 x 5 mm VQFN, single 1.8 V, 0.28 W typ [V S5]. IDD_TOTAL is 160 mA typ and 225 mA max at 1.8 V, which is 0.288 W and 0.405 W, measured with 4 lanes at 416 MHz clock [V S5]. Lower lane rates may use less **[U]**. The TSER953 version is rated -20 to 85 C [V S5b]; the -Q1 version -40 to 105 C [V S5].
- TI deserializer TDES954 / DS90UB954-Q1: 7 x 7 mm VQFN, output at 400/800/1500/1600 Mbps per lane. Our 311-371 Mbps per lane input must be re-timed to a rate the RTL8735B accepts **[U]** [V S5c].
- ADI MAX96717 (GMSL2): 5 x 5 mm TQFN, 4-lane CSI-2, fixed 3 or 6 Gbps link, AEC-Q100 [V S20, via search result summary]. A DigiKey forum post gives about 260 mW at 6 Gbps for a GMSL serializer in general, not this part at 3 Gbps [V S21, **U** for this part]. I could not read the datasheet power table.
- THine THCV243: 2.1 x 2.9 mm CSP35, CSI-2 input 1/2/4 lanes up to 1.2 Gbps per lane, V-by-One HS output up to 4 Gbps, supplies 1.7-3.6 V and 1.1-1.3 V, companion THCV242 deserializer [V S7, S22]. Power not found **[U]**. THine's page links sampling via Digi-Key [V S7a]. THine also sells a ready kit (THSER101, 20 m, Cat5e) for Raspberry Pi cameras, but its Tx board is 25 x 38 mm and Pi-specific [V S23].
- MIPI A-PHY reaches 15 m but is designed for automotive [V S4, S24]; the only silicon found is Valens engineering samples to automotive customers, so reject it for a personal build.
- **Integration risk [U]:** all serdes need register setup before the sensor is touched. On the RTL8735B this means code ahead of the closed sensor driver.

**D. Split design.** The RTL8735B datasheet lists USB 2.0 host and device, SD host, 10/100 RMII Ethernet, SPI master up to 25 Mbps and SPI slave up to 31.25 Mbps [V S8]. A 4 Mbps H.264 MP4 stream (vendor guidance, `processors.md`) fits SPI or USB easily. But the RTL8735B would stay in the Pendant, and the saving is only Wi-Fi TX and SD writes. Wi-Fi and BLE concurrent use is already a flagged risk. A UVC-device Pendant needs a host that can take H.264 UVC in the Counterweight (Linux SoC such as a Pi Zero 2 W [V `processors.md` S19]; ESP32 `usb_host_uvc` is documented for MJPEG, H.264 host support not confirmed [U S25]). That is a second software stack.

**E. Other options.**
- **UVC module in Pendant:** fails on power and depth (above). A custom low-power bridge to USB was not found.
- **Magnetic USB breakaway** is the only commercially available high-speed breakaway [V S12, S13], but it only helps USB-based designs.
- **Hybrid breakaway for B:** run the 2-contact power on the breakaway connector and make the data cable the strain-limited part, so it fails before the neck does. Needs lab verification; not acceptable without a measured release force [E].

## Winner: block diagram (Option B)

```
 PENDANT (target 0.15-0.25 W, about 20 x 15 x 8 mm)         NECKLACE CORD                COUNTERWEIGHT (2-3 W)
+--------------------------------------+     +-------------------------+     +----------------------------------------+
| Sensor module (GC2053 / F37 class)   |     | 3 shielded pairs/coax   |     | RTL8735B (AMB82-Mini class)            |
|  2-lane CSI-2, RAW10, 1080p30        |=====| CLK + D0 + D1 (100 ohm) |=====>| MIPI CSI-2 RX, ISP, H.264 encoder      |
| I2C (100 kHz)  <-------------------->|-----| SCL, SDA                |-----|  SD card (Onboard Storage)             |
| MCLK 24 MHz or local oscillator  <---|-----| 1 wire                  |-----|  Wi-Fi + BLE antenna (far face from    |
| PDM mic (CLK, DATA)  --------------->|-----| 2-3 wires               |---->|   neck, 10 mm clearance)               |
|                                      |-----|                         |     |  Charger, 3.7 V cell, 5 V -> 3.3 V     |
| 3.3 V LDOs for sensor rails  <-------|-----| 3.3 V + GND (2 wires)   |<----|  Power eFuse + NTC                     |
+--------------------------------------+     +----------+--------------+     +----------------------------------------+
                                                         |
                                       [BREAKAWAY CONNECTOR, about 14-18 contacts, 15-25 N release, custom]
                                       On release: sensor and mic lose power; Counterweight stays on,
                                       closes the MP4, restarts recording or stops on camera-lost watchdog.
```
Cable passes along one side of the neck only (25-45 cm); the other side is a plain strength member so the loop still opens at the connector [E].

## Risks

1. **CSI receiver tolerance at 30-45 cm is untested** and outside Realtek's under-100 mm advice [V S9]. Mitigation: the bench test below. If it fails, use C-THine.
2. **Breakaway connector with 3 controlled-impedance pairs** has no off-the-shelf answer; this is the largest buildability risk for B. Custom pogo array, stub length about 4 mm or less at 150 ps edge rates [E]. If it fails, B collapses to C, which also needs a custom coax connector.
3. **Closed sensor drivers and MCLK:** local oscillators may not work with Realtek's binaries **[U]**. Start with MCLK on the cable.
4. **Microphone over cable:** DMIC support on RTL8735B not verified **[U]**. Alternative is a mic in the Counterweight, worse for voice at the chest.
5. **Antenna behind the neck:** body shadowing of a phone in front. Mitigation: defer Transfer to off-body, or test an antenna on the Counterweight's top edge. No neck-specific absorption data found **[U]**.
6. **Flex life** of micro-coax unquantified [V S6]; cable stiffness and comfort **[U]**.
7. **EMI:** 622-742 Mbps edge content next to a 2.4 GHz radio; no measurement.
8. **Counterweight heat and cell temperature** are new loads; battery limits are in `power-thermal.md`.
9. **Pendant power figures are estimates.** No vendor publishes recording current for the RTL8735B (`processors.md`), and I could not confirm OV2735's 176 mW **[U]**.
10. **Safety and legal items** (breakaway force standard) remain open. v1 has no Recording Indicator.

## Cheap de-risking test (about 2 days, under 100 EUR [E])

1. Take the AMB82-Mini and its stock camera (JXF37P, 2-lane) on a 24-pin FPC adapter [V S9].
2. Build three extensions of the sensor lines: 10 cm FPC (control), 30 cm and 40 cm slim 4-lane FPC (for example Kurokesu 40 cm, whose price I did not find) or 30 cm I-PEX micro-coax [V S6].
3. Record 10 min at 1080p30 per length, indoors and with the cable flexed by hand and wrapped on a mannequin neck; log dropped frames, CSI errors, ISP resets and I2C failures. Add a deliberate disconnect and reconnect during recording.
4. Repeat with a ferrite or shield foil and with the Wi-Fi streaming on, to look for coexistence loss.
5. Add a mock breakaway: two 1.27 mm headers or spring pins with 5 mm stubs in each pair; rerun at 30 cm.
6. Measure the AMB82-Mini current with a DMM in series (vendor method: remove R62 and the log pins, from `processors.md` S4 and forum) during record, SD write and Wi-Fi Transfer. This feeds both the Counterweight and the Pendant budgets.

Pass: 0 errors in 10 min at 30 cm; clean recovery at disconnect. Fail at 30 cm: go to C with THCV243/THCV242 samples (after confirming power).

## Does the AMB82-Mini purchase still make sense?

Yes. Under A it is the test platform; under B it is the Counterweight processor and the cable test bed; under C it measures power, SD and Wi-Fi; under D it gives the UVC-device stream; under E it supplies the host-side numbers only. It is the cheapest way to answer the decisive question, which is its real power while recording. Risks about supply (backorder, bare SoC not sold in small quantities) are in `processors.md`.

## Sources

- S1 MIPI D-PHY spec page (4 m via ALP, v3.6 current): https://www.mipi.org/specifications/d-phy
- S2 MIPI blog, two new PHY versions (ALP, 4 m, 4.5/6 Gbps): https://www.mipi.org/blog/a-look-at-mipis-two-new-phy-versions
- S3 MIPI blog, CSI-2/DSI-2 in automotive (25 cm smartphone reach, proprietary bridges): https://www.mipi.org/blog/a-look-under-the-hood-at-mipi-csi-2-and-mipi-dsi-2-in-automotive
- S4 MIPI A-PHY (15 m, automotive); Valens sampling news: https://www.mipi.org/specifications/a-phy ; https://www.eenewseurope.com/en/first-samples-of-mipi-a-phy-chip/ (search result only)
- S5 TI DS90UB953-Q1 datasheet SNLS552 (read from PDF): https://www.ti.com/lit/gpn/DS90UB953-Q1
- S5b TI TSER953 product page and datasheet SNLS696: https://www.ti.com/product/TSER953
- S5c TI DS90UB954-Q1 / TDES954 pages: https://www.ti.com/product/DS90UB954-Q1 ; https://www.ti.com/product/TDES954
- S6 Kurokesu slim FPC and micro-coax for CSI-2: https://www.kurokesu.com/blog/slim-fpc-and-micro-coax-cables-for-csi-2-cameras
- S7 THine THCV243 news (package, 4-lane in, V-by-One HS out): https://www.eejournal.com/industry_news/thine-introduces-a-small-package-low-power-consumption-mipi-csi-2-serializer-ic-for-video-applications/ ; S7a THine SerDes page (Digi-Key sample link): https://www.thinesolutions.com/serdes-ic
- S8 RTL8735B datasheet Rev 1.0 Lite (USB 2.0 host and device, SD, RMII, SPI, 4-lane CSI): https://electronicscomp.com/image/catalog/Realtek-AMB82-Mini-DATASHEET.pdf
- S9 `docs/research/camera-sensors.md` (sensor.h, 24-pin map, FPC under 100 mm, forum sources 12-14)
- S10 Ameba Pro2 SDK USB app note (UVC device `media_uvcd`): https://ameba-doc-rtos-pro2-sdk.readthedocs-hosted.com/en/latest/application_note/19_USB.html
- S11 Realtek forum, UVC host not in sample set (staff): https://forum.amebaiot.com/t/using-rt8735b-for-wireless-transmission-of-external-uvc-to-raspberry-pi/4573
- S12 Rosenberger MagneticUSB (480 Mbit/s): https://www.digikey.hk/en/product-highlight/r/rosenberger/magneticusb-connector-solutions
- S13 Amphenol Gladiator magnetic connectors (USB 2.0, 90 ohm; found in a search result, page itself did not load): https://www.mouser.in/new/amphenol/amphenol-pcd-gladiator
- S14 Arducam B0630 datasheet (1.2 W, 19.5 x 15 x 17.3 mm, MJPG/YUY2/H.264): https://download.arducam.com/products/documents/B0630/20260611_140117/Arducam_Product_Datasheet_B0630.pdf
- S15 Arducam UB020202 datasheet (300 mA max): https://cdn.shopify.com/s/files/1/0176/3274/files/UB020202_IMX291_160_UVC_Camera_Datasheet.pdf
- S16 GC2053 module page (98 mW, seller figure): https://www.muchvision.com/mipi-camera-module/2mp-camera-module/gc2053-1080p-cmos-camera-module-30fps-low.html (search-result text only)
- S17 OmniVision OV2735 product brief (176 mW per search snippet, not confirmed): https://www.ovt.com/wp-content/uploads/2023/07/OV2735-PB-v1.4-WEB.pdf
- S18 On-body flexible antenna paper (54% on chest vs 85% free space, 10 mm gap, simulated): https://pmc.ncbi.nlm.nih.gov/articles/PMC8143588
- S19 Antenna-Theory, wearable antennas (about 10 dB loss at 2.7 mm over a lossy block): https://antenna-theory.com/antennas/wearable-antennas.php
- S20 ADI MAX96717 (package, lanes, link rates; via search result): https://www.analog.com/MAX96717/datasheet
- S21 DigiKey forum on GMSL PoC (about 260 mW at 6 Gbps, generic): https://forum.digikey.com/t/why-does-power-over-coax-poc-matter-for-robots-and-how-does-gmsl-use-it/63002/1
- S22 THCV243 summary (supplies, lanes): https://components101.com/news/thcv243-compact-and-low-power-mipi-csi-2-serializer-ic-for-endoscope-camera-applications
- S23 THine THSER101 datasheet (20 m, Tx board 25 x 38 mm): https://cdn.shopify.com/s/files/1/0176/3274/files/THSER101.pdf ; review: https://magazine.raspberrypi.com/articles/review-thine-cable-extension-kit
- S24 MIPI roadmap slides (D-PHY rates by version): https://www.mipi.org/hubfs/DevCon-2022/2022-MIPI-DevCon-PHY-Roadmap.pdf
- S25 Espressif `usb_host_uvc` component (formats not confirmed): https://components.espressif.com/component/espressif/usb_host_uvc
- Not read: The Imaging Source CSI-2 FPC length note (HTTP 403): https://www.theimagingsource.com/en-us/support/knowledge-base/mipi-csi-2-cameras-fpc-cable-length/
- Not verified: MIPI D-PHY normative channel-loss tables (members only); ADI datasheet power table; THCV243 datasheet; micro-coax OD and flex-cycle datasheets (I-PEX, Hirose, Molex).
