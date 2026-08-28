<div align="center">

# ⬡ PASSPHRASE FORGE

### Entropy-to-Key Generator · เครื่องสร้างรหัสผ่านความปลอดภัยสูงจาก Entropy กายภาพ

**_Don't Trust, Verify._**

![Network](https://img.shields.io/badge/network-none%20(CSP%20enforced)-2ea44f)
![Dependencies](https://img.shields.io/badge/dependencies-zero-2ea44f)
![Build](https://img.shields.io/badge/build-single%20file%20HTML-blue)
![Runs on](https://img.shields.io/badge/runs%20on-Tails%20%2F%20Tor%20%2F%20file%3A%2F%2F-9146ff)
![License](https://img.shields.io/badge/license-MIT-lightgrey)

**🇬🇧 [English](#-english)   ·   🇹🇭 [ภาษาไทย](#-ภาษาไทย)**

</div>

---

## 🔐 Verify File Integrity · ตรวจสอบความสมบูรณ์ของไฟล์

**EN —** Before running the tool, verify that your copy of `passphrase-forge.html` is authentic and unmodified. Compute its SHA-256 hash and confirm it matches the value below (comparison is case-insensitive). If it does **not** match, do not trust the file.

**TH —** ก่อนใช้งาน ให้ตรวจว่าไฟล์ `passphrase-forge.html` ที่คุณมีเป็นของแท้ ไม่ถูกแก้ไข โดยคำนวณค่า SHA-256 แล้วเทียบกับค่าด้านล่าง (ไม่สนตัวพิมพ์เล็ก/ใหญ่) — ถ้า **ไม่ตรง** อย่าใช้ไฟล์นั้น

```
SHA-256 (passphrase-forge.html):
C2A4DCEB6B5BA84A81E7F1AA22C40D08006ADFC92D0CE3A8B69DA7127EA8B9F8
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
| **Output** | A–Z / a–z / 0–9 / editable symbols · length slider **12–50** |
| **Unambiguous mode** | Removes look-alikes `0 O o 1 l I` + confusing symbols for hand-copying |
| **System salt (optional)** | Mixes a CSPRNG value so output isn't deterministic from dice alone |
| **Backup card** | Printable, grouped passphrase + **CRC32** typo-check + timestamped filename |
| **Verify panel** | Shows `E`, `R`, `K^L`, rejection limit, and a per-character base-K map |
| **Extras** | Input masking · airgap indicator · dark/light theme · panic wipe |

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

`connect-src 'none'` blocks **all** `fetch` / XHR / WebSocket. There is no CDN, no web font, no analytics — the file cannot phone home. This CSP, **not** the on-screen airgap light, is the real security boundary.

### Usage — step by step

**Prerequisites:** a browser (Tor Browser or any modern browser). For real secrets, use an air-gapped machine (Tails OS recommended).

1. **Get offline.** Copy `passphrase-forge.html` to a USB / Tails persistent storage. Open it via `file://`. On Tor Browser, allow JavaScript for this local page. **Disconnect the network** (or use Tails offline mode). The header light should read **AIRGAP READY** (green).

2. **`[01] Output spec.** Tick the character classes you want (A–Z / a–z / 0–9 / symbols) and edit the symbol set if desired. Slide **Length** (12–50). Enable **Unambiguous** if the passphrase will be hand-written. The panel shows `K`, bits/char, target entropy, and a strength rating.

3. **`[02] Source.** Choose Coin, D6, D16, or Cards. The help line tells you how many rolls you need (with a small recommended buffer so rejection ≈ 0).

4. **`[03] Input.** Physically roll/flip/draw and type the results. The field accepts only valid characters for the chosen source:
   - **Coin:** `0 1` or `H T`
   - **D6:** `1`–`6`
   - **D16:** `0`–`9`, `A`–`F`
   - **Cards:** rank + suit, e.g. `AS KH 10D 7C QS` (suits `S H D C`; type them run-together like `askh10d7c` and spacing is added automatically; reshuffle and keep drawing after 52).
   Watch the **ENTROPY POOL** bar fill to **READY**.

5. **`[04] Derive.** Optionally toggle **✦ SALT** (system-entropy mixing). Press **GENERATE**.

6. **`[05] Output.** Copy the passphrase, or press **BACKUP CARD → PRINT** for a printable card containing the grouped passphrase, a **CRC32 checksum**, and (if salt is off) the recorded rolls. Open the **Verify** panel to audit `E`, `R`, `K^L`, and the character map by hand. When done, press **PANIC WIPE**.

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

**PASSPHRASE FORGE** คือเครื่องมือ HTML ไฟล์เดียว ทำงานออฟไลน์ 100% ที่แปลง **entropy กายภาพ** — การโยนเหรียญ ทอยลูกเต๋า หรือจั่วไพ่ — ให้เป็นรหัสผ่านความปลอดภัยสูง โดยตัวอักษรทุกตำแหน่งถูกสุ่มแบบ **uniform อย่างพิสูจน์ได้** ไม่มี **modulo bias**, ไม่ต้องเชื่อ **hash**, และ **ไม่มีการเชื่อมต่อเครือข่ายแม้แต่ครั้งเดียว**

ออกแบบมาเพื่อสร้างและสำรอง **ความลับมูลค่าสูง อายุยาว** เช่น **BIP39 passphrase** ("คำที่ 25") บนเครื่อง air-gapped อย่าง **Tails OS**

### ความสามารถ

| ฟีเจอร์ | รายละเอียด |
|---|---|
| **แหล่ง Entropy** | เหรียญ (1 bit) · D6 (2.585 bit) · D16 (4 bit) · ไพ่ 52 ใบ (~5.70 bit ลดลงเรื่อย ๆ) |
| **ผลลัพธ์** | A–Z / a–z / 0–9 / สัญลักษณ์ (แก้ได้) · แถบเลื่อนความยาว **12–50** |
| **โหมด Unambiguous** | ตัดอักขระหน้าตาคล้ายกัน `0 O o 1 l I` + สัญลักษณ์กำกวม เพื่อคัดลอกด้วยมือไม่ผิด |
| **System salt (เลือกได้)** | ผสมค่าจาก CSPRNG เพื่อให้ผลไม่ deterministic จากลูกเต๋าอย่างเดียว |
| **Backup card** | พิมพ์ได้ · passphrase จัดกลุ่ม + **CRC32** กันจดผิด + ชื่อไฟล์มีวันที่-เวลา |
| **Verify panel** | แสดง `E`, `R`, `K^L`, ขอบเขต rejection และตารางแปลงตัวอักษรฐาน K |
| **อื่น ๆ** | Input masking · ไฟสถานะ airgap · ธีมมืด/สว่าง · panic wipe |

### หลักการทำงาน

ความปลอดภัยอยู่ที่ **คณิตศาสตร์** และทุกค่ากลางถูกแสดงให้ตรวจสอบด้วยมือได้

**สัญลักษณ์:** `K` = ขนาดชุดตัวอักษร · `L` = ความยาว · `r` = จำนวนหน้าของอุปกรณ์ (ฐาน) · `d` = ค่าที่สุ่มได้แต่ละครั้ง

1. **รวบรวม (mixed-radix).** ผลสุ่มแต่ละครั้งกลายเป็นเลขหนึ่งหลักของจำนวนเต็มก้อนใหญ่:
   `E = E·r + d` และ `R = R·r` (คำนวณด้วย `BigInt`)
   - เหรียญ `r=2`, D6 `r=6`, D16 `r=16`
   - **ไพ่** ใช้การจั่วแบบไม่คืน (*Lehmer code*): ฐานลดลง 52 → 51 → 50 … และ `d` คืออันดับของไพ่ที่จั่วในกองที่เหลือ · ไพ่เต็มสำรับให้ `log2(52!) ≈ 225.58` bit
   - `E` คือ entropy ที่เก็บได้เป็นจำนวนเต็มใน `[0, R)` · `R` คือจำนวนความเป็นไปได้ทั้งหมด

2. **ตรวจปริมาณ (Gate).** จะสร้างรหัสไม่ได้จนกว่า `R ≥ K^L` — คือเก็บ entropy พอกับพื้นที่ผลลัพธ์ที่ต้องการ

3. **กำจัด modulo bias (rejection sampling).** ถ้าเอา `E mod K^L` ตรง ๆ ตัวอักษรบางตัวจะออกบ่อยกว่า (เพราะ `R` มักหารด้วย `K^L` ไม่ลงตัว) เครื่องจึงรับ `E` เฉพาะเมื่อ `E < ⌊R / K^L⌋ · K^L` ถ้าเกินก็ขอให้ทอยเพิ่ม · ค่า `U = E mod K^L` ที่ผ่านจึง **uniform เป๊ะ** ทั่วทั้ง `K^L`

4. **ถอดกลับ (base conversion).** เขียน `U` เป็นเลขฐาน `K` แต่ละหลักเลือกตัวอักษร 1 ตัว → เทียบเท่ากับการ **สุ่มแต่ละตำแหน่งอิสระแบบ uniform** จากชุด `K` สัญลักษณ์

5. **(เลือก) Salt.** สุ่มค่า `S` แบบ uniform ใน `[0, R)` ด้วย `crypto.getRandomValues` แล้ว `E' = (E + S) mod R` เพื่อ whiten input กายภาพ — ทำให้แม้ลูกเต๋าลำเอียงหรือถูกแอบดูก็เดาผลไม่ได้ · เป็น *defense-in-depth* **ไม่ได้ลด** ปริมาณ entropy กายภาพที่ต้องเก็บ

> **ไม่มีการ hash** — entropy ของผลลัพธ์เท่ากับ entropy กายภาพที่ป้อน (`L · log2(K)` bit) ไม่มีอะไรต้องเชื่อนอกจากเลขคณิตที่ตรวจซ้ำได้เอง · ความแข็งแรงจริง = `min(entropy ที่ป้อน, พื้นที่ผลลัพธ์)` จึงเป็นเหตุผลที่เครื่องบังคับ `R ≥ K^L`

**การรับประกันว่าไม่มีข้อมูลรั่วออก** — หน้าเว็บฝัง Content-Security-Policy แบบเข้มงวด:

```html
<meta http-equiv="Content-Security-Policy"
      content="default-src 'none'; connect-src 'none'; ...">
```

`connect-src 'none'` บล็อก `fetch` / XHR / WebSocket **ทั้งหมด** ไม่มี CDN, web font, analytics — ไฟล์ส่งข้อมูลออกไม่ได้เลย · **CSP นี้** คือเส้นแบ่งความปลอดภัยจริง ไม่ใช่ไฟสถานะบนจอ

### วิธีใช้งาน — ทีละขั้น

**เตรียมพร้อม:** เบราว์เซอร์ (Tor Browser หรือรุ่นใหม่ใดก็ได้) · สำหรับความลับจริง ให้ใช้เครื่อง air-gapped (แนะนำ Tails OS)

1. **ออฟไลน์ก่อน.** คัดลอก `passphrase-forge.html` ลง USB / Tails persistent → เปิดผ่าน `file://` (บน Tor Browser อนุญาต JavaScript หน้านี้) → **ตัดการเชื่อมต่อเครือข่าย** (หรือใช้ Tails offline mode) · ไฟบนหัวควรขึ้น **AIRGAP READY** (เขียว)

2. **`[01] Output spec.** ติ๊กชุดตัวอักษรที่ต้องการ (A–Z / a–z / 0–9 / สัญลักษณ์) แก้ชุดสัญลักษณ์ได้ · เลื่อน **Length** (12–50) · เปิด **Unambiguous** ถ้าจะเขียนด้วยมือ · แผงจะบอก `K`, bit/ตัว, target entropy และระดับความแข็งแรง

3. **`[02] Source.** เลือก Coin / D6 / D16 / Cards · บรรทัดช่วยเหลือจะบอกว่าต้องสุ่มกี่ครั้ง (พร้อม buffer เล็กน้อยให้ rejection ≈ 0)

4. **`[03] Input.** ทอย/โยน/จั่วจริง แล้วพิมพ์ผล · ช่องรับเฉพาะอักขระที่ตรงกับ source:
   - **Coin:** `0 1` หรือ `H T`
   - **D6:** `1`–`6`
   - **D16:** `0`–`9`, `A`–`F`
   - **Cards:** อันดับ+ดอก เช่น `AS KH 10D 7C QS` (ดอก `S H D C` · พิมพ์ติดกันได้ เช่น `askh10d7c` ระบบเว้นวรรคให้ · ครบ 52 ใบให้สับใหม่แล้วจั่วต่อ)
   ดูแถบ **ENTROPY POOL** จนขึ้น **READY**

5. **`[04] Derive.** จะเปิด **✦ SALT** (ผสม system entropy) หรือไม่ก็ได้ → กด **GENERATE**

6. **`[05] Output.** คัดลอก passphrase หรือกด **BACKUP CARD → PRINT** เพื่อพิมพ์บัตรที่มี passphrase จัดกลุ่ม + **CRC32** + ผลสุ่ม (ถ้าปิด salt) · เปิดแผง **Verify** เพื่อตรวจ `E`, `R`, `K^L` และตารางตัวอักษรด้วยมือ · เสร็จแล้วกด **PANIC WIPE**

**ขั้นตอนตรวจ backup:** หลังคัดลอก passphrase ด้วยมือ ให้พิมพ์กลับเข้าช่องใดก็ได้ แล้วเทียบ CRC32 กับบนบัตร — ถ้าตรง = คัดลอกถูก

### การตรวจสอบและ Audit

- **คณิตศาสตร์แกนผ่านการทดสอบ:** ยืนยัน uniform (χ²/df ≈ 1.0 จาก 200k ตัวอย่าง) · CRC32 ตรง known vector (`123456789` → `CBF43926`) · entropy ไพ่ = `log2(52!) ≈ 225.58` bit
- **Dual-AI cross-review:** ตรวจโดย LLM 2 ตัวอิสระ · ข้อที่ถูกต้อง (custom-symbol ถูกเขียนทับ, clipboard API ล้าสมัย, CSPRNG loop ไม่มี cap) แก้แล้ว · ข้อที่ผิด threat model (เช่นเสนอเปลี่ยน CRC32 เป็น SHA-256 ทั้งที่ใช้แค่กันพิมพ์ผิด) วิเคราะห์แล้วปฏิเสธพร้อมเหตุผล
- **ตรวจเองได้:** ไฟล์อ่านออก — อ่าน CSP, อ่าน `generate()`, และไล่ตรวจแผง Verify ด้วยมือ

### ข้อควรระวังและข้อจำกัดความรับผิด

- โปรเจกต์ส่วนตัว **ไม่ใช่** ผลิตภัณฑ์ที่ผ่าน audit มืออาชีพ · อ่านซอร์สก่อนไว้ใจกับมูลค่าจริง
- **ใช้ลูกเต๋า/เหรียญที่ยุติธรรม** · rejection sampling แก้ bias ตอน *แปลง* ไม่ได้แก้ bias ของ *แหล่งสุ่ม* — ลูกเต๋าลำเอียงอาจอ่อนกว่า CSPRNG ดี ๆ
- **passphrase คือความลับ** · ใครมีมัน (+ mnemonic ที่รู้) คุมเงินได้ · เก็บบัตรสำรองเหมือนเงินสด/ทองคำ
- **ทดสอบก่อนใส่เงินจริง** · สร้างตัวทิ้งแล้ว verify ครบวงจร `seed + passphrase → address` แบบ **offline** ก่อน
- **SALT ปิด** = ผลสุ่ม+ตั้งค่าเดิมสร้าง passphrase ซ้ำได้ · **SALT เปิด** = สร้างซ้ำไม่ได้ ให้สำรองตัว passphrase เอง
- ให้ **ตามสภาพ ไม่มีการรับประกันใด ๆ** · รับผิดชอบกุญแจและเงินของตนเอง

---

<div align="center">

**© 2026 Chollatis Bitcoiner** · [learning.chontit.win](https://learning.chontit.win)
_Don't Trust, Verify._ · Powered by Claude AI

</div>
