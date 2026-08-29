# OCR Benchmark: พระราชบัญญัติการประกอบธุรกิจข้อมูลเครดิต (21 หน้า)

## ขอบเขต

ทดสอบ PDF ต้นฉบับจาก [ธนาคารแห่งประเทศไทย](https://www.bot.or.th/content/dam/bot/documents/th/laws-and-rules/laws-and-regulations/legal-department/7-ncb-act/7-1-ncb-act/7.1.2-Law_TH_CreditBureau%20Updated-2559.pdf) ชื่อ `Law_TH_CreditBureau_Updated-2559.pdf` จำนวน 21 หน้า A4. PDF นี้มี text layer ที่แตกอักขระไทยจากฟอนต์เก่า จึงใช้ภาพ render ต้นฉบับเป็นหลักในการตรวจความถูกต้อง ไม่ใช้ token score จาก text layer เป็นคะแนนความแม่นยำ.

ใช้หนึ่งภาพต่อหนึ่ง request, `temperature=0`, `max_tokens=8192` และตรวจเทียบ semantic จากภาพจริงครบ 21 หน้า. Qwen/OpenThai/Pathumma ใช้ภาพ `1555 x 2200 px`; Typhoon ใช้ `1272 x 1800 px` ตามคำแนะนำ fixed 1,800 px ใน [model card ของ Typhoon OCR 1.5](https://huggingface.co/typhoon-ai/typhoon-ocr1.5-2b).

| โมเดล | vLLM configuration | Prompt | ภาพ |
|---|---|---|---:|
| Qwen3.8-27B | BF16, MTP3, `max-num-seqs=10` | faithful document OCR | 2,200 px |
| OpenThai 2.0 | BF16, MTP3, `max-num-seqs=10` | faithful document OCR | 2,200 px |
| Pathumma Vision 3.0 preview | BF16, `max-num-seqs=4` | `document` (ทดลอง baseline เพิ่ม) | 2,200 px |
| Typhoon OCR 1.5 2B | `max-num-seqs=20` | fixed prompt ของผู้พัฒนาแบบไม่แก้ข้อความ | 1,800 px |

## เวลา, VRAM และความครบถ้วน

เวลาไม่รวม cold model load/warm-up. ค่า watermark คือจำนวนครั้งของ `สำนักงานคณะกรรมการกฤษฎีกา` ใน output ทั้งชุด; watermark ปรากฏจริงในภาพ แต่จำนวนที่สูงผิดปกติชี้ว่าระบบใช้ token กับพื้นหลังซ้ำจนตัดเนื้อหาหลัก.

| โมเดล | เวลา 21 หน้า | เวลา/หน้า | Throughput | VRAM ก่อน -> หลัง | `stop` / `length` | watermark ใน output |
|---|---:|---:|---:|---:|---:|---:|
| Qwen3.8-27B MTP3 | 57.57s | 2.74s | 222.91 token/s | 92.0 -> 92.0 GiB | 21 / 0 | 10 |
| OpenThai 2.0 MTP3 | 87.00s | 4.14s | 225.36 token/s | 88.1 -> 90.9 GiB | 21 / 0 | 11 |
| Pathumma seq4, document prompt | 122.19s | 5.82s | 680.25 token/s | 13.9 -> 13.9 GiB | 13 / 8 | 6,636 |
| Typhoon OCR 1.5, fixed prompt | 70.87s | 3.37s | 976.54 token/s | 35.7 -> 37.6 GiB | 15 / 6 | 4,519 |

Pathumma baseline `อ่านข้อความในภาพนี้` ทำได้แย่กว่า document prompt: `stop=12`, `length=9` และ watermark 7,511 ครั้ง จึงใช้ document prompt เป็นผลหลักของ Pathumma ในรายงานนี้.

## ตรวจความหมายจากภาพต้นฉบับ

นับเฉพาะ logical event ที่ทำให้ชื่อกฎหมาย บทบัญญัติ เงื่อนไข วันที่ ชื่อบุคคล หรือข้อความต่อเนื่องหายไปจนความหมายเปลี่ยน. ไม่นับ Markdown, spacing, การตัดบรรทัด, รูปแบบเลข หรือตัวอักษรผิดเล็กน้อยที่ยังอ่านความหมายเดิมได้. ช่วงข้อความที่ถูกตัดต่อเนื่องทั้ง block จะนับเป็นหนึ่งเหตุการณ์ ดังนั้นจำนวนนี้เป็น **confirmed lower bound** และห้ามใช้จัดอันดับโดยไม่ดูคอลัมน์ `length`.

| โมเดล | เหตุการณ์สาระเพี้ยนที่ยืนยันได้อย่างน้อย | หน้าที่มีผลกระทบ | ผลสรุป |
|---|---:|---|---|
| Qwen3.8-27B MTP3 | **>=114** | 1-21 | มีการแทนคำ/ข้อมูลเชิงกฎหมายเป็นคนละเรื่องอย่างเป็นระบบ; ใช้ไม่ได้ |
| OpenThai 2.0 MTP3 | **>=20** | 2, 4, 5, 8-11, 13-14, 16, 18-21 | เนื้อหาโดยรวมอ่านได้และไม่ตัดหน้า แต่เลขมาตรา/ชื่อ heading และข้อมูลท้ายเอกสารผิดหลายจุด |
| Pathumma seq4, document prompt | **>=5** | 1, 4, 5, 14, 18, 21 | ส่วนที่ไม่ถูกตัดรักษาเนื้อหาได้ดี แต่ 8 หน้า `length` และบางหน้าหายทั้งช่วงจาก watermark |
| Typhoon OCR 1.5, fixed prompt | **>=6** | 3, 8, 9, 15, 18, 20 | fixed prompt ยังถูก watermark กิน token; 4 หน้าขาด block ของบทบัญญัติ |

ตัวอย่างที่ชี้ขาด:

| หน้า | ต้นฉบับ | Qwen | OpenThai | Pathumma | Typhoon |
|---:|---|---|---|---|---|
| 1 | `พระราชบัญญัติการประกอบธุรกิจข้อมูลเครดิต พ.ศ. ๒๕๔๕` และวัน `๘ พฤศจิกายน ๒๕๔๕` | เป็น `พระราชนิพนธ์...ข้ามประเทศ พ.ศ. ๒๕๔๘`; นิยามและเงื่อนไขใช้บังคับผิด | ชื่อกฎหมาย ปี วัน และมาตรา 1-3 ถูกต้องเป็นหลัก | เก็บสาระหลักได้ แต่ข้อความ watermark มากและ output ชน limit | ชื่อ/ปีหลักถูก แต่ watermark จำนวนมากและหน้าจบด้วย `length` |
| 5 | มาตรา 9 และบทบัญญัติการจัดตั้ง/ใบอนุญาต | บทบัญญัติและเงื่อนไขเปลี่ยน | อ่าน `มาตรา 9` เป็น `มาตรา 8` | ตัดหลังหมวด 3; มาตรา 16-17 หาย | มี noise จาก watermark |
| 10 | `การอุทธรณ์ข้อโต้แย้ง`, มาตรา 28 และหมวด 5 | สาระมาตรา 28/บทบาทกรรมการสลับและเติมข้อมูล | `การอุทธรณ์ข้อโต้แย้ง` เป็น `การอนุมัติข้อเสนอแนะ`; label มาตรา 28/29 ไม่ครบ | มาตรา 28 และเนื้อหาหลักถูกเมื่อ output ไปถึง | มาตรา 28 และเนื้อหาหลักถูกเมื่อ output ไปถึง |
| 16 | บทกำหนดโทษ มาตรา 52-58 | เลขมาตราและเงื่อนไขทางอาญาผิด | heading 52-58 เป็น 22-28 | มีข้อความเกิน watermark | มีข้อความเกิน watermark |
| 21 | รายชื่อ/วันที่ผู้จัดทำ 5 รายการ | ชื่อและวันที่ผิด 5 รายการ | วัน `๒๐ ม.ค. ๒๕๕๙` เป็น `๒๕๕๘` และชื่อ `ปริญญาสิทธิ์` ผิด | output แทบเหลือ watermark/ข้อความไม่ครบ | ตารางอ่านได้ค่อนข้างดี แต่ชื่อ `อังคุมาลี` ผิด |

## ข้อสรุปเชิงใช้งาน

1. **ไม่ใช้ Qwen3.8-27B กับเอกสารกฎหมายชุดนี้** แม้เร็วที่สุด เพราะความผิดเป็นการเปลี่ยนชื่อกฎหมาย ปี เลขมาตรา เงื่อนไข และบทกำหนดโทษ ไม่ใช่เพียงคำสะกด.
2. **OpenThai เป็นผลลัพธ์ที่ใช้งานต่อได้มากที่สุดในรอบ raw-page นี้**: ทั้ง 21 หน้าไม่ถูกตัดและอ่านข้อความหลักได้ดี แต่ต้องทำ validation บังคับสำหรับเลขมาตรา หมวด ปี/วันที่ และชื่อบุคคลก่อนใช้เป็น draft กฎหมาย.
3. **Pathumma มีความถูกต้องเชิงความหมายดีในหน้าที่ output จบสมบูรณ์ แต่ยัง deploy ดิบไม่ได้**. ต้องมี preprocessing เพื่อลบ/ปิด watermark และ completeness gate ที่ retry ทุก record ซึ่งจบ `length`; หน้าที่ต้อง retry แน่นอนคือ 4, 5, 14 และ 21.
4. **Typhoon fixed prompt ทำตามข้อกำหนดผู้พัฒนาแล้ว แต่ยังไม่ปลอดภัยสำหรับ full legal OCR**. ต้อง retry อย่างน้อยหน้า 9, 15, 18 และ 20 เพราะ block ของกฎหมายหายจากการชน token limit.

## Rerun: suppress pale watermark, no Paddle layout

ทดสอบเพิ่มเฉพาะ Pathumma และ Typhoon โดย **ไม่ใช้ Paddle, layout detector, crop หรือ AI image editing**. ใช้ `benchmark/suppress_light_watermark.py` แปลงภาพเป็น grayscale และเปลี่ยนเฉพาะ pixel ที่มี luminance `>=220` เป็นขาว. เกณฑ์นี้เลือกจาก histogram ของภาพ: watermark สีเทาอ่อนมี peak ที่ 230 ขณะที่หมึกเอกสารเข้มกว่ามาก. ตรวจภาพ output แล้วตัวอักษรหลัก ตัวเลขไทย เส้น และเชิงอรรถยังอยู่.

### ตารางสรุปผลล่าสุด

ตารางนี้ใช้สำหรับเลือกโมเดลในเอกสาร BOT ชุดนี้: Pathumma และ Typhoon ใช้ผลหลัง `gray220`; Qwen และ OpenThai เป็นผล raw-page baseline เดิม. `Semantic errors` เป็น confirmed lower bound จากการตรวจภาพต้นฉบับครบ 21 หน้า ไม่ใช่ token mismatch.

| โมเดล | Input ที่ใช้ | เวลา/หน้า | ผล `length` | Semantic errors ที่ยืนยันได้ |
|---|---|---:|---:|---:|
| Qwen3.8-27B MTP3 | raw, 2,200 px | 2.74s | 0/21 | >=114 |
| OpenThai 2.0 MTP3 | raw, 2,200 px | 4.14s | 0/21 | >=20 |
| Pathumma | gray220, 2,200 px | 1.61s | 0/21 | >=2 |
| Typhoon fixed prompt | gray220, 1,800 px | 0.74s | 0/21 | 0 confirmed |

| โมเดล | ก่อน gray220: เวลา/หน้า | หลัง gray220: เวลา/หน้า | `length` ก่อน -> หลัง | watermark ก่อน -> หลัง | Semantic lower bound หลัง |
|---|---:|---:|---:|---:|---:|
| Pathumma seq4, document | 5.82s | 1.61s | 8/21 -> 0/21 | 6,636 -> 21 | **>=2** |
| Typhoon OCR 1.5, fixed prompt | 3.37s | 0.74s | 6/21 -> 0/21 | 4,519 -> 21 | **0 confirmed** |

`>=2` และ `0 confirmed` เป็นผล semantic review จากภาพต้นฉบับครบ 21 หน้า. หน้า 19 ระบุ `มาตรา ๖` ในภาพจริง จึงไม่ใช่ความผิดของทั้งสองโมเดลตามที่เคยระบุไว้ก่อนหน้านี้. รอบใหม่นี้ไม่มี output ที่จบ `length`; จึงเปรียบเทียบคุณภาพได้ตรงกว่าเดิม.

| โมเดล | ความผิดเชิงสาระที่ยังพบ | สิ่งที่แก้ได้จากรอบเดิม |
|---|---|---|
| Pathumma gray220 | หน้า 17: มาตรา 63 ข้ามฐานความผิด `มาตรา 57`; หน้า 20: `การออกบัตรเครดิต` เป็น `การออกแบบบัตรเครดิต` | หน้าที่เคยถูกตัด 4, 5, 14, 21 กลับมาครบ; หน้า 18 เก็บผู้รับสนองพระบรมราชโองการ/ยศ/ตำแหน่งครบ |
| Typhoon fixed gray220 | ไม่พบความผิดเชิงสาระที่ยืนยันได้ในการตรวจรอบนี้ | หน้าที่เคยตัด 9, 15, 18, 20 กลับมาครบ; ความผิดก่อนหน้าหน้า 3 และ 8 ไม่พบซ้ำ |

ทั้งสองโมเดลยังอาจติดเลขเชิงอรรถเป็น superscript ต่อท้ายเลขมาตรา เช่น `มาตรา ๒¹`, `มาตรา ๑๘⁹`, `มาตรา ๒๐/๑¹¹` และ `มาตรา ๓๑/๑¹⁴`. นี่สะท้อนเชิงอรรถที่มีในภาพ ไม่ใช่การเปลี่ยนเลขมาตรา แต่ downstream parser ควรแยก superscript ออกก่อนนำไป index.

**ข้อสรุปรอบแก้:** สำหรับ PDF นี้ให้ใช้ **Typhoon OCR 1.5 fixed prompt + gray220** เป็น candidate หลัก: เร็วกว่า Pathumma, จบครบทุกหน้า และไม่พบ semantic error ที่ยืนยันได้จากภาพในการตรวจรอบนี้. Pathumma gray220 เป็น fallback ที่ดี แต่ต้องแก้/ตรวจเพิ่มหน้า 17 และ 20. ก่อนปล่อยเป็นเอกสารกฎหมายฉบับใช้งานจริง ทั้งสองยังควรมี rule ตรวจเลขมาตรา ปี/วันที่ และ completeness gate รายหน้า.

### Audit ก่อนสร้าง Golden: เนื้อหากฎหมายหลัก

PDF ต้นฉบับมี text layer ภาษาไทยที่เสียหาย จึงนับจากการเทียบ output กับภาพ render ต้นฉบับทีละหน้า ไม่ใช้ `pdftotext` เป็น ground truth. Audit ชุดนี้นับคำหรือวลีเชิงกฎหมายหลักที่ต่างจากภาพจริง; ไม่นับ Markdown, การตัดบรรทัด, ช่องว่างในคำไทย, เลขหน้า และ marker ของเชิงอรรถ. ผล word-level ที่ครอบคลุม header/footnote ใช้ Golden dataset ในหัวข้อถัดไปเป็นผลหลัก.

| โมเดล | คำแทนผิด | คำหาย | คำเกิน | รวมที่ยืนยันจากภาพ | ผลกระทบเลขไทย |
|---|---:|---:|---:|---:|---|
| Pathumma gray220 | 2 | 1 | 1 | **4** | `มาตรา ๕๗` หาย 1 รายการ; ไม่พบเลขไทยที่ถูกแทนเป็นเลขอื่น |
| Typhoon fixed gray220 | 0 | 0 | 0 | **0** | ไม่พบในเนื้อหาหลัก |

รายการ Pathumma ที่ยืนยันได้: หน้า 2 `ผลกระทบต่อสิทธิเสรีภาพ` เป็น `ผลกระทบท่อสิทธิเสรีภาพ`; หน้า 17 วลี `มาตรา ๕๗` หายจากรายการมาตรา 63; หน้า 18 `การกระทำของกรรมการ` เป็น `การทำของกรรมการ`; หน้า 20 เติมคำ `แบบ` จนเป็น `การออกแบบบัตรเครดิต`. Typhoon รักษารายการ `มาตรา ๕๗` และวลี `การออกบัตรเครดิต` ได้ถูกต้อง.

ตัวเลข 4 และ 0 คือจำนวน **confirmed differences ของเนื้อหาหลัก** จาก audit ชุดแรก ไม่ใช่การรับรอง mathematical zero-error. มี utility ทำซ้ำได้ที่ `benchmark/score_thai_word_deltas.py` ซึ่งใช้ PyThaiNLP `newmm` และ page-aligned sequence alignment เพื่อหา candidate differences; ผลอัตโนมัติไม่ถูกนำมาปนกับตารางนี้ เพราะ reference seed ของ Typhoon ไม่เป็น independent transcript และ footnote/รูปแบบการจัดข้อความอาจทำให้จำนวน lexical delta สูงเกินจริง.

### Alignment diagnostics ก่อน Golden

ตารางนี้เพิ่ม Qwen3.8 และ OpenThai จาก JSON ที่รันไว้แล้ว โดยสร้าง lexical alignment กับ Typhoon gray220 ซึ่งเป็น visual-review seed จำนวน 6,839 token หลังตัดเลขหน้า, header `สำนักงานคณะกรรมการกฤษฎีกา`, Markdown/LaTeX และ superscript เชิงอรรถ. คอลัมน์ `แทน/หาย/เกิน` จึงเป็น **candidate delta** เพื่อเปรียบเทียบแนวโน้ม ไม่ใช่จำนวนคำผิดจาก ground-truth ที่ยืนยันแบบ mathematical exact.

| โมเดล | แทนคำ candidate | คำหาย candidate | คำเกิน candidate | รวม candidate | Word accuracy เทียบ seed | ตรวจภาพจริง |
|---|---:|---:|---:|---:|---:|---|
| Qwen3.8-27B MTP3 | 1,082 | 304 | 843 | 2,229 | 79.734% | `>=114` material lexical/semantic events; ไม่มี manual S/M/E ledger รายคำครบชุด |
| OpenThai 2.0 MTP3 | 191 | 203 | 48 | 442 | 94.239% | `>=20` material lexical/semantic events; ไม่มี manual S/M/E ledger รายคำครบชุด |
| Pathumma gray220 | 18 | 11 | 10 | 39 | 99.576% | 2 แทน / 1 หาย / 1 เกิน = **4 confirmed** |
| Typhoon fixed gray220 | 0 | 0 | 0 | 0 | 100.000% | **0 confirmed**; ไม่ใช่การพิสูจน์ zero-error |

ช่องว่างระหว่าง candidate กับผลตรวจภาพของ Pathumma เกิดจากเชิงอรรถ, เลขกำกับเชิงอรรถที่ติดกับเลขมาตรา/ปี, และการจัดบรรทัดที่แตกต่างกัน จึงใช้ผลตรวจภาพเป็นตัวตัดสินความถูกต้องจริง. สำหรับ Qwen/OpenThai ความต่างมีขนาดใหญ่และยืนยันจากภาพว่าเป็นทิศทางที่ถูกต้อง: Qwen มีการแทนคำ/เติมเนื้อหาผิดทั่วทั้งชุด ขณะที่ OpenThai ใกล้กว่าแต่ยังผิดเลขมาตรา, heading, ปี/วันที่ และชื่อบุคคล. `มาตรา ๖` บนหน้า 19 ถูกต้องในต้นฉบับ และไม่นับเป็นความผิดของโมเดลใด.

## Golden Dataset และการให้คะแนนรอบหลัก

Terra สร้าง [Golden transcript](bot_credit_bureau_21p_golden_transcript_20260829.json) จากภาพ `gray220` ต้นฉบับ 21 หน้า และตรวจภาพด้วยสายตาครบทุกหน้าในวันที่ 2026-08-29. JSON เก็บ text รายหน้า, สถานะการตรวจ, SHA-256 ของภาพต้นฉบับ, correction ledger และรายการ glyph ที่ไม่แน่ใจ. การตรวจ validation ยืนยันว่าเลขหน้า 1-21 เรียงครบ, สถานะ `completed_visual_review` ครบทุกหน้า, `uncertain_glyphs` ว่าง และ hash ภาพตรง 21/21. ดู protocol และข้อจำกัดใน [เอกสาร Golden dataset](BOT_CREDIT_BUREAU_GOLDEN_DATASET_2026-08-29.md).

ตารางนี้เป็นผลหลักสำหรับ word-level benchmark. ใช้ `benchmark/score_golden_ocr.py`, PyThaiNLP `newmm` และ page-aligned sequence alignment กับ Golden โดยตรง. Normalization ตัดเฉพาะ presentation syntax/เลขหน้า แต่เก็บ header, เนื้อหากฎหมาย, เชิงอรรถ, amendment reference และเลขไทยไว้.

| โมเดล | คำแทนผิด | คำหาย | คำเกิน | รวม lexical delta | Word accuracy | ความถูกต้อง token ตัวเลข |
|---|---:|---:|---:|---:|---:|---:|
| Qwen3.8-27B MTP3 | 1,095 | 306 | 831 | **2,232** | 79.592% | 65.385% |
| OpenThai 2.0 MTP3 | 197 | 225 | 56 | **478** | 93.853% | 55.769% |
| Pathumma gray220 | 9 | 11 | 5 | **25** | 99.709% | 97.802% |
| Pathumma raw | 592 | 127 | 6,089 | **6,808** | 89.527% | 80.220% |
| Typhoon fixed gray220 | 3 | 0 | 0 | **3** | 99.956% | 99.176% |
| Typhoon fixed raw | 957 | 162 | 3,647 | **4,766** | 83.700% | 74.725% |

Typhoon มี delta 3 รายการทั้งหมดเป็น **เลขอ้างอิงเชิงอรรถที่หาย** ไม่ใช่คำในเนื้อความหลัก: หน้า 18 ขาด `๑๙` หลัง `มาตรา ๖๔`; หน้า 19 ขาด `๒๐` และ `๒๑` หลังชื่อพระราชบัญญัติแก้ไขเพิ่มเติม. Sequence alignment แสดงเป็น `คำแทนผิด` เพราะ Golden เก็บเลขอ้างอิงติดกับ token ก่อนหน้า เช่น `๖๔๑๙` แต่การตรวจภาพตีความได้ว่าเป็น footnote-marker omission. ดังนั้น Typhoon ยังมี `0` ความผิดที่ยืนยันได้ในเนื้อความหลัก แต่ไม่ใช่ zero-error สำหรับข้อความที่มองเห็นทั้งหมด.

Pathumma มี delta จากเชิงอรรถ/เลขอ้างอิงเพิ่มจาก audit เนื้อหาหลักเดิม 4 จุด. Qwen และ OpenThai ถูกวัดกับ Golden เดียวกันแล้ว; ผลจึงใช้เปรียบเทียบ S/M/E ระหว่างทั้งสี่โมเดลได้โดยตรง. คะแนนยังเป็น lexical metric และควรใช้ควบคู่กับ semantic review สำหรับเอกสารกฎหมาย.

Pathumma raw และ Typhoon raw ใช้ prompt, รุ่นโมเดล, ลำดับหน้า และขนาดภาพเท่าเดิมกับรอบ gray220 แต่ส่งภาพก่อน suppress watermark โดยตรง. คะแนนลดลงเพราะ watermark สีอ่อนถูกอ่านซ้ำ: Pathumma มีคำเกิน 6,089 token และ Typhoon มีคำเกิน 3,647 token. จึงเป็นหลักฐานว่า `gray220` ช่วยที่การเตรียมภาพ ไม่ใช่การเปลี่ยน prompt หรือเกณฑ์ให้คะแนน.

## หลักฐานในเครื่อง

- PDF: `inputs/bot_credit_bureau_2559/Law_TH_CreditBureau_Updated-2559.pdf`
- ภาพ 2,200 px: `inputs/bot_credit_bureau_2559/pages_2200/`
- ภาพ 1,800 px: `inputs/bot_credit_bureau_2559/pages_1800/`
- Qwen: `results/qwen38_mtp3_seq10_bot_credit_bureau_21p_2200_20260828.json`
- OpenThai: `results/openthai2_mtp3_seq10_bot_credit_bureau_21p_2200_20260828.json`
- Pathumma document/baseline: `results/pathumma_seq4_document_bot_credit_bureau_21p_2200_20260828.json`, `results/pathumma_seq4_bot_credit_bureau_21p_2200_20260828.json`
- Typhoon fixed: `results/typhoon_fixed_seq20_bot_credit_bureau_21p_1800_20260828.json`
- Gray220 utility/images: `benchmark/suppress_light_watermark.py`, `inputs/bot_credit_bureau_2559/pages_2200_gray220/`, `inputs/bot_credit_bureau_2559/pages_1800_gray220/`
- Pathumma/Typhoon gray220: `results/pathumma_seq4_document_gray220_bot_credit_bureau_21p_2200_20260828.json`, `results/typhoon_fixed_seq20_gray220_bot_credit_bureau_21p_1800_20260828.json`
- Thai word-delta utility: `benchmark/score_thai_word_deltas.py`
- Golden transcript/score: `results/bot_credit_bureau_21p_golden_transcript_20260829.json`, `results/bot_credit_bureau_21p_golden_scores_20260829.json`
- หน้า Best Result แบบ 5 คอลัมน์: `dashboard/ncb_best_result_comparison.html`, `dashboard/data/bot_credit_bureau_best_result_view_20260829.json` (local server port `7012`)

ผล JSON ดิบมีเนื้อหาเอกสาร จึงไม่ควรเผยแพร่สาธารณะโดยไม่ตรวจข้อกำหนดข้อมูลก่อน.

## Resolution sweep ใหม่: raw เทียบ gray220, `max-num-seqs=7`

รอบนี้เป็นผลล่าสุดที่ใช้เปรียบเทียบโมเดลโดยตรงกับ Golden dataset เดียวกันจำนวน 21 หน้า สร้างภาพที่ความยาวด้านมากสุด `1024`, `1400`, `1600`, `1800`, `2000` และ `2200` px ทั้งแบบภาพต้นฉบับ (`raw`) และภาพ `gray220` รวม 48 งานย่อย หรือ 1,008 หน้า OCR. ทุกงานใช้หนึ่งภาพต่อหนึ่ง request, `temperature=0`, `max_tokens=8192`, client concurrency=7 และ vLLM `max-num-seqs=7`.

`gray220` คือ grayscale แล้วตั้ง pixel ที่ luminance ตั้งแต่ 220 เป็นสีขาว เพื่อระงับ watermark สีอ่อนเท่านั้น ไม่มี Paddle layout detection, crop, rotation หรือการสร้างภาพใหม่ในรอบนี้. จึงวัดผลของ preprocessing ได้ตรงกับภาพเต็มหน้า.

| โมเดล | recipe ที่แนะนำจาก sweep | Word accuracy | เลขไทย/ตัวเลข | แทนผิด / หาย / เกิน | เวลา/หน้า | VRAM peak |
|---|---|---:|---:|---:|---:|---:|
| Qwen3.8-27B | gray220, 2,200 px | 93.824% | 78.297% | 372 / 52 / 319 | 3.906s | 90.88 GiB |
| OpenThai 2.0 | raw, 2,200 px | 93.372% | 53.022% | 200 / 255 / 51 | 4.302s | 91.00 GiB |
| Pathumma Vision 3.0 | gray220, 1,600 px | 99.752% | 98.352% | 15 / 2 / 9 | 0.909s | 12.31 GiB |
| Typhoon OCR 1.5 | fixed prompt, gray220, 1,800 px | 99.927% | 98.901% | 4 / 1 / 0 | 1.108s | 36.06 GiB |

ผล Typhoon ที่ 2,200 px มี Word accuracy เท่ากับ 99.927% แต่เลขต่ำกว่าเล็กน้อย (98.626%) และช้ากว่า (1.325s/หน้า) จึงเลือก 1,800 px เป็นจุดสมดุลของชุดเอกสารนี้. Pathumma ที่ 1,600 px เป็นจุดสูงสุดของ Word accuracy ใน sweep; 1,800 และ 2,200 px ยังดีมากแต่ช้ากว่าเล็กน้อย.

### ผลของ raw และ gray220 ที่ 2,200 px

| โมเดล | raw: word / เลข / เวลา | gray220: word / เลข / เวลา | ข้อสังเกต |
|---|---|---|---|
| Qwen3.8-27B | 93.532% / 78.846% / 3.912s | 93.824% / 78.297% / 3.906s | ผลใกล้กันมาก; gray220 ไม่ได้เพิ่มความแม่นยำเลข |
| OpenThai 2.0 | 93.372% / 53.022% / 4.302s | 91.173% / 41.209% / 4.167s | raw เหมาะกว่าในเอกสารนี้ โดยเฉพาะตัวเลข |
| Pathumma Vision 3.0 | 86.861% / 76.923% / 4.507s | 99.665% / 96.978% / 1.126s | gray220 ลดการอ่าน watermark ซ้ำอย่างชัดเจน |
| Typhoon OCR 1.5 | 81.311% / 75.824% / 6.032s | 99.927% / 98.626% / 1.325s | gray220 เป็นเงื่อนไขสำคัญสำหรับเอกสารมี watermark ชุดนี้ |

### MTP และข้อจำกัดของรอบนี้

OpenThai 2.0 รันด้วย MTP 3 token และ log ของ vLLM ยืนยันการสร้าง MTP drafter/การยอมรับ draft token แล้ว. แต่รอบ sweep นี้ไม่มี control run แบบ MTP ปิดในเงื่อนไขเดียวกัน จึงยังระบุเป็นเปอร์เซ็นต์ว่า MTP ทำให้ end-to-end เร็วขึ้นเท่าใดไม่ได้. Qwen3.8-27B รอบสุดท้ายต้องปิด MTP เพราะ vLLM บน Blackwell/SM120 ล้มเหลวระหว่างจัดสรร MTP CUDA buffer; ผล Qwen ในตารางนี้จึงเป็น **MTP ปิด** และห้ามนำเวลาไปสรุปเป็นผล A/B ของ MTP.

### หลักฐานทำซ้ำได้

- Notebook ที่เก็บ output OCR เต็มทุกหน้าของแต่ละ run: `notebooks/qwen38_resolution_sweep_raw_gray220_20260829.ipynb`, `notebooks/openthai2_resolution_sweep_raw_gray220_20260829.ipynb`, `notebooks/pathumma_resolution_sweep_raw_gray220_20260829.ipynb`, `notebooks/typhoon_resolution_sweep_raw_gray220_20260829.ipynb`
- คะแนน raw/gray ทุกความละเอียดและ VRAM ก่อน/peak/หลัง: `dashboard/data/bot_credit_bureau_resolution_sweep_20260829.json`
- หน้าเปรียบเทียบ token รายหน้ากับ Golden และภาพที่ส่งเข้า inference: `dashboard/ncb_best_result_comparison.html`
- ทุกคะแนนใช้ page-aligned Thai token alignment; ระบบ normalize whitespace/newline แต่ไม่ตัดเลขไทย เนื้อหา หรือเชิงอรรถ.
