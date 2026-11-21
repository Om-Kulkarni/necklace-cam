# Power and thermal budget (ticket #4)

Question: what battery life and heat can the Necklace sustain with a rear-battery Counterweight? Terms follow `CONTEXT.md`. Items marked **[E]** are my estimates or assumptions; **[U]** means I could not verify it from a primary source.

## Recommendation

1. **Cell**: one 3.7 V Li-polymer pouch cell with built-in protection module (PCM) in the Counterweight. Start with **1000 mAh (3.7 Wh) [E]**. A 500 mAh cell (e.g. 503035 size, 10.5 g as sold by Adafruit [14]) is a fallback; 2000 mAh is likely too heavy and bulky for a neck-worn Counterweight **[E]**. Require a cell with an IEC 62133-2 report and a UN 38.3 test summary.
2. **Charging**: USB-C 5 V into a small power-path charger with NTC/JEITA support, such as TI BQ25180 (3.0-5.9 V input, 5 mA-1 A charge, JEITA, battery-short and thermal protection [6]). Charge at <= 0.5C (500 mA for 1000 mAh), off-body, 0-45 C.
3. **Power transit**: regulate to a fixed 5 V in the Counterweight and send it over two 28-30 AWG stranded silicone conductors. Keep the path ES1 (<= 60 V dc [5]) and add a current-limiting eFuse in the Counterweight.
4. **Breakaway**: a releasable connector in the cord, so a snag opens the loop and cuts power. Target release force 15-25 N **[E, no adult-necklace standard found]**.
5. **Thermal is the binding limit, not energy.** The Pendant can only shed about 0.3-0.4 W continuously at skin-safe temperature **[E]**. Firmware must duty-cycle Capture and Transfer. A 30 s Clip is fine; back-to-back recording is not.

## Assumptions

| Item | Value | Basis |
|---|---|---|
| Usable battery energy | 81% of nominal (90% converter efficiency x 90% usable capacity) | **[E]** |
| Active power, 1080p30 Clip recording | 2 W (range 1-3 W) | Ticket scenario range |
| Active power, Wi-Fi Transfer | 2.5 W (range 1.5-3 W) | **[E]**; ESP32-S3 class SoC peaks about 283 mA TX, 88 mA RX per a third-party summary [8]; camera/storage add to it |
| Standby | 0.01 W, 16 h/day | **[E]**; ESP32-S3 class light sleep about 380 uA [8] |
| Clip | 30 s, about 10 Mbps, about 37 MB | **[E]** |
| Transfer of one Clip | 20 s at about 15-20 Mbps effective | **[E]** |
| Overhead per Clip (wake, file close) | 5 J | **[E]** |
| Pendant | 30 x 25 x 12 mm, surface 28.2 cm^2, mass 12 g, c about 1 J/g/K so C about 12 J/K | Size from ticket; rest **[E]** |
| Heat path | Back face (7.5 cm^2) against body/clothing treated as insulated; remaining 20.7 cm^2 sheds heat at combined convection + radiation h = 10 W/m^2/K, so G = 0.021 W/K | **[E]**, textbook order of magnitude |
| Local ambient at the chest | 30 C | **[E]** |

## Energy budget

Per Clip at 2 W record, 2.5 W transfer: 60 J + 50 J + 5 J = 115 J = 0.032 Wh.
Clips per charge = (usable Wh - 0.16 Wh standby) / Wh per Clip. All values **[E]**.

| Cell | Nominal | Usable | Clips/charge, low (1 W / 1.5 W) | Clips/charge, mid (2 W / 2.5 W) | Clips/charge, high (3 W / 3 W) | Continuous record at 1 / 2 / 3 W (energy only) |
|---|---|---|---|---|---|---|
| 500 mAh | 1.85 Wh | 1.50 Wh | about 74 | about 42 | about 31 | 1.5 / 0.75 / 0.5 h |
| 1000 mAh | 3.70 Wh | 3.00 Wh | about 158 | about 89 | about 66 | 3.0 / 1.5 / 1.0 h |
| 2000 mAh | 7.40 Wh | 6.00 Wh | about 324 | about 182 | about 136 | 6.0 / 3.0 / 2.0 h |

Standby alone costs about 0.16 Wh/day, so it is not decisive.

## Thermal budget

Skin-contact limits (primary-derived, secondary-reported):
- IEC 62368-1 adopted ISO 13732-1:2006 as the basis for its touch limits [1]. For class TS1, plastic and rubber surfaces touched for more than 1 min are limited to **48 C**, 10 s-1 min to 60 C, 1-10 s to 77 C, under 1 s to 94 C. Metal is 48 / 51 / 60 / 70 C [1]. TS2 is 10 K higher. The brief derives these from Table 42 of the first edition; table numbers differ by edition [1][2].
- A necklace Pendant touches skin or clothing for hours, so the >1 min row applies: **48 C max, and I design to 43 C**. Under abnormal conditions the standard relaxes limits only if the malfunction is evident to the user [1].
- Edition 4 reportedly makes the limit depend on accessibility and expected contact duration [3] **[U: not read in full text]**.
- ISO 13732-1 covers unprotected skin and contact of 0.5 s and longer; it assumes surface temperature stays about constant during contact [4, via search summary]. The 43 C "long contact" figure comes from a contact-burn paper cited in search results, not from the paywalled standard **[U]**.

Lumped model: T(t) = Tamb + (P/G)(1 - exp(-t/tau)), with tau = C/G of about 580 s. Steady state needs P <= G x (Tlimit - Tamb).

| Pendant dissipation | Steady rise | Time to 43 C | Time to 48 C | Sustainable continuous power |
|---|---|---|---|---|
| 1 W | 48 K | about 3 min | about 4.5 min | 0.27 W (43 C), 0.37 W (48 C) |
| 2 W | 97 K | about 1.4 min | about 2 min | same |
| 3 W | 145 K | about 0.9 min | about 1.3 min | same |

All **[E]**. Time scales with Pendant mass/12 g; steady power scales with real exposed area and airflow.

Consequences **[E]**:
- **Maximum continuous recording before the Pendant reaches 48 C: about 2 min at 2 W; about 4.5 min at 1 W; about 77 s at 3 W.** The cell could supply 1-3 h; heat, not energy, limits it.
- A single 30 s Clip at 2 W raises the Pendant about 5 K. Safe.
- Sustained average dissipation must stay <= about 0.3-0.4 W, i.e. about one 115 J Clip (record + Transfer) every 5-7 min, or 9-12 Clips/hour. Longer runs need a heat spreader (metal frame, thermally coupled to the chain), lower-power encoding, or deferring Transfer until the Necklace is off-body or docked.
- Put the DC converter in the Counterweight so conversion loss is not added to the Pendant.
- Add an NTC in the Pendant: throttle at 41 C, stop recording at 45 C (stopping must still keep the Recording Indicator honest).

## Power through the necklace

Conductor: 28-30 AWG copper (about 0.21-0.34 ohm/m) **[E, standard copper resistivity]**. Loop length 2 x 0.5 m **[E]**.

| Delivery | Load | Current | Drop, 30 AWG loop (0.34 ohm) | Cord loss |
|---|---|---|---|---|
| Raw cell 3.7 V | 2 W | 0.54 A | 0.18 V (5%) | 0.10 W |
| Raw cell 3.7 V | 3 W | 0.81 A | 0.27 V (7%), worse near 3.0 V cutoff | 0.22 W |
| Regulated 5 V | 3 W | 0.60 A | 0.20 V (4%) | 0.12 W |

5 V at the source gives lower current and keeps cell sag away from the SoC. Cord loss is spread along the cord and away from the Pendant.

## Safety requirements

- **Electrical class**: 3.7-5 V is far below the IEC 62368-1 ES1 limit of 60 V dc [5]. Power-source class (PS1/PS2) depends on available power including cell short-circuit capability; I did not verify the PS thresholds **[U]**. Have a lab classify it.
- **Short circuits**: cell PCM (overcurrent/short/over-/under-voltage), plus an eFuse or current limit at the Counterweight output, insulated conductors, no exposed contacts at the breakaway connector. Charger parts such as BQ25180 add battery-short, over-voltage and thermal protection [6].
- **Cell compliance**: IEC 62133-2 covers safety of portable sealed lithium cells and batteries including foreseeable misuse; the 2017 edition has Amendment 1 (2021) [7]. Standalone it is not sufficient for the end product, which still needs charging and normal-load evaluation [7]. I could not read the standard text **[U]**. UN 38.3 requires a test summary available on request for shipping [9].
- **Charging window**: typical Li-ion cell specs allow charge at 0-45 C, discharge at -20-60 C, with reduced current near 0-15 C [10][11]. Use charger JEITA with a cell NTC. Never charge while worn.
- **Breakaway / strangulation**: I found no adult-necklace breakaway standard. CPSC identifies neck drawstrings on children's outerwear as a strangulation hazard [12]. A safety-glasses lanyard is rated about 3 lb (about 13 N) as a single product example [13]. A retailer document cited ASTM F963 as requiring release under 5 lbf (about 22 N) for toys; I could not verify this in the standard **[U]**. Design target 15-25 N **[E]**, tested with a lab. Release must open the circuit, and the cord must not be stronger than the release. If sold for children, a stricter regime applies **[U]**.
- **Thermal**: 48 C max for >1 min contact [1], target 43 C, firmware throttle, no Counterweight heat on the neck during charging.

## Open risks

1. **SoC and camera power is assumed**, not measured. The 1-3 W range could be 0.7 W with a good encoder or 4 W with a cheap one. Measure on a dev board first.
2. **Thermal model is a hand estimate** (mass, h, contact area, ambient). Real values need a thermal mock with heater and thermocouples against skin simulant, worn indoors, outdoors and in sun.
3. **Breakaway force and standard** unresolved; test-house and legal input needed, especially for children or EU/US markets.
4. **Primary standards paywalled**: IEC 62368-1 tables, IEC 62133-2, ISO 13732-1 and ASTM F963 were not read; limit numbers come from a UL technical brief (2010, first edition).
5. **Cell dimensions and weights** for 1000/2000 mAh are guesses; the PKCell datasheet fetch timed out, so charge-rate and PCM thresholds were not confirmed.
6. **Wi-Fi Transfer cost** depends on router distance and phone behaviour.
7. Li-polymer swelling and cell abuse near the neck; consider crush protection and a mechanical enclosure rating.
8. LiFePO4 is a possible safer chemistry but has lower voltage and energy density **[U, not researched]**.

## Sources

1. UL, IEC 62368-1 Technical Brief: Touch Temperature Limits (2010): https://japan.ul.com/wp-content/uploads/sites/27/2014/06/1_techbrief_touchtemp.pdf
2. Advanced Energy, design considerations for max temperature per IEC standards (no table values given): https://www.advancedenergy.com/en-us/about/news/blog/design-considerations-for-maximum-temperature-per-iec-safety-standards/
3. TUV Rheinland, IEC 62368-1 FAQ: https://tuv.com/landingpage/en/iec-62368-1/meta-navigation/faq
4. ISO 13732-1:2006 catalogue page (found via search; direct content not read): https://www.iso.org/standard/43558.html
5. Littelfuse, IEC 62368-1 technical paper (ES1/ES2 limits): https://www.LittelFuse.com/~/media/electronics/technical_papers/littelfuse_birth_of_a_safety_standard_the_iec_62368_1_technical_paper.pdf.pdf
6. TI BQ25180 product page: https://www.ti.com/product/BQ25180
7. IEC 62133-2 consolidated listing: https://webstore.iec.ch/en/publication/70017 ; CSA listing note: https://www.csagroup.org/store/product/2704746/
8. Deneyap, ESP32-S3 power consumption (third-party; Espressif datasheet https://documentation.espressif.com/esp32-s3_datasheet_en.html section 5.6 not read): https://deneyapkart.org/en/docs/guc-tuketimi-esp32-s3
9. PHMSA, Lithium Battery Test Summary (UN 38.3): https://www.phmsa.dot.gov/sites/phmsa.dot.gov/files/2024-09/Lithium-Battery-Test-Summary-2024.pdf
10. TI, battery charging temperature (JEITA) note: https://ti.com/lit/pdf/slyt365
11. Example 18650 cell spec, temperature limits: https://www.tme.eu/Document/c1efad4b4ec6606ff8f2072be6abd02a/FEB-18650-4000mAh%20V2.pdf
12. CPSC, drawstrings in children's upper outerwear FAQ: https://www.cpsc.gov/business--manufacturing/business-education/business-guidance/drawstrings-in-childrens-upper-outerwear/frequently-asked-questions-faqs
13. Klein Tools breakaway lanyard (single-product example): https://www.kleintools.com/product/60177
14. Adafruit 500 mAh cell (weight, retired product): https://adafruit.com/products/1578
