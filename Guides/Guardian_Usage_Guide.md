# Hướng dẫn sử dụng Guardian Skills

## Kịch bản 1: Viết code mới (BE)

```
1. Nhận issue → Paste vào chat + Prompt 4 (Phân tích Issue)
   → AI trả: folder structure, flow, transaction boundary, test strategy

2. Code xong → Paste Handler vào chat + Prompt 1 (Governance Mode)
   → AI chạy FGO → trả report: score + violations

3. Fix violations → Tạo PR trên GitHub
   → PR template tự hiện → Tick checklist
   → CI chạy: build, test, coverage
   → Merge
```

---

## Kịch bản 2: Viết code mới (FE)

```
1. Nhận issue → Paste vào chat + Prompt 2 (Thiết kế UI trên Stitch)
   → AI trả: UX flow, component tree, screen designs

2. Code theo thiết kế → Paste vào chat + Prompt 3 (FE Strict Implementation)
   → AI code theo đúng design, tự check FFO

3. Review code → Paste vào chat + Prompt 1 (Governance Mode)
   → AI chạy FFO → trả report: score + violations

4. Fix → Tạo PR → Tick checklist → CI chạy → Merge
```

---

## Kịch bản 3: Review PR người khác

```
1. Mở PR → Copy danh sách files changed
2. Paste vào chat + Prompt 6 (PR Review Mode)
   → AI phân loại từng file → chạy skill tương ứng
   → Trả Final Dashboard: APPROVE / REQUEST CHANGES / BLOCK
```

---

## Kịch bản 4: Quick check 1 file

```
Paste code vào chat + nói:
"Chạy FFA-ERR cho file này"
hoặc
"Chạy FFE-SEC cho component này"
→ AI chạy đúng 1 skill, trả report nhanh
```

---

## Tham chiếu nhanh

| Prompt   | Dùng khi                | Orchestrator |
| -------- | ----------------------- | ------------ |
| Prompt 1 | Review code BE hoặc FE  | FGO / FFO    |
| Prompt 2 | Thiết kế UI trên Stitch | —            |
| Prompt 3 | Code FE theo design     | FFO          |
| Prompt 4 | Phân tích issue mới     | FGO          |
| Prompt 5 | Code BE theo yêu cầu    | FGO          |
| Prompt 6 | Review PR tổng thể      | FGO + FFO    |

Danh sách Prompts đầy đủ nằm trong file `prompt.md` ở root workspace.
