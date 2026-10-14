
Họ tên: Lê Trọng Khánh
`dataset_version`: d10-21e4070301

## Bảng tổng hợp (số của bạn, lấy từ `report/evidence/*.json`)

| Mã | Setup | Tham số huấn luyện | VRAM đỉnh (GB) | Phút | F1 | CI95 | FP/ảnh | F1 pothole | F1 crack | F1 manhole |
|---|---|---|---|---|---|---|---|---|---|---|
| E0 | zero-shot | 0 | | | | | | | | |
| E1 | LoRA LLM | 33030144 | 6.72 | 47.86 |0.27 |[0.2362204724409449, 0.30798479087452474] |1.2668810289389068 |0.22 |0.1941176470588235 | 0.5199999999999999|
| E2 | + projector | | | | | | | | | |
| E3 | + LoRA ViT | | | | | | | | | |
| E4 | (ghi biến và giá trị) | | | | | | | | | |

## 10 câu hỏi (trả lời bằng **số của bạn**, không chép lại tài liệu)

### Câu 1 — Kiến trúc: ViT, projector và LLM của Qwen3-VL-4B có bao nhiêu tham số mỗi khối, và E1/E2/E3 mỗi cái cập nhật khối nào?
Ở E1, tôi đóng băng toàn bộ trọng số gốc và chỉ cập nhật adapter
LoRA trên LLM. Regex chọn được 252 module: 36 lớp, mỗi lớp gồm
4 projection attention và 3 projection MLP. Cấu hình LoRA là
r=16, alpha=32, dropout=0.05. Tổng số tham số được huấn luyện
là 33.030.144. ViT và projector đều được đóng băng.

### Câu 2 — E0: `parse_valid_rate` bằng bao nhiêu, và hai lỗi phổ biến nhất của model gốc là gì (kèm một ví dụ output thô)?
<điền>

### Câu 3 — Thang 0–1000: nếu chấm output theo pixel (hoặc ngược lại) thì F1 ra sao và vì sao? Bạn đã kiểm chứng điều này thế nào?
<điền>

### Câu 4 — Mask loss: vì sao chỉ tính loss trên phần trả lời? Điều gì xảy ra nếu tính trên cả prompt và token ảnh?
Tôi mask token prompt, token ảnh và padding bằng -100 để loss
chỉ tính trên câu trả lời JSON và token kết thúc. Với mẫu kiểm tra,
ảnh có 220 token; phần labels khác -100 giải mã ra đúng JSON
đáp án và <|im_end|>. Batch hai ảnh có input_ids, attention_mask
và labels cùng kích thước (2, 581).

Nếu tính loss trên cả prompt và token ảnh, mục tiêu huấn luyện
sẽ bao gồm việc dự đoán lại phần đầu vào thay vì tập trung vào
câu trả lời định vị. Tôi chưa chạy thí nghiệm bỏ mask để đo
mức ảnh hưởng bằng số.

### Câu 5 — Vì sao không thể huấn luyện đầy đủ projector nếu ViT bị lượng tử hoá 4-bit, và bạn đã xử lý thế nào? Vì sao projector phải ở float32?
<điền>

### Câu 6 — Split theo phiên quay: vì sao chia ngẫu nhiên sẽ cho F1 cao giả? Số ảnh và số box từng lớp ở mỗi split là bao nhiêu?
Dữ liệu là frame cắt từ video, nên các frame gần nhau có thể
rất giống nhau. Chia ngẫu nhiên có nguy cơ đưa các frame cùng
phiên quay vào cả train và test, khiến F1 đánh giá cao giả tạo.
Tôi kiểm tra giao của các tập session và không thấy phiên nào
xuất hiện ở hai split.
| Split | Ảnh | Box pothole | Box crack | Box manhole |
|---|---:|---:|---:|---:|
| train | 1422 | 1074 | 1750 | 691 |
| val | 276 | 66 | 470 | 103 |
| test | 311 | 121 | 299 | 163 |

### Câu 7 — So sánh E1/E2/E3: mở projector và LoRA trên ViT có đáng không (F1, CI95, VRAM, thời gian)?
<điền>

### Câu 8 — Tập test có 311 ảnh: khoảng tin cậy rộng thế nào, và điều đó giới hạn kết luận so sánh các setup ra sao?
<điền>

### Câu 9 — Phân tích lỗi theo lớp: lớp nào kém nhất và vì sao (kích thước box, số mẫu, ngữ nghĩa)? Kèm ảnh minh hoạ.
<điền>

### Câu 10 — E4: bạn đổi biến nào, kết quả thế nào, và kết luận có đáng tin không?
<điền>

## Nhìn lại

Khi kiểm tra E1 trên 5 ảnh val với max_new_tokens=256,
một output bị cắt giữa JSON và chỉ 4/5 output hợp lệ.
Sau khi tăng giới hạn lên 1024, cả 5 output đều hợp lệ.
Tôi chọn giới hạn 1024 dựa trên val và áp dụng cùng giới hạn
cho các setup được so sánh. Kết quả E0 ban đầu ở 256 token
được giữ riêng.
