# Camera sensors and modules for the Pendant (ticket #3)

Status: research only. Tags: **[V]** verified in the cited primary source, **[I]** inferred from a primary source, **[E]** my estimate, **[U]** unverified. Untagged figures come from the numbered source given.

## Recommendation

Use the **Raspberry Pi Camera Module 3 Wide (Sony IMX708)** as the first-build camera. Fall back to the standard Camera Module 3 if a narrower view is preferred.

- It is the only candidate found with all of: 1080p-or-better video, autofocus, HDR, a first-party datasheet, a Raspberry Pi-maintained driver with libcamera support, a stated production life to at least January 2030, and ordinary retail availability. [1][2][4]
- **The 30 x 25 x 12 mm envelope does not leave room for electronics behind this module.** The module is 25 x 24 x 11.5 mm (12.4 mm tall for Wide) [1], so it consumes essentially all of the envelope depth, and the Wide variant is taller than 12 mm. Treat pendant depth as an open parameter: grow it, or let the lens protrude through the front face. No lens-height-above-PCB figure was found [U].
- A wide view (about 100 degrees horizontal) is the safer match for a chest-level, un-aimed camera (see "Field of view").
- The IMX708 driver supports only 2 MIPI lanes [4], so the host needs a 2-lane CSI-2 port and a Raspberry Pi kernel/libcamera stack. That constrains the processor-board choice (other tickets).

## Comparison

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

## Open risks

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
