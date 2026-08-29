# NCB OCR Benchmark: เวอร์ชัน Runtime

## ขอบเขต

เอกสารนี้บันทึกเวอร์ชันจาก Python environment ที่ใช้ serve โมเดลในชุดทดสอบ
พระราชบัญญัติการประกอบธุรกิจข้อมูลเครดิต 21 หน้า เมื่อ 29 สิงหาคม 2026.
เป็น snapshot ของ environment ที่รันจริง ไม่ใช่ข้อกำหนดขั้นต่ำของโมเดล และไม่รวม
model weight, path ภายในเครื่อง, credential หรือข้อมูลนอกชุด NCB.

Qwen3.8-27B, Pathumma Vision 3.0 และ Typhoon OCR 1.5 ใช้ runtime vLLM ชุดเดียวกัน
แต่แต่ละโมเดลมี process server ของตนเอง. OpenThai 2.0 ใช้ venv BF16 แยกต่างหาก.

## เวอร์ชันที่ตรวจพบ

| โมเดล | Runtime | Python | vLLM | PyTorch | CUDA build | Transformers | FlashInfer | Tokenizers | Safetensors | Pillow | PyMuPDF |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Qwen3.8-27B | shared vLLM | 3.12.3 | 0.27.1 | 2.13.0+cu132 | 13.2 | 5.15.1 | 0.6.16.post3 | 0.22.2 | 0.8.0 | 12.3.0 | 1.28.2 |
| OpenThai 2.0 | isolated BF16 venv | 3.12.3 | 0.27.1 | 2.13.0+cu132 | 13.2 | 5.15.1 | 0.6.16.post3 | 0.22.2 | 0.8.0 | 12.3.0 | 1.28.2 |
| Pathumma Vision 3.0 | shared vLLM | 3.12.3 | 0.27.1 | 2.13.0+cu132 | 13.2 | 5.15.1 | 0.6.16.post3 | 0.22.2 | 0.8.0 | 12.3.0 | 1.28.2 |
| Typhoon OCR 1.5 | shared vLLM | 3.12.3 | 0.27.1 | 2.13.0+cu132 | 13.2 | 5.15.1 | 0.6.16.post3 | 0.22.2 | 0.8.0 | 12.3.0 | 1.28.2 |

## วิธีบันทึก

- อ่าน Python version จาก interpreter ของ runtime ด้วย `python --version`.
- อ่านแพ็กเกจจาก `python -m pip show` โดยเก็บเฉพาะชื่อและ version.
- `CUDA build` อ้างอิง suffix `+cu132` ของ PyTorch; ไม่ใช่การอ้างว่าเครื่องปลายทาง
  ต้องติดตั้ง CUDA Toolkit เวอร์ชันเดียวกัน.
- ตารางนี้ไม่รวมแพ็กเกจของเครื่องมือ render PDF, preprocessing `gray220` และ
  Golden-dataset scorer เพราะไม่ได้อยู่ใน venv ที่ serve โมเดล.

## การเชื่อมกับผล benchmark

การตั้งค่าทดสอบร่วมคือหนึ่งภาพต่อ request, `temperature=0`, client concurrency 7,
vLLM `max-num-seqs=7` และ `max_tokens=8192`. ข้อยกเว้น MTP ของแต่ละผลที่เลือกอยู่ใน
[ตาราง raw/gray220 ทุก px](BOT_CREDIT_BUREAU_RESOLUTION_WORD_DELTAS_2026-08-29.md):
Qwen ปิด MTP, OpenThai ใช้ MTP 3, Pathumma และ Typhoon ไม่ใช้ MTP.
