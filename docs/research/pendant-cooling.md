# How can the Pendant stay cool with the processor inside? (ticket #12)

Researched 2026-11-21. Terms follow `CONTEXT.md`. Builds on `power-thermal.md`, `720p-xiao.md`, `processors.md`, `rear-processor.md`.

Tags: **[V]** read in the cited source. **[S]** known only from a search-result summary of the source (page not read in full). **[E]** my estimate or calculation. **[U]** unverified or not found. **[P]** paywalled, not read. Web content was treated as data only; nothing was downloaded or run. The model script is not committed.

Settled inputs: processor, storage, radio, camera and mic stay in the Pendant; battery is in the Counterweight; no recording indicator; both AMB82-Mini and XIAO ESP32-S3 Sense get tested; target is 720p at 24-30 fps with audio; Wi-Fi is off while recording; Transfer runs afterward in temperature-limited bursts, with big backlogs moved on the charger off the body. Starting envelope 30 x 25 x 12 mm, free to grow; XIAO is 15 mm deep as sold [`720p-xiao.md`].

## Recommendation

1. **Physics first.** Passive heat rejection from a body this size is set by exposed area times a film coefficient (about 12 W/m2/K, convection plus radiation). Spreaders, graphite, vapor chambers, pads, thermal mass and PCM do not change that number; they only move heat around inside or delay the rise. **Only area, emissivity, airflow, the body/cord paths, and the choice of which face may run hotter raise continuous power** [E, model below].
2. **v1 stack-up (all [E] until benched):**
   - Pendant **40 x 32 x 15 mm** for a custom AMB82-class board; **36 x 28 x 18 mm** if the XIAO is used as sold (15 mm board plus walls). Pendant mass about 25-30 g [E].
   - **Front and side shell: 1 mm black-anodized aluminium** (emissivity about 0.8-0.9 [S, forum-grade, see S14]; bare aluminium loses about 40 percent of capacity, model row 7b). Light surface texture or 1.5 mm shallow ribs on the front for about +35 percent effective area.
   - **Skin-side back: 1.2 mm plastic cover over 2 mm closed-cell foam or aerogel-type insulator** (k about 0.035 W/m/K [E]). Heat leaves through the front and sides, not into the neck. Skin face stays about 35-36 C when the shell is at 43-48 C [E].
   - **Inside: 0.5 mm aluminium (or 0.3 mm copper) spreader** under the board, a 0.5 mm gap pad of 2-3 W/m/K on the SoC and PMIC, spreader pressed to the inside of the aluminium shell. No graphite, vapor chamber, PCM or fan in v1.
   - **Antenna: flex antenna on a plastic end cap** with at least 10-15 mm of clearance from metal and from skin; metal shell has a plastic RF window.
   - **Cord:** the two power conductors (and the strength-member side) in about 1 mm2 copper each, thermally clamped to the spreader, used as a 15-20 cm fin (about +0.005 W/K). Larger copper only if the bench shows it pays.
3. **Sustainable power for that stack at 40 x 32 x 15 mm** (surface at or below 43 C, 30 C air, still indoor air): **about 0.9 W**. At the aggressive 48 C front-surface limit about 1.25 W. In a pessimistic case (half the film coefficient, i.e. under a jacket or shirt) **about 0.57 W**. Plan on **0.55-0.9 W continuous**.
4. **Consequence for the boards.** XIAO (about 0.7 W webcam average [V `720p-xiao.md`]) fits continuous recording only in the nominal case. AMB82-Mini (unmeasured, 1-1.5 W assumed [U]) is duty-limited at this size: about 9 min to 43 C at 1 W from a cold start, about 4 min at 1.5 W. Firmware policy (below) is not optional.
5. **Smallest sizes that sustain each power level** are in section 4. Continuous 1.5-2 W needs a 53 x 45 x 21 to 63 x 53 x 25 mm Pendant even at best case; that is a different product. If the measured recording power is above about 1 W, re-open ticket #10 (rear processor) rather than growing the Pendant.
6. **Biggest uncertainty:** the real film coefficient on a worn, partly covered body and the real recording power of both boards. Both are cheap to measure (bench test).

## 1. Touch and skin-contact limits

| Contact | Limit | Basis and status |
|---|---|---|
| Touchable surface, more than 1 min, **metal** or **plastic/rubber** | **48 C** (TS1) | IEC 62368-1 Table 42 as abridged in the UL technical brief [V S3]. Basis is ISO 13732-1:2006 [V S3]. Standard text **[P]**. |
| 10 s to 1 min | metal 51 C, plastic 60 C | [V S3] |
| 1 s to 10 s | metal 60 C, plastic 77 C | [V S3] |
| Under 1 s | metal 70 C, plastic 94 C | [V S3] |
| TS2 | TS1 + 10 K (58 C above 1 min) | [V S3]. Needs a safeguard or instruction; not for a worn product. |
| Abnormal or single fault, malfunction not evident | TS2, based on under 1 s limits (80 C metal) | [V S3]. Firmware failure that is silent (no recording indicator) is exactly this case, so the hard cutoff must be hardware. |
| Skin, unlimited contact | **15-43 C nominal**; 43-49 C = pain, controls needed; **49 C and above = tissue damage** | NASA-STD-3001 technical brief [V S1]. Pain threshold 43.7 C [Greene 1958], 44.6 C [Lloyd-Smith 1948], 43-46 C [Defrin 2006]; damage to porcine skin at 49 C [Moritz and Henriques 1946], all as quoted in [V S1]. |
| Skin, long exposure | 44 C for 420 min, 45 C for 150-180 min, 46 C for 45-90 min, 47 C for 35-60 min, 48 C for 10-20 min produce threshold burns | Moritz and Henriques data table as reproduced in [V S1, p. 6]. Animal skin plus human data; read as order of magnitude. |
| Heat-pain threshold at the **chest** | about 41-43 C by sex and method (women lower, about 41.5) | Defrin 2006 chart reproduced in [V S1, p. 6]. This is why the skin face is designed to 41 C, not 43 C. |
| ISO 13732-1 draft, 0.5-10 s | bare metal no-burn about 54.5 C at 10 s; ceramics about 65.5 C; plastics about 70 C (read off the curves) | Draft ISO/DIS 13732-1:2004 [V S2]. The draft says material differences "disappear" for long contact, and its data assume the surface temperature is held by the product's mass or a heat source (worst case). The 1 min and longer table is beyond the free preview **[P]**. |
| Medical applied parts (IEC 60601-1 Table 24) | not verified | Only the table title was found [S S5]. **[P]**. Do not cite a number. |
| ASTM C1055 (contact burn guide) | not verified | **[P]** |

**What this means for the Pendant [E].**
- The 48 C figure is a "touchable surface, can withdraw" limit. A worn Pendant cannot be assumed withdrawn, and Moritz and Henriques put threshold burns at 10-20 min for 48 C [V S1]. So:
- **Skin-contact face: design limit 41 C, hard limit 43 C.** Applies to hours of contact. This face is also where metal versus plastic hardly matters (long contact, curves converge [V S2]).
- **Front and side surfaces a hand can hold: warn 43 C, stop 45 C, hard 48 C.** Metal and plastic share the 48 C row above 1 min [V S3]. Metal feels hotter for short touches (51 versus 60 C in the 10 s to 1 min row [V S3]); a finger touch of 1-10 s at 45 C is far below either value.
- Thermal mass matters for burns too: a small, light object cools where it touches skin, so the standard's constant-temperature assumption is conservative [V S2, S3]. I did not credit this.
- Adult skin only. Children and elderly skin burn at lower temperatures; ISO 13732-1 says its scope includes them [V S2]. Treat as **[U]** for any child-facing product.

## 2. Model

**Nodes.** Core `c` (board, SoC, sensor, SD) and shell `s` (outer surface and inner spreader). Back face is thermally isolated except through `G_sk`.

```
C_c dTc/dt = P - (Tc - Ts)/R_cs - G_f (Tc - Ta)
C_s dTs/dt = (Tc - Ts)/R_cs - (G_a + G_ch)(Ts - Ta) - G_sk (Ts - Tsk)
G_a   = (h_c + h_r) * f * A_exp          h_r = eps * 4 * sigma * Tm^3
T_hot = Ts + theta * P                    (hottest outer spot; theta = hot-spot K/W)
P_sus(Tlim) ~ (G_a+G_ch)(Tlim - Ta - theta P) + G_sk (Tlim - theta P - Tsk)
```
Steady state is the 2x2 linear solution of the same equations (the script checks it against the time integration). Single-node form: `T(t) = Ta + (P/G)(1 - e^(-t/tau))`, `tau = C/G`.

**Assumptions** (all [E] unless cited):

| Symbol | Baseline value | Basis |
|---|---|---|
| Geometry | 30 x 25 x 12 mm; area 28.2 cm2; back face 7.5 cm2; `A_exp` = 20.7 cm2 | `power-thermal.md` |
| `h_c` | 6 W/m2/K | Laminar vertical-plate natural convection at this size gives about 7; I used 6 (textbook correlation `Nu ~ 0.59 Ra^0.25`, Incropera/Holman family, not re-read; one forum source says 2-4 for large still surfaces [S S14]) |
| `h_r` | 6.1 W/m2/K at eps 0.9; 0.7 at eps 0.1 | `4 eps sigma Tm^3`, Tm = 309 K (my arithmetic) |
| `G_a` baseline | 0.025 W/K (total film 12 W/m2/K) | earlier ticket used 0.021 at 10 W/m2/K; same order |
| `Ta`, `Tsk` | 30 C chest-level air, 34 C skin | `power-thermal.md`; skin 34 C typical [U] |
| `C_c`, `C_s` | 5 and 7 J/K (12 g total) | `power-thermal.md` |
| `R_cs`, `theta` baseline | 10 K/W, 3 K/W | Biggest guess: air-filled plastic box with a board in it. Bench must calibrate. |
| `G_sk` insulating back | 0.012 W/K | 7.5 cm2 over 2 mm foam, k 0.035 |
| Base case for all rows | back face adiabatic (`G_sk` = 0) except option 9 and R | comparable with `power-thermal.md` |
| Power `P` | dissipated electrical power in the Pendant (radiated RF is negligible) | |

Spreading arithmetic (why spreaders do not raise capacity): lateral conductance `k*t` is Al 0.5 mm 0.10 W/K, Cu 0.3 mm 0.12 W/K, graphite 25 um 0.049 W/K (using 1950 W/m/K in-plane [V S6]), 1 mm Al shell 0.20 W/K, 0.4 mm vapor chamber about 1.4-4 W/K at the 3500-10000 W/m/K range vendors and papers quote [S S9, S10]. With `h` near 12 W/m2/K the fin length `1/sqrt(h/(k t))` is 6-13 cm for the metals, so on a 3 cm body any of them already makes the surface nearly isothermal; the vapor chamber's extra conductance buys nothing for steady power [E]. Graphite sheet (0.049 W/K) is actually weaker than 0.5 mm aluminium for the same area, and its through-thickness conductivity is poor [S S6, not found].

### Results: time to surface limit and steady temperature

Cell format: **minutes to 43 C / minutes to 48 C / steady hot-surface temperature in C** (hot surface = shell + hot-spot term). Start at ambient 30 C. "never" means the limit is not reached. Steady temperatures above 60 C are extrapolations (the firmware would stop long before). Last four columns: `P_sus` in W for the 43 C limit, the 48 C limit, 43 C with half the film coefficient (under a jacket, `h` x 0.5), and 43 C with 35 C ambient. All [E].

| Option (30 x 25 x 12 mm unless stated) | 0.5 W | 0.7 W | 1 W | 1.5 W | 2 W | P_sus 43 | P_sus 48 | 43, h x0.5 | 43, Ta 35 |
|---|---|---|---|---|---|---|---|---|---|
| 0 Baseline plastic | 7.6 / 15 / 52 | 4.6 / 7.5 / 60 | 2.9 / 4.4 / 73 | 1.8 / 2.6 / 95 | 1.2 / 1.8 / 116 | 0.30 | 0.42 | 0.16 | 0.19 |
| 1 Al spreader 0.5 mm + pad | 8.7 / 18 / 51 | 5.2 / 8.5 / 59 | 3.3 / 5.0 / 71 | 2.0 / 3.0 / 92 | 1.5 / 2.1 / 112 | 0.32 | 0.44 | 0.16 | 0.19 |
| 1b Cu spreader 0.3 mm + pad | 8.5 / 18 / 51 | 5.1 / 8.4 / 59 | 3.2 / 4.9 / 71 | 2.0 / 2.9 / 92 | 1.5 / 2.1 / 112 | 0.32 | 0.44 | 0.16 | 0.19 |
| 2 Graphite 25 um | 8.1 / 17 / 51 | 4.9 / 8.0 / 59 | 3.1 / 4.7 / 72 | 2.0 / 2.9 / 92 | 1.4 / 2.0 / 113 | 0.31 | 0.43 | 0.16 | 0.19 |
| 3 Vapor chamber 0.4 mm | 8.8 / 19 / 50 | 5.3 / 8.6 / 58 | 3.4 / 5.1 / 70 | 2.1 / 3.1 / 91 | 1.6 / 2.2 / 111 | 0.32 | 0.44 | 0.16 | 0.20 |
| 4 Gap pads only | 7.3 / 15 / 52 | 4.3 / 7.2 / 60 | 2.6 / 4.2 / 73 | 1.5 / 2.4 / 95 | 1.0 / 1.6 / 116 | 0.30 | 0.42 | 0.16 | 0.19 |
| 5 +10 g aluminium mass | 13 / 27 / 52 | 8.0 / 13 / 60 | 4.9 / 7.6 / 73 | 2.8 / 4.4 / 95 | 1.9 / 3.0 / 116 | 0.30 | 0.42 | 0.16 | 0.19 |
| 6 +3 g PCM (540 J, 40-44 C) | 22 / 61 / 52 | 9.4 / 30 / 60 | 2.8 / 17 / 73 | 1.7 / 9.3 / 95 | 1.1 / 4.6 / 116 | 0.30 | 0.42 | 0.16 | 0.19 |
| 7a Al shell, black anodized | 9.1 / 18 / 51 | 5.5 / 8.9 / 59 | 3.5 / 5.3 / 72 | 2.2 / 3.2 / 93 | 1.7 / 2.3 / 114 | 0.31 | 0.43 | 0.16 | 0.19 |
| 7b Al shell, bare | 7.2 / 11 / 66 | 4.8 / 7.1 / 81 | 3.2 / 4.7 / 103 | 2.1 / 3.0 / 139 | 1.6 / 2.2 / 176 | 0.18 | 0.25 | 0.09 | 0.11 |
| 8 Fins/texture (+35% area) | 12 / never / 45 | 6.1 / 12 / 52 | 3.6 / 5.8 / 61 | 2.2 / 3.2 / 76 | 1.6 / 2.3 / 91 | 0.42 | 0.59 | 0.21 | 0.26 |
| 9 Insulating back, heat out front | 11 / never / 45 | 5.7 / 12 / 51 | 3.4 / 5.6 / 59 | 2.1 / 3.1 / 73 | 1.5 / 2.2 / 87 | 0.42 | 0.59 | 0.26 | 0.30 |
| 10 Vents (+15% G, -2 K/W) | 8.2 / 22 / 49 | 4.7 / 8.0 / 57 | 2.8 / 4.5 / 68 | 1.7 / 2.6 / 87 | 1.1 / 1.8 / 106 | 0.34 | 0.47 | 0.18 | 0.21 |
| 11 Micro-blower 0.6 L/min | 11 / never / 46 | 5.4 / 11 / 52 | 3.1 / 5.2 / 61 | 1.8 / 2.8 / 77 | 1.2 / 1.9 / 92 | 0.42 | 0.58 | 0.27 | 0.26 |
| 12 TEC (0.25 W input) | 3.9 / 6.4 / 62 | 2.8 / 4.5 / 71 | 2.0 / 3.0 / 84 | 1.2 / 1.9 / 106 | 0.9 / 1.4 / 127 | 0.05 | 0.17 | 0.00 | 0.00 |
| 13 Cord 2 x 1 mm2 Cu | 9.5 / never / 47 | 5.3 / 9.3 / 54 | 3.2 / 5.0 / 65 | 2.0 / 2.9 / 82 | 1.4 / 2.1 / 100 | 0.37 | 0.52 | 0.22 | 0.23 |
| 13b Cord 2 x 2 mm2 Cu | 12 / never / 45 | 5.7 / 12 / 51 | 3.4 / 5.4 / 60 | 2.0 / 3.0 / 75 | 1.4 / 2.1 / 90 | 0.43 | 0.60 | 0.28 | 0.27 |
| **R v1 stack, 30 x 25 x 12** | never / never / 42 | 8.2 / never / 46 | 4.2 / 7.9 / 52 | 2.4 / 3.8 / 63 | 1.7 / 2.5 / 73 | 0.57 | 0.80 | 0.37 | 0.39 |
| **R v1 stack, 40 x 32 x 15** | never / never / 38 | never / never / 41 | 9.0 / never / 45 | 3.8 / 7.5 / 51 | 2.5 / 4.1 / 58 | **0.88** | **1.25** | **0.57** | **0.61** |

Reading the table:
- Baseline sustains about 0.30 W at 43 C (0.42 W at 48 C), matching the 0.27-0.38 W in `power-thermal.md`. At 1 W the baseline hits 43 C in 3 min.
- The R stack at the original size roughly doubles sustained power (0.30 to 0.57 W) from four things: black emissivity versus nothing special, 35 percent more area, the insulating back letting the body absorb about 0.1 W while the skin face stays cool, and the cord fin. Growing to 40 x 32 x 15 adds another 55 percent.
- Option rows 1-4 look nearly identical because they only reduce `R_cs` and `theta`; they lower the core temperature (for example at 1 W the core is 74 C instead of 80 C, from the script) and make the surface uniform, but the surface temperature is set by `G_a`.
- Core-to-shell: at 2 W the baseline core would sit 14 K over the hot surface; the Al spreader cuts that to about 6 K [E].

**Delay versus capacity.**

| Effect | Options |
|---|---|
| **Only delay heating** (no gain in steady power) | thermal mass (5), PCM (6), gap pads (4), graphite (2), vapor chamber (3), Al/Cu spreader (1, 1b). These lower core temperature or add minutes; PCM also lets the Pendant hold under 43-44 C but only while it has latent heat left, and it must re-freeze, which needs the Pendant to cool below about 38-40 C. 3 g of paraffin stores about 540 J (latent heat about 180 J/g; RT58 paraffin is listed at 179 kJ/kg [S S12]; the melt range I used is chosen, not from a datasheet) = about 13 min at 1 W net. |
| **Raise continuous power** | more exposed area (larger Pendant, fins/texture 8), higher emissivity (7a versus 7b), insulating back with a hotter front (9), cord conduction (13), forced air (11), venting (10, small) |
| **Lowers continuous power** | bare metal shell (7b), TEC (12) |

## 3. Option comparison

Columns: added thickness/mass/power/cost per option; costs are small-quantity prototype guesses in USD [E]. "Gain" is from section 2 at 30 x 25 x 12 mm. Buildability is for a one-person prototype.

| Option | Added mm | Added g | Added W | Cost [E] | Buildability | Water, dust, sweat | Safety | Gain (section 2) | Type |
|---|---|---|---|---|---|---|---|---|---|
| Al or Cu internal spreader | 0.3-0.5 | 0.7-1.4 | 0 | 1-3 | Easy (cut sheet, pad) | Sealed inside; none | Needs insulation from board; if Cu touches nothing exposed, fine | +0.02 W sustained; core -6 K at 1 W | Delay / lower core temp |
| Graphite sheet | 0.03-0.1 | 0.1 | 0 | 1-3 | Easy; fragile edges, conductive flakes short circuits (use laminated PET grade, 1600 vs 1950 W/m/K [S S6]) | Sealed inside | Conductive debris near electronics | +0.01 W | Delay / lower core temp |
| Vapor chamber (0.4 mm) | 0.4 | 1-2 | 0 | 5-15 or custom MOQ [U] | Hard to source one-off; not at this price class [S S9] | Sealed | Leak or burst unlikely at 0.4 mm; fails when bent [U] | +0.02 W | Delay / lower core temp |
| Gap pads (1-3 W/m/K, 0.5 mm: about 2.5 K/W on a 10 x 10 mm SoC at 2 W/m/K) | 0.5 | 0.3 | 0 | 0.5 | Easy | Sealed | Compressible, silicone oil bleed [U] | Core -3 to -5 K | Lower core temp |
| Added thermal mass (10 g Al) | 4-7 | 10 | 0 | 1-2 | Easy | Sealed | None | 1 W: 2.9 to 4.9 min to 43 C | Delay |
| PCM (3 g, 540 J) | 3-4 | 3-5 | 0 | 2-5 | Moderate (leak-proof pouch; paraffin expands about 10 percent [U]) | Leak risk onto board | Flammable organic, leak near Li-ion is ugly; keep it away from skin | 0.7 W: 7.5 to 30 min to 48 C | Delay (recovers only when cooled) |
| Metal shell (black anodized) | 0 vs plastic wall | +3.6 | 0 | 10-40 anodized prototype | Moderate (CNC, anodizing) | Good with gasket; sweat corrodes bare Al; anodize protects | Skin contact allergy low for anodized Al; hot touch row same as plastic above 1 min | +0.01 W vs plastic; **bare** metal -0.12 W | Neutral or capacity loss (emissivity) |
| Plastic shell | 0 | 0 | 0 | 1-5 | Easy (print, mould) | Easiest to seal | Lower touch burn for short contact | baseline | none |
| Fins / texture | 2-3 (fins) or 0 (texture) | 3-5 | 0 | 10-30 | Moderate | Dirt and sweat traps; snags on cord and clothing | Edges against skin if facing back (keep fins front only) | +0.12 W (+35 percent area; [E], fin spacing about 3-4 mm and natural convection limit it) | **Capacity** |
| Larger Pendant | per dimension | proportional | 0 | 0 | Easy | none | none | Capacity scales with exposed area | **Capacity** |
| Insulating skin-side back + hot front | 2-3 | 1-2 | 0 | 1-3 | Easy | Foam must not wick sweat (closed cell) | **Best skin protection**: face about 35-36 C at 43-48 C shell [E] | +0.12 W (the body absorbs about 0.1 W at 43 C) | **Capacity** (and makes 45-48 C front acceptable) |
| Vents + natural convection | 0 | 0 | 0 | 0 | Easy | **Poor**: sweat and dust ingress; blocked by clothing | Sweat on live parts | +0.04 W [E, unverified] | Small capacity |
| Micro-blower | 4-6 incl. duct and driver | 3-5 | **0.1-0.3 [U, not found]** | 10-20 plus boost driver [U] | Hard: Murata MZB1001T02 is 20 x 20 x 1.85 mm, 1.4 g, 1.2 L/min no load, 2.5 kPa, 11.5-21 V drive circuit you build [S S7]; sample driver is evaluation only [S S7] | **Poor**: needs inlet and outlet openings | Moving parts, ultrasonic 26 kHz drive [S S7] | +0.12 W (0.6 L/min carries about 0.012 W/K of air heat capacity rate; 1.2 L/min only 0.023) | **Capacity**, small, at a power and reliability cost |
| TEC (3.3 mm, 30 x 30 mm class) | 3-4 | 5-8 | **+0.2-0.4** | 5-15 plus driver | Moderate | Sealed possible | Hot side must reject Q_c + P_in | **-0.25 W** | **Reduces capacity** |
| Cord/chain conduction to Counterweight | 0 in Pendant | +8 g (2 x 1 mm2 Cu), +16 g (2 x 2 mm2) in cord | 0 | 2-5 | Moderate: stiffness, breakaway connector, strength member | Jacket sealed | Cord surface near Pendant is warm (about shell temperature); sleeve it; must not defeat the breakaway | +0.07 W (1 mm2), +0.13 W (2 mm2) | **Capacity**, but mostly as a fin, see below |

**Thermoelectric cooler: verified counterproductive for the surface limit.** A TEC rejects `Q_c + P_in` at its hot side [thermodynamics]. Laird's CP10-71-06 optimum-COP point pumps 1.46 W for 0.38 W input (COP 3.83); at maximum Qc it pumps 12.72 W for 28.81 W (COP 0.44) [S S8]. Even at the good point every watt pumped adds about 0.26 W of extra heat to the same shell, and the shell-to-ambient limit is unchanged. The model (0.25 W for about 0.5 W pumped) cuts `P_sus` from 0.30 to 0.05 W. It is useful only to cool one component (the sensor) below the shell temperature, at a cost in power and 3 mm of depth. Not for v1.

**Cord conduction mostly acts as a fin, not a conduit.** Pure conduction through 1 mm2 of copper over a 0.45 m leg is `k A / L = 390 x 1e-6 / 0.45 = 0.00087 W/K`. Fin behaviour is stronger: `sqrt(h P k A)` is 0.0037 W/K per 1 mm2 wire (h = 10) and 0.0024 W/K sheltered (h = 4), with a 10-20 cm decay length [E, arithmetic]. So only the first 20 cm of cord matters, and the Counterweight sink is irrelevant; the Counterweight takes under 0.1 W in the best case (no battery risk). The cord must stay below 43 C where it touches the neck: its temperature decays away from the Pendant [E].

### Side effects

| Concern | Effect and limit | Status |
|---|---|---|
| **Camera sensor image noise** | Dark signal doubles about every 6-8 C [S S13: arXiv tutorial; Teledyne says every 6-7 C halves it; one vendor note says 9 K [S S13]]. Sensor 20 K hotter than a cool start is 4-9 times the dark signal; shading and hot pixels grow. The OV5640 can exceed 70 C in continuous use and Seeed shows a register tweak (about 54 C) with added noise risk [V `720p-xiao.md`]. | Keep the sensor off the SoC heat path (flex or separate corner, no spreader under it), put an NTC on it, and measure dark frames (bench). |
| **SD card** | Typical extended-grade microSD parts list -25 to 85 C operating [S S11]; consumer cards often only 0-70 C or unstated [U]. Write speed at high temperature is not published in what I found [U]. | Policy: card temperature limit 65 C; NTC beside the slot. At v1 core steady values of 40-55 C this passes [E]. |
| **Battery** | In the Counterweight. Charge window 0-45 C, discharge to 60 C [`power-thermal.md` S10, S11]. Cord heat is under 0.1 W [E]. Charging while worn is forbidden by `power-thermal.md`; Transfer backlog is therefore charger-only. | No new risk from chain conduction. A hot, sunlit Counterweight is a separate risk. |
| **Wi-Fi antenna** | Metal near the antenna detunes it; Espressif advises at least 15 mm clear area around a PCB antenna if it cannot overhang the board edge, and tests in the real enclosure [S S4, ESP32-S2 guide, S3 and C6 guides have per-module keep-out figures]. Body effect: 10 mm clearance advised and 54 percent efficiency on the chest versus 85 percent in free space, from `rear-processor.md` (S18 there). | Plastic window or end cap, external flex antenna, no metal in front of it. Wi-Fi is off during recording, so heat does not come from RF while detuned, but a detuned antenna raises Transfer time and power. Test RSSI and throughput with and without the metal shell. XIAO external antenna connector: **[U, not verified]**. |
| **Water, dust, sweat** | Sealed shell and no vents are best; anodize protects aluminium from sweat. Vents and blowers fail this. | [E] |
| **Skin-side safety** | Insulating back keeps the skin face 35-36 C at shell 43-48 C [E]; below the 41 C design limit and the 43 C pain onset [V S1]. | A failed foam (compressed, wet) changes this: skin NTC is mandatory. |

## 4. Smallest Pendant that sustains each power

R stack, aspect 30 : 25 : 12 scaled uniformly, so depth grows with width (so XIAO at 15 mm needs the 0.7 W row or larger; with depth fixed at 18 mm the 36 x 28 x 18 body gives 0.83 W nominal). Width x height x depth, mm. Mass grows roughly with area; the 40 x 32 x 15 body is about 25-30 g [E].

| Continuous power | Nominal (43 C, 30 C air, h x1) | Aggressive (48 C front) | Pessimistic (43 C, h x0.5, under a layer) | Plain plastic box, nominal 43 C (reference) |
|---|---|---|---|---|
| 0.5 W | 28 x 24 x 11 | 23 x 19 x 9 | 37 x 31 x 15 | 40 x 34 x 16 |
| 0.7 W | 34 x 29 x 14 | 28 x 24 x 11 | 44 x 37 x 18 | 49 x 41 x 19 |
| 1 W | 42 x 35 x 17 | 34 x 29 x 14 | 54 x 45 x 22 | 61 x 51 x 24 |
| 1.5 W | 53 x 45 x 21 | 43 x 36 x 17 | 69 x 58 x 28 | 80 x 67 x 32 |
| 2 W | 63 x 53 x 25 | 51 x 43 x 20 | 82 x 68 x 33 | 102 x 85 x 41 |

With a micro-blower (0.6 L/min) and a 2 x 2 mm2 cord, nominal 43 C sizes shrink about 12-15 percent in each dimension (1 W: 37 x 31 x 15; 2 W: 59 x 50 x 24) [E]. With 35 C ambient instead of 30 C the sizes grow about 20-25 percent (1 W: 52 x 44 x 21). These are tolerance bands, not different answers: **1 W continuous is about 42 x 35 x 17 mm at best and 54 x 45 x 22 mm when covered; 2 W is not a necklace pendant**.

## 5. Firmware thermal policy

**Sensors (4 NTCs, 10 k, plus chip sensors where present):**

| ID | Location | Used for |
|---|---|---|
| NTC_A | on the spreader beside SoC/PMIC | core and rate-of-rise |
| NTC_B | inside the front shell, mid-face, bonded with pad | hot outer surface (the front/side limit) |
| NTC_C | on the skin-side face, under the back cover, over the insulator | skin-face limit; also on-body detection |
| NTC_D | beside the camera sensor (and SD slot if board space) | noise and SD limits |
| SoC temperature | ESP32-S3 on-chip sensor [S S12, not read in full]; AMB82 [U] | cross-check; not trusted alone |

Treat an open or shorted NTC as "hot". Calibrate NTC_B against an IR or contact probe on the outer surface in the bench test; inner-to-outer offset for 1 mm Al is under 1 K, for a 1.2 mm plastic wall about 1.5 K at 0.5 W [E].

**Limits (hysteresis 3 K, re-arm after 60 s):**

| Level | Triggers (any) | Action |
|---|---|---|
| 0 Normal | B < 41, C < 38, A < 60 C | none |
| 1 Soft limit | B >= 41, C >= 38, A >= 60, or predicted B >= 45 within 5 min (`dT/dt` over last 60 s, filtered) | drop to 720p24 and lower bitrate; Transfer paused; flag to phone over BLE when connected |
| 2 Warning | B >= 43, C >= 39.5, A >= 65 | finish the current 30 s segment, then stop; BLE warning; log |
| 3 Stop | B >= 45, C >= 41, A >= 70, D (sensor) >= 70, SD >= 65 | close the file within 3 s (below), power camera, mic and radios off, SoC sleeps; no restart until B < 38, C < 36, A < 50 for 60 s |
| 4 Hard cutoff | B >= 48, C >= 43, A >= 80, or NTC fault | **hardware comparator** on NTC_B cuts the sensor and radio supplies independent of firmware; firmware also sleeps SoC after `f_close` |

Aligns with `power-thermal.md` (throttle 41, stop 45) and adds the skin-face limit. With no recording indicator, a silent stop can leave the wearer thinking recording is still on; the Pendant has no other output channel, so the phone notification (and a logged stop reason) is the only warning path and is a UX and safety risk to carry to the indicator ticket **[E]**.

**Safely closing the file.** Record in 30 s segments (the AMB82 MP4 example default [V `720p-xiao.md`]); on level 3, flush the encoder, finalize the MP4 index, `f_sync`, close, in under 2-3 s. XIAO MJPEG-in-AVI needs the index written at end of file, so write periodic index checkpoints (or a fragmented layout) so a cut leaves a playable file **[E, XIAO library support unverified]**. Margin check: worst-case rise is about `P/C_eff` = 2 W / 19 J/K = 0.1 K/s at the v1 size (0.15 K/s baseline), so from 45 C to the 48 C hard line takes 20-30 s, far more than the close time. Never cut SD power mid-write: the hard cutoff drops sensor and radio rails only.

**Transfer burst control** (Wi-Fi TX adds an assumed 0.8-2.5 W [`power-thermal.md`, `720p-xiao.md`]):
- Start only if not recording and B < 38, C < 36, A < 55.
- Burst until B >= 41 or C >= 38 or A >= 60 or 60 s elapsed; rest until B <= 37. Rough burst length `t = C_eff (41 - B - theta P) / (P_tx - P_sus)`; at v1 size from 36 C at 2 W this is about 30-60 s [E, C_eff about 19 J/K]. Average duty about `P_sus / P_tx` = 35-45 percent at v1 size.
- A 1 min AMB82 Clip needs 8-15 s of Wi-Fi; XIAO MJPEG needs 40-70 s [V `720p-xiao.md`], i.e. one or two bursts each.
- **Large backlog:** only when the Counterweight sees charger power (charging is off-body per `power-thermal.md`). Allowed continuous with B limit 45 C (stop 45, hard 48) and the Pendant lying free; the backlog resumes when B < 40.
- Adaptive scaling: after boot, use the measured `dT/dt` and steady values to scale the burst length (model update, section 7).

## 6. Bench experiment (both boards)

**Cost about USD 80-150 [E]; one day for the dummy, one day for each board.**

**Track A, enclosure and thermal model (no board needed).** 3D-print the Pendant body at 30 x 25 x 12 and 40 x 32 x 15; fit a 5 V power resistor (1-5 ohm class) as heater with a 4-wire shunt/INA226 (or inline USB-C power meter) to log true watts at 0.5, 0.7, 1, 1.5, 2 W. Build variants: plastic baseline; Al spreader + pad; black-anodized or matte-black-painted Al front; insulating back; with and without cord stubs.

**Track B, real boards.**
- XIAO ESP32-S3 Sense in a printed shell sized for 15 mm: 720p MJPEG record to SD with audio, log fps, drops, SD errors.
- AMB82-Mini: the bare board (about 60 x 40 mm [C `720p-xiao.md`]) does not fit; record its power and temperatures in the open, then reproduce the same watts in the Track A dummy.
- Modes (each 5 min settle plus until steady or trip): idle with camera on, 720p24, 720p30 with audio, SD-only write, Wi-Fi TX burst to a phone hotspot, record + Wi-Fi (worst case).

**Instrumentation.** 4-6 NTCs (10 k, 1 percent) or type-T thermocouples with MAX31856-class amplifiers on core, spreader, front shell, skin face, sensor, SD slot, plus ambient; logger at 1 Hz. Calibrate in an ice bath (0 C) and a 37 C water bath against a reference thermometer (target 0.5 K). Optional: IR camera or matte-taped surface with a pyrometer for outer-surface maps. USB power meter at 1 Hz. Dark frames (lens capped) at start and at steady state; a grey card for noise.

**Conditions.**
1. Still air, 25 C, in a draft-free cardboard box.
2. Warm room or box at 30-32 C.
3. Forced air 1 m/s (fan) as a walking proxy.
4. Skin simulant: the Pendant face-down on a 34 C isothermal plate under one cotton layer (conservative heat sink); then on the volunteer builder's own chest, over a shirt and under a light jacket, with the firmware stop thresholds live and an independent skin NTC. Adults only, short-term, stop at C >= 41 C. This is a thermal test, not a medical one.
5. Cool-down after each run (for `C` and `tau`).

**Pass/fail (v1 stack):**

| Check | Pass | Fail |
|---|---|---|
| Continuous 720p24-30 + audio, 30 min at 30 C | B <= 45 C and C <= 41 C throughout, no firmware stop | any stop, or B > 45 C |
| Steady core and sensor | A <= 65 C, D <= 65 C | A > 70 C or D > 70 C |
| SD card | <= 65 C, no write errors, no stalls above 200 ms | errors, or stalls |
| Video | at least 24 fps, at most 2 percent dropped frames in the last 10 min (matches `720p-xiao.md`) | below |
| Image noise | grey-card and dark-frame noise at steady state at most 1.3 times the cold start | above |
| Wi-Fi | with metal shell, RSSI loss at most 3 dB and throughput at least 80 percent of the open board | worse |
| Model | predicted steady B within 3 K, `tau` within 25 percent | else recalibrate |
| Covered case | with a jacket, B <= 45 C at the planned duty | else duty or size grows |

**How results update the model.** From each steady run: `G_a,meas = (P - G_sk (Ts - Tsk)) / (Ts - Ta)`; scale `h_c` so the model matches; from the first minute `C = P / (dT/dt)`; `R_cs = (Tc - Ts)/P`; `theta = (T_max_surface - T_mean_surface)/P`; fit `tau` on the cool-down. Re-run `P_sus` and the sizing table with the measured `h`; if the covered-case `P_sus` at the chosen size is below the measured recording power, either enlarge (section 4), add the micro-blower (+0.12 W [E]), or move to the rear processor (ticket #10). Update `power-thermal.md` with measured watts for both boards.

## 7. Risks and unknowns

1. **Film coefficient on a real body is the dominant unknown**; `h` x0.5 halves capacity. The hot-spot factor and `R_cs` are guesses.
2. **Recording power of both boards is unmeasured** (AMB82 1-1.5 W assumed [U]; XIAO about 0.7 W is a webcam average, not 720p24 recording with SD [V `720p-xiao.md`]).
3. **Skin and standard limits:** the 48 C touch figure and the 41-43 C skin figures rest on secondary or free-preview sources; IEC 62368-1, IEC 60601-1 Table 24, ISO 13732-1:2006 and ASTM C1055 are **[P]**. A certifier should set the final limits and contact-time class.
4. **Emissivity of black anodize** was sourced only from forum and blog-level pages [S S14]; measure.
5. **Vapor-chamber, TEC, blower numbers** come from vendor pages read through search summaries; the Murata power figure was not found.
6. **Antenna** behaviour with a metal shell and body is untested; XIAO external-antenna connector unverified.
7. **PCM** melting range, expansion and leak behaviour are [E], not from a datasheet (the Rubitherm RT44HC sheet was not reachable).
8. **Dark-current doubling** varies by sensor process (6-10 K); no OV5640 or JXF37P figure was found.
9. **Cord as heat path** adds stiffness, mass and a breakaway-force interaction; do not trade breakaway safety for a 0.07 W gain.
10. **Under-clothing, sunlight and cars:** 35-40 C ambient cuts capacity by 35 percent or more (table column "Ta 35").

## Sources

- S1 NASA-STD-3001 Technical Brief OCHMO-TB-009, Physiological Effects of Touch Temperature (2022): https://nasa.gov/wp-content/uploads/2023/03/touch-temperature-technical-brief-ochmo.pdf **[V, pages 1-6]**
- S2 ISO/DIS 13732-1 (2004 draft preview, free): https://cdn.standards.iteh.ai/samples/34075/81bf61b1e37b446f98cd36b0ccb0fefc/ISO-DIS-13732-1.pdf **[V, 13 preview pages]**; catalogue page for the 2006 standard: https://www.iso.org/standard/43558.html **[P]**
- S3 UL, IEC 62368-1 Technical Brief: Touch Temperature Limits (2010): https://japan.ul.com/wp-content/uploads/sites/27/2014/06/1_techbrief_touchtemp.pdf **[V]**
- S4 Espressif hardware design guidelines, PCB layout: https://docs.espressif.com/projects/esp-hardware-design-guidelines/en/latest/esp32s3/pcb-layout-design.html and ESP32-S2 guide https://www.espressif.com/sites/default/files/documentation/esp32-s2_hardware_design_guidelines_en.pdf **[S]**
- S5 IEC 60601-1 preview index (table titles only): https://webstore.iec.ch/preview/info_iec60601-1{ed3.1}en.pdf **[S, P]**
- S6 Panasonic PGS graphite sheets (1950 W/m/K in-plane, 25 um; through-thickness value not found): https://industrial.panasonic.com/cdbs/www-data/pdf/AYA0000/AYA0000C59.pdf **[S]**
- S7 Murata MZB1001T02 microblower: https://www.murata.com/en-global/products/mechatronics/fluid/microblower_mzb1001t02 ; spec https://www.mouser.com/datasheet/2/281/MZB1001T02_SPEC_Ver2-202918.pdf (fetch timed out) ; driver guide https://www.mouser.lt/pdfDocs/Murata-MZB1001T02-Microblower-Driver-Circuit-Guidev12.pdf **[S]**
- S8 Laird CP10-71-06 TEC listing: https://www.lairdthermal.com/products/thermoelectric-cooler-modules/peltier-cp-series/CP10-71-06-L1-W4.5 ; Wakefield micro TEC catalog: https://wakefieldthermal.com/content/catalogs/Micro%20Thermoelectric%20Coolers%20v11%20%281%29.pdf **[S]**
- S9 Boyd ultra-thin vapor chambers (0.4 mm Cu, 0.3 mm SS): https://www.boydcorp.com/thermal/two-phase-cooling/vapor-chambers/ultra-thin-vapor-chambers.html **[S]**
- S10 Ultra-thin vapor chamber with composite wick, 0.39 mm, 3473 W/m/K: https://mdpi.com/2072-666X/15/5/627 **[S]**
- S11 Extended-grade microSD datasheet example (-25 to 85 C): https://mm.digikey.com/Volume0/opasdata/d220001/medias/docus/8442/Swissbit_02-03-2025_S-50_data_sheet.pdf **[S]**
- S12 ESP32-S3 datasheet (temperature ranges by variant): https://documentation.espressif.com/esp32-s3_datasheet_en.html **[S]**; paraffin latent heat listing (RT58, 179 kJ/kg): https://Www.apaci.Com.au/RawPCMProducts/RawOrganicPCM.aspx **[S]**
- S13 Dark signal doubling every 6-8 C: https://arxiv.org/pdf/1412.4031 ; Teledyne: https://teledynevisionsolutions.com/learn/learning-center/imaging-fundamentals/dark-current ; Allied Vision (9 K): https://www.alliedvision.com/assets/support/Camera-Documentation/Allied-Vision/FAQs-Application-Notes/Image-Quality_vs_Temperature-on-SWIR-Cameras.pdf **[S]**
- S14 Convection and emissivity, low-trust: https://industrialmonitordirect.com/blogs/knowledgebase/calculating-natural-convection-cooling-limits-for-aluminum-enclosures ; https://eng-tips.com/threads/gray-body-radiation-equation-applicability.277493 **[S, forum/blog grade]**
- S15 Gap pad examples (TGP HC1000: 1.0 W/m/K; 2500S20: 2.4 W/m/K at 0.5 mm): https://www.mouser.com/datasheet/2/48/GP_HC1000-EN-1534567.pdf **[S]**
- S16 Conductivity values (Al about 215 in a worked example, polycarbonate 0.19, nylon 0.25, air 0.026 W/m/K): https://www.engineeringtoolbox.com/thermal-conductivity-d_429.html **[V, partial]**
- Repo documents: `docs/research/power-thermal.md`, `720p-xiao.md`, `processors.md`, `rear-processor.md`. Seeed XIAO camera heat note and board numbers are as cited there.
- Not read or not found: IEC 62368-1, IEC 60601-1, ISO 13732-1:2006 full text, ASTM C1055, Rubitherm RT44HC datasheet, Murata power consumption, copper conductivity page (390 W/m/K from handbook knowledge **[E]**).
