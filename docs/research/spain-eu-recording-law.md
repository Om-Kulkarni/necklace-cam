# Spain and EU rules for wearable recording (ticket #6)

Status: research note, 2026-11-21. **This is not legal advice.** It is a desk reading of primary sources by a non-lawyer; have a Spanish lawyer review before shipping. "(Uncertain)" marks interpretations I could not settle from the text.

Terms follow `CONTEXT.md`: Necklace, Pendant (front camera unit), Counterweight (rear battery unit), Capture, Clip, Recording Indicator.

## Plain-language conclusion

1. **No Spanish or EU rule I found requires a recording indicator light on a consumer camera used by a private individual.** Nothing in GDPR, LOPDGDD, LO 1/1982 or the Criminal Code mandates an LED. The only indicator-style guidance is non-binding: EDPB Statement 01/2026 on smart glasses and an AEPD blog post (both below).
2. **Purely personal use, shown only to family and friends, is outside GDPR** (Art. 2(2)(c)), but the exemption is read narrowly. Posting footage publicly, recording for work/commercial/political purposes, or continuously filming public space takes the user out of it (grey area for the last).
3. **Outside GDPR is not outside the law.** Spanish civil law on honor, privacy and image (LO 1/1982) and the Criminal Code (Art. 197) still apply. Secretly recording private moments, intimate settings, or conversations you are not part of is the real legal danger. Recording a conversation you take part in is generally not a breach of secrecy of communications in Spain, but publishing it can be unlawful.
4. **A rear-only indicator is legal but a poor design choice.** It satisfies no legal duty and no guidance, and it makes the device look covert, which regulators flag. Recommend a front-facing, hardware-forced indicator (optionally plus a wearer-side one on the Counterweight).

## Legal framework (what binds)

**GDPR scope.** GDPR does not apply to processing by a natural person in a purely personal or household activity (Art. 2(2)(c); Recital 18). The EUR-Lex page would not render for me, so wording is as quoted in EDPB Guidelines 3/2019 para 11 and AEPD guide s.4.3 and 6.1.

**CJEU case law** (decided under Directive 95/46; EDPB and AEPD treat it as carrying over to GDPR)
- *Lindqvist* C-101/01, paras 25-27 and 47: putting personal data on an internet page is "processing"; the household exemption covers only private or family life, "clearly not" publication to an indefinite number of people.
- *Rynes* C-212/13, paras 31-35: exemption is narrow; a camera that records, even partly, a public space is directed outward from the private sphere and is not purely household. Para 34 notes the controller's legitimate interests (property, health, life) can still be weighed.
- *Buivids* C-345/17, paras 35-36, 43-44 and operative part: recording to a camera's memory is automated processing; a one-off recording still counts; publishing on a public video site is outside the exemption; the journalistic exemption may apply only if the sole object is disclosing information, opinions or ideas to the public, for the national court to decide.
- Caveat (uncertain): these concern the old Directive and fixed cameras or uploads. The CJEU has not decided whether a worn, mobile camera storing locally is "directed outward". EDPB Guidelines 3/2019 para 13 places a holiday video shown to friends, and a remote-area action cam used for private viewing, inside the exemption.

**EDPB Guidelines 3/2019 (video devices, v2.0).** Exemption narrowly construed (paras 11-13). First-layer warning signs should be seen before someone enters the monitored area (paras 110-111). These duties bind controllers (users outside the exemption) and are written for fixed surveillance; applying them to a worn camera is by analogy (uncertain).

**EDPB Statement 01/2026 on smart glasses (adopted 13 Oct 2026)** - non-binding, "preliminary":
- Household exemption covers personal use such as sharing with family or friends only, not an indefinite public audience or professional, official or political use (paras 13-14). Criminal, civil or labour law may still apply (paras 15, 34).
- Producers and app providers can be controllers and must not rely on users for transparency (paras 16-18).
- Bystanders may be unaware; current indicator practice leaves many unaware (paras 21-22).
- Suggested in-the-moment transparency that is hard to disable and easy for bystanders to notice (para 24, fn 17): an LED visible in many lighting conditions and noticeable to people in the field of capture, active long enough to be understood, with **recording not possible while the indicator is covered**; also audio cues, phone proximity notices (not as sole method), and design cues. Fn 19: a white LED can be mistaken for glare and visually impaired people may miss LEDs.
- Sensitive settings: medical facilities, schools and places with minors, changing rooms, pools and beaches, toilets, workplaces (para 10). Children need particular care (para 32). Minimise continuous recording; vague safety worries do not justify it (para 33). Microphones may capture background conversations (para 7).
- It addresses smart glasses; applying it to a necklace camera is by analogy (uncertain), though the reasoning (discreet worn camera, unaware bystanders) fits.

**LOPDGDD (LO 3/2018)**
- Art. 2: excludes what GDPR Art. 2(2) excludes.
- Art. 22: video surveillance (purpose; public roads only as strictly necessary; one-month retention; information notice, 22.4; filming only one's own home is outside, 22.5). Designed for surveillance systems, not wearables.
- Art. 89: employers may use cameras for staff control only with prior clear notice; sound only where relevant security risks, proportionate and minimal; cameras and microphones banned in changing rooms, toilets and canteens.
- Art. 7: a minor's own consent valid from age 14; below that, parental consent. Art. 84: if sharing a minor's images online is an unlawful intrusion, the public prosecutor can act. Art. 92: schools and bodies working with minors need consent (per Art. 7) before publishing minors' data on social networks.
- Arts. 84, 89, 92 were read from a summary of the BOE text; verify wording.

**LO 1/1982 (honor, privacy, image; civil law)**
- Art. 2(2)-(3): no intrusion if law authorizes or the person consents; consent revocable.
- Art. 7: intrusions include using listening or recording devices to learn about intimate life or to record private manifestations or letters not addressed to the user (7.1-7.2); filming or publishing a person's image in private places or moments (7.5; whether the text also adds "or outside them" is to be verified); commercial use of voice or image (7.6).
- Art. 8(2): no breach in capturing or publishing images of public figures in public places or events, or incidental appearances in graphic reporting of public events.
- This is civil liability, outside data protection; the AEPD itself sends image disputes here (AEPD guide s.4.3, 6.2).

**Criminal Code (LO 10/1995), Art. 197**
- 197.1: using technical means to listen to, transmit, record or reproduce sound or image, without consent, to discover secrets or violate someone's privacy: 1-4 years prison plus fine. The intent element matters (uncertain how courts treat an overt consumer camera).
- 197.3: disseminating what was captured: 2-5 years.
- 197.5: higher penalty where the victim is a minor or a person with disability needing special protection, or data is sensitive.
- 197.7: sharing without consent images or recordings obtained with the person's consent in a home or other place out of sight of others, if it seriously harms their privacy: 3 months to 1 year or fine.
- Art. 201: prosecution generally needs a complaint by the victim, except where the victim is a minor or incapacitated, or many people or the general interest are affected.

**Recording conversations**
- You take part: STC 114/1984 (BOE 21 Dec 1984, ground 7, number to verify) says that recording a conversation with another person is not by itself a breach of Art. 18.3 of the Constitution, whereas recording others' conversation is. Later Supreme Court rulings reportedly follow it (e.g. STS 45/2014; seen only in secondary sources, not verified). Use or disclosure can still breach privacy (LO 1/1982 Art. 7; CP 197.3, 197.7).
- You do not take part (background chatter, a conversation recorded while you are absent): risk under CP 197.1, LO 1/1982 Art. 7.2 and Constitution Art. 18.3. A necklace microphone that picks up bystanders is the highest-risk feature.

## Guidance (not binding)

- AEPD *Guía sobre el uso de videocámaras* (Feb 2025): the household exemption is lost when information reaches an indefinite number of people (s.6.1); personal-use cameras on a cyclist's or motorcyclist's helmet filming scenery are domestic, but later online publication is subject to data protection (section on helmet and dash cameras; page not pinned); parents filming school events for family purposes are within the exemption (s.4.3); no cameras in changing rooms or toilets (section on private spaces).
- AEPD blog on smart sunglasses (13 Jul 2026): image and voice of identifiable people are personal data; avoid indiscriminate or continuous capture of third parties in public; **a recording light does not replace clear notice, especially where people may feel uncomfortable or expect privacy**; avoid capturing minors and vulnerable people unless necessary; recording may be restricted in bathrooms, changing rooms, public buildings and homes; think before sharing. No articles cited. I found no AEPD guidance specific to necklace or body-worn cameras.

## Scenario table

| Scenario | GDPR | Spanish civil/criminal risk | Practical reading |
|---|---|---|---|
| Walking in public; footage kept private or shown to family | Likely household exemption (EDPB 3/2019 para 13 examples; AEPD helmet-cam view). Continuous capture of public space is the grey area (uncertain). | Low if not targeting or harassing people (LO 1/1982 Art. 8(2)). | Fine in practice; avoid indiscriminate continuous recording (AEPD blog). |
| Same footage posted publicly or live-streamed | Exemption lost (Lindqvist 47; Buivids 43). User is a controller: legal basis, transparency, rights. Journalistic exemption only if sole purpose is informing the public. | Image right: consent normally needed unless Art. 8(2) applies; CP 197.3 if the capture was itself unlawful. | Blur or get consent for bystanders; highest-exposure activity. |
| Conversation you take part in | Same as above for storage and sharing | Recording generally not a secrecy breach (STC 114/1984); disclosure may be unlawful (LO 1/1982 Art. 7; CP 197.7). | Fine to keep; do not publish. Tell people where practical. |
| Conversations between others (background audio) | Personal data if identifiable | CP 197.1, LO 1/1982 Art. 7.2, Constitution Art. 18.3 | Avoid; audio opt-in, muted by default. |
| Someone's home (as guest) | Household possible | Intimate settings: LO 1/1982 7.1, 7.5; CP 197.1, 197.7 if shared | Get host consent; no recording in private rooms. |
| Workplace | Not household if professional; LOPDGDD Art. 89 (notice, no sound unless justified, no changing rooms or toilets) | Labour discipline; confidentiality; intrusion claims | Off by default unless the employer allows. |
| Schools, places with minors | Parent filming own child at family events: household (AEPD s.4.3). Other children: consent rules (LOPDGDD Art. 7, 92). | CP 197.5 for minors; LOPDGDD Art. 84 prosecutor; school rules | Avoid or ask; never publish other children. |
| Gyms, changing rooms, toilets, pools | Named as sensitive (EDPB 01/2026 para 10) | Highest risk: LO 1/1982 7.1, 7.5; CP 197.1, 197.3 | Do not record; consider geofencing (EDPB para 33). |
| Healthcare settings | Health data, sensitive | As above | Do not record. |

## Indicator recommendation

**Legal requirement versus good practice**
- *Required:* nothing found that mandates any indicator for a private individual. If the user becomes a controller (public posting, professional use), GDPR transparency (Arts. 12-14, cited at EDPB 01/2026 para 24) applies, but it does not prescribe a light and a light alone would not satisfy it. The Statement says producers should not rely on users for transparency (para 18); the company could be a controller or joint controller for cloud or AI features (uncertain, depends on architecture).
- *Strongly recommended by regulators (soft law):* in-the-moment, hard-to-disable, bystander-visible signals (EDPB 01/2026 para 24); AEPD says a light supplements, not replaces, telling people.
- *Our design choice:* for a personal memory camera, defensibility rests on not being covert.

**Back-of-device indicator.** A light on the Counterweight (rear of the neck) is seen by the wearer and people behind them, not by people in front being filmed. It would not meet the EDPB test of being noticeable to individuals within the field of collection, it cuts against the AEPD point that people should be able to tell they are recorded, and it invites a "covert recording" characterization relevant to LO 1/1982 Art. 7 and CP 197.1 intent arguments (uncertain how a court would weigh it; an indicator neither creates nor removes liability on its own). Lawful to ship, but weak evidence of good faith.

**Recommendation**
1. Front-facing Recording Indicator on the Pendant, hardware-forced on whenever the camera or microphone captures, with recording inhibited if it is covered or fails (EDPB para 24).
2. Bright and distinct from ambient light, not a lone white LED (fn 19); consider a start-of-recording chirp.
3. A wearer-side indicator on the Counterweight is fine as an addition, not a substitute.
4. Audio off by default or clearly separate, with an explicit in-app label.
5. In-app guidance: no recording in changing rooms, toilets, schools, healthcare; get consent before publishing bystanders; stop when asked.
6. Data-protection-by-design review for any cloud, AI or sharing feature (GDPR Art. 25, cited at EDPB 01/2026 fn 21 and 32).

## Open questions

- Does continuous recording of public space by a worn camera stored only locally count as "directed outward" under *Rynes*? No CJEU or AEPD ruling on wearables found.
- Will the AEPD or Spanish courts apply the EDPB smart-glasses statement to a necklace camera?
- Is the company a (joint) controller for the phone app, Transfer or any cloud processing, and so owing Art. 13/14 information to bystanders?
- How far does the intent requirement in CP 197.1 protect an overt consumer camera, and how would a hidden-looking design affect that?
- Exact wording of LO 1/1982 Art. 7.5 and LOPDGDD Arts. 84, 89, 92 against the BOE consolidated text.
- Not reviewed: ePrivacy and telecom rules for the device (EDPB 01/2026 para 32), product safety and radio equipment rules, EU AI Act if recognition features are added, regional or venue rules.

## Sources

- GDPR (Reg. 2016/679), Art. 2(2)(c), Recital 18, Arts. 6, 12-14, 25: https://eur-lex.europa.eu/eli/reg/2016/679/oj (did not render for me; relied on EDPB/AEPD quotations)
- Lindqvist C-101/01 (paras 25-27, 47): https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:62001CJ0101
- Ryneš C-212/13 (paras 31-35): https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:62013CJ0212
- Buivids C-345/17 (paras 35-36, 43-44, operative part): https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:62017CJ0345
- EDPB Guidelines 3/2019 on video devices (paras 11-13, 110-111): https://www.edpb.europa.eu/sites/default/files/files/file1/edpb_guidelines_201903_video_devices_en.pdf
- EDPB Statement 01/2026 on smart glasses: https://www.edpb.europa.eu/system/files/2026-11/20261013_edpb_012026_statement_smart-glasses_en.pdf and https://www.edpb.europa.eu/news/edpb-publishes-statement-on-the-data-protection-challenges-of-smart-glasses_en
- LOPDGDD, LO 3/2018 (Arts. 2, 7, 22, 84, 89, 92): https://www.boe.es/buscar/act.php?id=BOE-A-2018-16673
- LO 1/1982 (Arts. 2, 7, 8): https://www.boe.es/buscar/act.php?id=BOE-A-1982-11196
- Criminal Code, LO 10/1995 (Arts. 197, 201): https://www.boe.es/buscar/act.php?id=BOE-A-1995-25444
- STC 114/1984: https://www.boe.es/boe/dias/1984/12/21/pdfs/T00028-00033.pdf
- AEPD, Guía sobre el uso de videocámaras (Feb 2025; s.3.7.2 area, 4.3, 6.1-6.2): https://www.aepd.es/guias/guia-videovigilancia.pdf
- AEPD blog, smart sunglasses (13 Jul 2026): https://www.aepd.es/prensa-y-comunicacion/blog/buenas-practicas-si-vas-a-usar-gafas-de-sol-inteligentes-este-verano
