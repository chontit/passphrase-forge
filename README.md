<div align="center">

# ⬡ PASSPHRASE FORGE

### Entropy-to-Key Generator · เครื่องมือสร้างรหัสผ่านความปลอดภัยสูงจากเอนโทรปีทางกายภาพ

**_Don't Trust, Verify._**

![Network](https://img.shields.io/badge/network-none%20(CSP%20enforced)-2ea44f)
![Dependencies](https://img.shields.io/badge/dependencies-zero-2ea44f)
![Build](https://img.shields.io/badge/build-single%20file%20HTML-blue)
![Runs on](https://img.shields.io/badge/runs%20on-Tails%20%2F%20Tor%20%2F%20file%3A%2F%2F-9146ff)
![License](https://img.shields.io/badge/license-MIT-lightgrey)
![Version](https://img.shields.io/badge/version-2.0.0-ff9f1c)

**🇬🇧 [English](#-english)   ·   🇹🇭 [ภาษาไทย](#-ภาษาไทย)**

</div>

---

## 🔐 Verify File Integrity · ตรวจสอบความสมบูรณ์ของไฟล์

**EN —** Before running the tool, verify that your copy of `passphrase-forge.html` is authentic and unmodified. Compute its SHA-256 hash and confirm it matches the value below (comparison is case-insensitive). If it does **not** match, do not trust the file.

**TH —** ก่อนใช้งาน ให้ตรวจว่าไฟล์ `passphrase-forge.html` ที่คุณมีเป็นของแท้ ไม่ถูกแก้ไข โดยคำนวณค่า SHA-256 แล้วเทียบกับค่าด้านล่าง (ไม่สนตัวพิมพ์เล็ก/ใหญ่) — ถ้า **ไม่ตรง** อย่าใช้ไฟล์นั้น

```
SHA-256 (passphrase-forge.html):
F4FFEA165EB651853D33C0D66D105201734387D4342EF6BA5932958C84E6CE96
```

| OS | Command / คำสั่ง |
|---|---|
| **Windows** — PowerShell | `Get-FileHash .\passphrase-forge.html -Algorithm SHA256` |
| **Windows** — CMD (CertUtil) | `certutil -hashfile passphrase-forge.html SHA256` |
| **macOS** — Terminal | `shasum -a 256 passphrase-forge.html` |
| **Linux / Tails** — Terminal | `sha256sum passphrase-forge.html` |

> **EN —** PowerShell prints uppercase; `shasum` / `sha256sum` print lowercase — both are correct. This check proves the file wasn't *tampered with*; reading the source (the CSP + `generate()`) proves what it *does*. If you edit the file, its hash changes — recompute it.
>
> **TH —** PowerShell ให้ตัวพิมพ์ใหญ่ ส่วน `shasum` / `sha256sum` ให้ตัวพิมพ์เล็ก — ถูกทั้งคู่ · การตรวจนี้พิสูจน์ว่าไฟล์ *ไม่ถูกดัดแปลง* ส่วนการอ่านซอร์ส (CSP + `generate()`) พิสูจน์ว่ามัน *ทำอะไร* · ถ้าแก้ไฟล์ ค่า hash จะเปลี่ยน ต้องคำนวณใหม่

---

## 🇬🇧 English

### Overview

**PASSPHRASE FORGE** is a single-file, fully offline HTML tool that turns **physical entropy** — coin flips, dice rolls, or playing cards — into a high-security passphrase. Every character position is filled by a **provably uniform** random choice, with **no modulo bias**, **no hashing you have to trust**, and **zero network calls**.

It is built for generating and backing up **high-value, long-lived secrets** — for example a **BIP39 passphrase** (the "25th word") — on an air-gapped machine such as **Tails OS**.

### Features

| Feature | Detail |
|---|---|
| **Entropy sources** | Coin (1 bit) · D6 (2.585 bit) · D16 (4 bit) · 52-card deck (~5.70 bit, decreasing) |
| **Output** | A–Z / a–z / 0–9 / editable symbols · length slider **12–64** · security floor 80–256 bit with auto-length |
| **Ambiguity filter** | **4 nested levels** (L3 ⊂ L2 ⊂ L1 ⊂ L0). L0 full (K=77) · L1 drops `0 O o 1 l I` (K=71) · L2 also drops `8/B 6/G/b 5/S/s 2/Z/z 9/g/q` (K=60) · L3 uppercase+digits only, no case, no symbols (K=27). Smaller K = fewer bits/char, so the tool raises the required length to hold the same total entropy |
| **Simulate rolls** | Optional `SIMULATE ROLLS (CSPRNG)` button fills the entropy pool from `crypto.getRandomValues` using the same unbiased rejection sampling. **Test / low-stakes only** — the result is flagged SIMULATED in the status line, the verify panel (`entropy origin`) and the printed backup card. Never use it for a wallet seed |
| **System salt (optional)** | Mixes a CSPRNG value so output isn't deterministic from dice alone |
| **Backup card** | Printable, grouped passphrase + **CRC32** typo-check + timestamped filename + tool version and filter level |
| **Verify panel** | Shows `E`, `R`, `K^L`, rejection limit, entropy origin, ambiguity level, and a per-character base-K map |
| **Extras** | Output masking · link-state readout with a **TEST EGRESS** CSP self-check · dark/light theme · panic wipe |

### How it works (the principle)

The security is **mathematical**, and every intermediate value is displayed so you can re-check it by hand.

**Notation:** `K` = alphabet size · `L` = passphrase length · `r` = a device's number of faces (radix) · `d` = a single roll's value.

1. **Collect (mixed-radix).** Each roll becomes one digit of a big number:
   `E = E·r + d` and `R = R·r` (computed with `BigInt`).
   - Coin `r=2`, D6 `r=6`, D16 `r=16`.
   - **Cards** use draw-without-replacement (a *Lehmer code*): the radix decreases 52 → 51 → 50 … and `d` is the drawn card's rank among the cards still in the deck. A full deck carries `log2(52!) ≈ 225.58` bits.
   - `E` is the collected entropy as one integer in `[0, R)`; `R` is the total number of possible roll sequences.

2. **Gate.** Generation is blocked until `R ≥ K^L` — i.e. you have collected at least as much entropy as the output space needs.

3. **Remove modulo bias (rejection sampling).** Simply taking `E mod K^L` would make some characters slightly more likely (because `R` is usually not a multiple of `K^L`). Instead the tool accepts `E` only if `E < ⌊R / K^L⌋ · K^L`; otherwise it asks for one more roll. The accepted value `U = E mod K^L` is then **exactly uniform** over all `K^L` possibilities.

4. **Decode (base conversion).** `U` is written in base `K`; each digit selects one character. The result is equivalent to independently drawing each of the `L` positions uniformly from the `K`-symbol alphabet.

5. **(Optional) Salt.** A value `S` is drawn uniformly from `[0, R)` using `crypto.getRandomValues`, then `E' = (E + S) mod R`. This whitens the physical input, so even biased or observed dice can't predict the output. Salt is *defense-in-depth* — it does **not** reduce the amount of physical entropy you must collect.

> **No hashing is used.** The output entropy equals the verified physical input entropy (`L · log2(K)` bits). There is nothing to trust but arithmetic you can re-check. The effective strength is `min(input entropy, output space)` — which is why the tool enforces `R ≥ K^L`.

**No-exfiltration guarantee.** The page ships a strict Content-Security-Policy:

```html
<meta http-equiv="Content-Security-Policy"
      content="default-src 'none'; connect-src 'none'; ...">
```

`connect-src 'none'` blocks **all** `fetch` / XHR / WebSocket. There is no CDN, no web font, no analytics — the file cannot phone home. This CSP, **not** any on-screen status light, is the real security boundary — and the header **TEST EGRESS** button lets you prove it is enforced.

### Usage — step by step

**Prerequisites:** a browser (Tor Browser or any modern browser). For real secrets, use an air-gapped machine (Tails OS recommended).

1. **Get offline.** Copy `passphrase-forge.html` to a USB / Tails persistent storage. Open it via `file://`. On Tor Browser, allow JavaScript for this local page. **Disconnect the network** (or use Tails offline mode). Note that `navigator.onLine` is not a reliable airgap indicator — on Tails / Tor Browser it stays `true` even with the network pulled, so the header reports **LINK STATE UNVERIFIED** rather than claiming you are online. The real barrier is the CSP `connect-src 'none'`: press **TEST EGRESS** in the header to prove it, and look for **EGRESS BLOCKED (verified)**.

2. **`[01]` Output spec.** Tick the character classes you want (A–Z / a–z / 0–9 / symbols) and edit the symbol set if desired. Pick an **Ambiguity filter** level (L0–L3) — L2 for hand-copying, L3 for handwriting, OCR or metal stamping. Set a **Security floor** (80–256 bit); with **Auto-length** on, `L = ceil(floor / log2 K)` is applied automatically whenever `K` changes. Moving the slider by hand turns Auto-length off and shows how many characters you are short. The panel shows `K`, bits/char, min length at the floor, target entropy, and a strength rating.

3. **`[02]` Source.** Choose Coin, D6, D16, or Cards. The help line tells you how many rolls you need (with a small recommended buffer so rejection ≈ 0).

4. **`[03]` Input.** Physically roll/flip/draw and type the results. The field accepts only valid characters for the chosen source:
   - **Coin:** `0 1` or `H T`
   - **D6:** `1`–`6`
   - **D16:** `0`–`9`, `A`–`F`
   - **Cards:** rank + suit, e.g. `AS KH 10D 7C QS` (suits `S H D C`; type them run-together like `askh10d7c` and spacing is added automatically; reshuffle and keep drawing after 52).
   Watch the **ENTROPY POOL** bar fill to **READY**.

5. **`[04]` Derive.** Optionally toggle **✦ SALT** (system-entropy mixing). Press **GENERATE**.

6. **`[05]` Output.** Copy the passphrase, or press **BACKUP CARD → PRINT** for a printable card containing the grouped passphrase, a **CRC32 checksum**, and (if salt is off) the recorded rolls. Open the **Verify** panel to audit `E`, `R`, `K^L`, and the character map by hand. When done, press **PANIC WIPE**.

**Backup verification workflow:** after copying the passphrase by hand, type it back into any field and compare its CRC32 to the one on the card — if they match, you copied it correctly.

### Verification & Audit

- **Core math tested:** uniformity confirmed (χ²/df ≈ 1.0 over 200k samples); CRC32 checked against known vectors (`123456789` → `CBF43926`); card entropy verified as `log2(52!) ≈ 225.58` bit.
- **Dual-AI cross-review:** independently audited by two separate LLMs. Valid findings (custom-symbol overwrite, deprecated clipboard API, unbounded CSPRNG loop) were fixed; mismatched "best-practice" suggestions (e.g. swapping the CRC32 *typo* checksum for SHA-256) were reasoned through and declined with rationale.
- **Verify it yourself:** the file is human-readable — read the CSP, read `generate()`, and hand-check the Verify panel.

### Security notes & disclaimer

- Personal project, **not** a professionally audited product. Review the source before trusting it with real value.
- **Use fair dice/coins.** Rejection sampling removes *conversion* bias, not *source* bias — biased physical entropy can be weaker than a good CSPRNG.
- **The passphrase is a secret.** Anyone with it (plus a known mnemonic) controls the funds. Store the backup card like cash/gold.
- **Test before funding.** Generate a throwaway and verify the full `seed + passphrase → address` pipeline **offline** first.
- With **SALT off**, the exact rolls + settings reproduce the passphrase; with **SALT on**, they do not — back up the passphrase itself.
- Provided **as-is, without warranty of any kind**. You are responsible for your own keys and funds.

---

## 🇹🇭 ภาษาไทย

### ภาพรวม

**PASSPHRASE FORGE** คือเครื่องมือรูปแบบ HTML ไฟล์เดียวที่ทำงานแบบออฟไลน์ 100% ออกแบบมาเพื่อแปลง **เอนโทรปีทางกายภาพ (Physical Entropy)** — เช่น การโยนเหรียญ ทอยลูกเต๋า หรือจั่วไพ่ — ให้กลายเป็นรหัสผ่านที่มีความปลอดภัยสูง ตัวอักษรในทุกตำแหน่งถูกสุ่มด้วยความน่าจะเป็นที่เท่ากันอย่างพิสูจน์ได้ (Provably Uniform) ปราศจากความเอนเอียงทางคณิตศาสตร์ (No Modulo Bias) ไม่พึ่งพากระบวนการ Hash ที่ตรวจสอบไม่ได้ และ **ไม่มีการเชื่อมต่อเครือข่ายโดยเด็ดขาด**

เหมาะสำหรับการสร้างและสำรอง **ความลับที่มีมูลค่าสูงและต้องเก็บรักษาระยะยาว** เช่น **BIP39 Passphrase** (คำที่ 25) โดยใช้งานบนเครื่องที่ตัดขาดจากเครือข่าย (Air-gapped Machine) อย่าง **Tails OS**

### ความสามารถ

| ฟีเจอร์ | รายละเอียด |
|---|---|
| **แหล่งกำเนิด Entropy** | เหรียญ (1 bit) · ลูกเต๋า D6 (2.585 bit) · ลูกเต๋า D16 (4 bit) · ไพ่ 52 ใบ (~5.70 bit และลดลงเรื่อย ๆ ตามจำนวนไพ่ที่เหลือ) |
| **รูปแบบผลลัพธ์** | รองรับ A–Z / a–z / 0–9 / สัญลักษณ์ (แก้ไขเองได้) · ปรับความยาวได้ตั้งแต่ **12–64** ตัวอักษร · กำหนดระดับความปลอดภัยขั้นต่ำ (Security Floor) ได้ตั้งแต่ 80–256 bit พร้อมระบบปรับความยาวอัตโนมัติ |
| **ตัวกรองอักขระกำกวม (Ambiguity Filter)** | **กรองได้ 4 ระดับซ้อนกัน** (L3 ⊂ L2 ⊂ L1 ⊂ L0) · L0 ใช้ชุดเต็ม (K=77) · L1 ตัด `0 O o 1 l I` ออก (K=71) · L2 ตัดเพิ่ม `8/B 6/G/b 5/S/s 2/Z/z 9/g/q` (K=60) · L3 เหลือเพียงตัวพิมพ์ใหญ่กับตัวเลข ไม่มีตัวพิมพ์เล็กและสัญลักษณ์ (K=27) · ยิ่งกรองออกมาก จำนวน bit ต่อตัวอักษรยิ่งลดลง ระบบจะคำนวณเพิ่มความยาวรหัสผ่านชดเชยให้อัตโนมัติ |
| **ระบบจำลองการสุ่ม (Simulate Rolls)** | ปุ่ม `SIMULATE ROLLS (CSPRNG)` เติมเอนโทรปีให้อัตโนมัติจากตัวสุ่มของระบบ (`crypto.getRandomValues`) ด้วย rejection sampling แบบเดียวกับเส้นทางหลัก · **ใช้สำหรับทดสอบระบบหรือรหัสผ่านที่ไม่สำคัญเท่านั้น** ผลลัพธ์จะถูกติดธง SIMULATED ทั้งในแถบสถานะ แผงตรวจสอบ (`entropy origin`) และบนบัตรสำรองที่พิมพ์ออกมา · **ห้ามใช้กับ Wallet Seed เด็ดขาด** |
| **System Salt (ทางเลือกเสริม)** | ผสมค่าจาก CSPRNG เข้าไปในกระบวนการ เพื่อไม่ให้ผลลัพธ์ผูกติดกับค่าลูกเต๋าเพียงอย่างเดียว |
| **บัตรสำรองข้อมูล (Backup Card)** | สั่งพิมพ์ออกทางเครื่องพิมพ์ได้ · รหัสผ่านจัดกลุ่มให้อ่านง่าย + เช็กซัม **CRC32** ป้องกันการจดผิด + ชื่อไฟล์ระบุวันเวลา + บันทึกเวอร์ชันเครื่องมือและระดับ Filter ที่ใช้ |
| **แผงตรวจสอบ (Verify Panel)** | แสดงกระบวนการทางคณิตศาสตร์อย่างโปร่งใส ทั้งค่า `E`, `R`, `K^L`, ขอบเขตการคัดกรอง (Rejection Limit), แหล่งที่มาของเอนโทรปี, ระดับ Filter และตารางแปลงเลขฐาน K แบบตัวต่อตัว |
| **อื่น ๆ** | ซ่อนรหัสผ่านขณะแสดงผล (Input Masking) · แถบสถานะการเชื่อมต่อพร้อมปุ่มทดสอบ **TEST EGRESS** · ธีมมืด/สว่าง · ปุ่ม Panic Wipe เพื่อลบข้อมูลบนหน้าจอทันที |

### หลักการทำงาน

ความปลอดภัยของระบบนี้ตั้งอยู่บน **คณิตศาสตร์** ล้วน ๆ โดยค่าตัวแปรระหว่างทางทั้งหมดจะถูกแสดงไว้บนหน้าจอ เพื่อให้คุณคำนวณตรวจสอบซ้ำด้วยตนเองได้

**คำศัพท์ที่ใช้:** `K` = ขนาดของชุดตัวอักษร · `L` = ความยาวของรหัสผ่าน · `r` = จำนวนหน้าของอุปกรณ์สุ่ม (ฐานเลข) · `d` = ผลลัพธ์จากการสุ่มแต่ละครั้ง

1. **รวบรวมเอนโทรปี (Mixed-radix)** — ผลการสุ่มแต่ละครั้งถูกนำไปต่อกันเป็นเลขจำนวนเต็มขนาดใหญ่ตัวเดียว: `E = E·r + d` และ `R = R·r` (ประมวลผลด้วย `BigInt`)
   - เหรียญ `r=2` · ลูกเต๋า D6 `r=6` · ลูกเต๋า D16 `r=16`
   - **ไพ่** ใช้หลักการจั่วแบบไม่ใส่คืน (*Lehmer Code*) ฐานจะลดลงเรื่อย ๆ จาก 52 → 51 → 50 … และ `d` คือลำดับของไพ่ที่จั่วได้เทียบกับกองที่เหลือ · ไพ่หนึ่งสำรับเต็มให้เอนโทรปีประมาณ `log2(52!) ≈ 225.58` bit
   - `E` คือเอนโทรปีที่รวบรวมได้ในรูปเลขจำนวนเต็มในช่วง `[0, R)` ส่วน `R` คือจำนวนลำดับการสุ่มที่เป็นไปได้ทั้งหมด

2. **ตรวจสอบปริมาณเอนโทรปี (Gate)** — ระบบจะไม่ยอมให้สร้างรหัสผ่านจนกว่า `R ≥ K^L` นั่นคือคุณต้องรวบรวมเอนโทรปีให้มากพอครอบคลุมความเป็นไปได้ทั้งหมดของผลลัพธ์

3. **กำจัดความเอนเอียง (Rejection Sampling)** — หากนำ `E mod K^L` มาใช้ตรง ๆ ตัวอักษรบางตัวจะมีโอกาสออกบ่อยกว่าตัวอื่นเล็กน้อย (เพราะ `R` มักหารด้วย `K^L` ไม่ลงตัว) ระบบจึงยอมรับค่า `E` ก็ต่อเมื่อ `E < ⌊R / K^L⌋ · K^L` เท่านั้น หากเกินขอบเขตนี้จะแจ้งให้สุ่มเพิ่ม · ค่า `U = E mod K^L` ที่ผ่านเกณฑ์แล้วจะกระจายตัว **เท่ากันอย่างสมบูรณ์ (Uniform)** ทั่วทั้งขอบเขต `K^L`

4. **แปลงกลับเป็นตัวอักษร (Base Conversion)** — นำค่า `U` มาเขียนในเลขฐาน `K` โดยแต่ละหลักจับคู่กับตัวอักษรหนึ่งตัว ผลลัพธ์เทียบเท่ากับการ **สุ่มตัวอักษรแต่ละตำแหน่งอย่างเป็นอิสระด้วยความน่าจะเป็นเท่ากัน** จากชุดสัญลักษณ์ `K` ตัว

5. **เติม System Salt (ทางเลือกเสริม)** — สุ่มค่า `S` ในช่วง `[0, R)` ด้วย `crypto.getRandomValues` แล้วผสมผ่านสมการ `E' = (E + S) mod R` เพื่อเจือจางข้อมูลนำเข้า (Whiten Input) ทำให้แม้ใช้ลูกเต๋าที่ลำเอียงหรือถูกแอบดูผลการทอย ก็ยังเดารหัสผ่านไม่ได้ · Salt เป็นเพียงเกราะชั้นเสริม (*Defense-in-depth*) และ **ไม่ได้ช่วยลด** ปริมาณเอนโทรปีทางกายภาพที่คุณต้องรวบรวม

> **ระบบนี้ไม่มีการใช้ Hashing** — เอนโทรปีของผลลัพธ์เท่ากับเอนโทรปีทางกายภาพที่ป้อนเข้ามาพอดี (`L · log2(K)` bit) ไม่มีอัลกอริทึมลับให้ต้องเชื่อ มีแต่คณิตศาสตร์ที่คุณตรวจสอบเองได้ · ความแข็งแกร่งที่แท้จริง = `min(เอนโทรปีที่ป้อน, พื้นที่ความเป็นไปได้ของผลลัพธ์)` นี่คือเหตุผลที่ระบบบังคับ `R ≥ K^L` เสมอ

**การรับประกันว่าไม่มีข้อมูลรั่วไหล** — หน้าเว็บนี้ฝัง Content-Security-Policy ที่เข้มงวดไว้:

```html
<meta http-equiv="Content-Security-Policy"
      content="default-src 'none'; connect-src 'none'; ...">
```

คำสั่ง `connect-src 'none'` บล็อก `fetch` / XHR / WebSocket **ทั้งหมด** ไม่มีการเรียกใช้ CDN, Web Font หรือสคริปต์ Analytics ภายนอก — ไฟล์นี้ส่งข้อมูลออกไปไหนไม่ได้ · **นโยบาย CSP นี้** คือเส้นแบ่งความปลอดภัยที่แท้จริง ไม่ใช่ไฟสถานะบนหน้าจอ

### วิธีใช้งาน — ทีละขั้น

**การเตรียมความพร้อม:** ใช้เบราว์เซอร์ยุคใหม่หรือ Tor Browser · หากนำไปใช้กับความลับที่มีมูลค่าจริง ควรใช้เครื่องที่ไม่ได้เชื่อมต่อเครือข่าย (Air-gapped) เช่น Tails OS

1. **ตัดการเชื่อมต่อ (Go Offline)** — คัดลอกไฟล์ `passphrase-forge.html` ลง USB หรือที่เก็บข้อมูลถาวรของ Tails แล้วเปิดผ่านโปรโตคอล `file://` (หากใช้ Tor Browser ให้อนุญาตการรันสคริปต์สำหรับหน้านี้) จากนั้น **ถอดสายแลนหรือปิด Wi-Fi**
   > **ข้อควรทราบ:** ฟังก์ชัน `navigator.onLine` ใช้ยืนยันการตัดขาดเครือข่ายไม่ได้ — บน Tails/Tor Browser ค่านี้เป็น `true` เสมอแม้ตัดการเชื่อมต่อแล้ว ส่วนหัวจึงแสดง **LINK STATE UNVERIFIED** แทนที่จะกล่าวหาว่าคุณกำลังออนไลน์ · ปราการที่แท้จริงคือ CSP `connect-src 'none'` ทดสอบได้ด้วยการกดปุ่ม **TEST EGRESS** แล้วสังเกตข้อความ **EGRESS BLOCKED (verified)**

2. **`[01]` กำหนดรูปแบบผลลัพธ์ (Output Spec)** — เลือกชุดอักขระที่ต้องการ (A–Z / a–z / 0–9 / สัญลักษณ์) และแก้ไขชุดสัญลักษณ์ได้ตามต้องการ · เลือกระดับ **Ambiguity Filter** L0–L3 (L2 เหมาะกับการคัดลอกด้วยมือ · L3 เหมาะกับการเขียนด้วยลายมือ, OCR หรืองานตอกสลักแผ่นโลหะ) · กำหนด **Security Floor** (80–256 bit) หากเปิด **Auto-length** ระบบจะตั้งค่า `L = ceil(floor ÷ log2 K)` ให้อัตโนมัติทุกครั้งที่ `K` เปลี่ยน · หากเลื่อนแถบความยาวเอง ระบบจะปิด Auto-length แล้วแจ้งว่าคุณยังขาดอีกกี่ตัวอักษร · แผงสถานะแสดงค่า `K`, จำนวน bit ต่อตัวอักษร, ความยาวขั้นต่ำที่ต้องใช้, เอนโทรปีเป้าหมาย และระดับความแข็งแกร่ง

3. **`[02]` เลือกแหล่งอ้างอิง (Source)** — เลือกเหรียญ, ลูกเต๋า D6, ลูกเต๋า D16 หรือไพ่ 52 ใบ · ระบบจะคำนวณล่วงหน้าว่าต้องสุ่มกี่ครั้ง (พร้อมเผื่อไว้เล็กน้อยเพื่อให้โอกาสถูก Reject ใกล้ศูนย์)

4. **`[03]` ป้อนข้อมูลการสุ่ม (Input)** — ทอยลูกเต๋า โยนเหรียญ หรือจั่วไพ่จริง แล้วพิมพ์ผลลัพธ์ลงในช่อง (ช่องนี้รับเฉพาะตัวอักษรที่ตรงกับแหล่งอ้างอิงที่เลือก)
   - **เหรียญ (Coin):** `0 1` หรือ `H T`
   - **ลูกเต๋า (D6):** `1`–`6`
   - **ลูกเต๋า 16 หน้า (D16):** `0`–`9`, `A`–`F`
   - **ไพ่ (Cards):** อันดับไพ่ตามด้วยดอก เช่น `AS KH 10D 7C QS` (ดอกไพ่ใช้ `S H D C` · พิมพ์ติดกันเป็น `askh10d7c` ได้เลย ระบบจะเว้นวรรคให้เอง · หากจั่วครบ 52 ใบ ให้สับใหม่แล้วจั่วต่อได้ทันที)

   สังเกตแถบ **ENTROPY POOL** จนขึ้นสถานะ **READY** · หากต้องการเพียงทดสอบระบบ กดปุ่ม **🎲 SIMULATE ROLLS** เพื่อให้เครื่องเติมค่าให้ (ผลลัพธ์จะถูกติดธง SIMULATED และห้ามนำไปใช้จริง)

5. **`[04]` ประมวลผล (Derive)** — เลือกเปิดหรือปิด **✦ SALT** (นำเอนโทรปีของเครื่องมาผสม) แล้วกด **GENERATE**

6. **`[05]` ผลลัพธ์และสำรองข้อมูล (Output)** — คัดลอกรหัสผ่าน หรือกด **BACKUP CARD → PRINT** เพื่อพิมพ์บัตรสำรองที่มีรหัสผ่านจัดกลุ่มให้อ่านง่าย, เช็กซัม **CRC32**, เวอร์ชันเครื่องมือ, ระดับ Filter และผลการสุ่มทั้งหมด (เฉพาะเมื่อปิด Salt) · เปิดแผง **Verify** เพื่อตรวจสอบค่า `E`, `R`, `K^L` และตารางแปลงตัวอักษรด้วยตาเปล่า · เมื่อเสร็จสิ้นให้กด **PANIC WIPE**

**ข้อแนะนำในการคัดลอกด้วยมือ:** หลังจดรหัสผ่านลงกระดาษแล้ว ให้พิมพ์กลับเข้าไปในช่องข้อความใดก็ได้ แล้วเทียบค่า CRC32 กับบนบัตร — หากตรงกัน แสดงว่าคัดลอกถูกต้องครบทุกตัวอักษร

### การตรวจสอบและ Audit

- **ผ่านการทดสอบแกนกลางคณิตศาสตร์** — ยืนยันการกระจายตัวแบบ Uniform (χ²/df ≈ 1.0 จากกลุ่มตัวอย่าง 200k) · ตรวจสอบ CRC32 กับ Known Vectors แล้ว (`123456789` → `CBF43926`) · ยืนยันเอนโทรปีของไพ่ที่ `log2(52!) ≈ 225.58` bit · ตัวสุ่มจำลอง (Simulate) ผ่าน chi-square 120,000 ตัวอย่างต่อค่า
- **ผ่านการตรวจทานไขว้โดย AI สองระบบ (Dual-AI cross-review)** — โค้ดถูก Audit โดย LLM ต่างรุ่นกันอย่างเป็นอิสระ · ข้อบกพร่องที่พบจริง (การเขียนทับชุดสัญลักษณ์ที่ผู้ใช้กำหนดเอง, Clipboard API ที่ล้าสมัย, ลูป CSPRNG ที่ไม่มีขอบเขต) ได้รับการแก้ไขแล้ว · ส่วนข้อเสนอที่ไม่ตรงกับ Threat Model (เช่น เสนอให้เปลี่ยนเช็กซัมจาก CRC32 เป็น SHA-256 ทั้งที่มีไว้เพียงกันพิมพ์ผิด) ได้รับการพิจารณาและปฏิเสธพร้อมเหตุผลกำกับ
- **ระบบโปร่งใส ตรวจสอบเองได้** — ซอร์สโค้ดในไฟล์อ่านทำความเข้าใจได้ คุณสามารถตรวจนโยบาย CSP, ฟังก์ชัน `generate()` และไล่ดูตัวแปรในแผง Verify ด้วยตนเอง

### ข้อควรระวังและข้อจำกัดความรับผิด

- โปรเจกต์นี้เป็นโปรเจกต์ส่วนตัว **ไม่ใช่** ผลิตภัณฑ์ที่ผ่านการ Audit ระดับองค์กร · กรุณาตรวจสอบซอร์สโค้ดด้วยตนเองก่อนนำไปใช้กับสินทรัพย์ที่มีมูลค่าจริง
- **ใช้ลูกเต๋าหรือเหรียญที่ได้มาตรฐาน** · Rejection Sampling แก้ความเอนเอียงที่เกิดจาก *กระบวนการแปลงค่า (Conversion Bias)* แต่แก้ความเอนเอียงจาก *แหล่งกำเนิด (Source Bias)* ไม่ได้ — ลูกเต๋าที่ไม่ได้มาตรฐานอาจให้ความปลอดภัยต่ำกว่า CSPRNG ทั่วไปเสียอีก
- **ห้ามใช้ `SIMULATE ROLLS` กับเงินจริง** · ปุ่มนี้มีไว้ทดสอบระบบเท่านั้น เพราะความปลอดภัยจะย้ายไปขึ้นกับ CSPRNG ของระบบปฏิบัติการ ซึ่งคุณตรวจสอบด้วยตาไม่ได้
- **Passphrase คือความลับสูงสุด** · ผู้ใดได้รหัสนี้ไป (พร้อม Mnemonic Phrase) จะควบคุมสินทรัพย์ได้ทันที · เก็บบัตรสำรองไว้ในที่ปลอดภัยเทียบเท่าเงินสดหรือทองคำ
- **ทดสอบให้แน่ใจก่อนฝากเงินจริง** · ควรสร้างรหัสทดลองแล้วทดสอบกระบวนการ `Seed + Passphrase → Address` แบบ **ออฟไลน์** ให้ครบวงจรก่อนทุกครั้ง
- **หากปิด SALT** ผลการสุ่มเดิมกับการตั้งค่าเดิมจะได้ Passphrase ตัวเดิมเสมอ · **หากเปิด SALT** จะสร้างซ้ำไม่ได้ ต้องสำรองตัวรหัส Passphrase เอง
- ซอฟต์แวร์นี้ให้ **"ตามสภาพที่เป็นอยู่" (As-is) โดยไม่มีการรับประกันใด ๆ ทั้งสิ้น** · ผู้ใช้รับผิดชอบความปลอดภัยของกุญแจและสินทรัพย์ด้วยตนเอง

---

## 📄 License

Released under the **MIT License**. See [`LICENSE`](LICENSE).

---

<div align="center">

**© 2026 Chollatis Bitcoiner** · [learning.chontit.win](https://learning.chontit.win)
_Don't Trust, Verify._ · Powered by Claude AI

</div>
