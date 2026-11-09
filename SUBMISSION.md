# Nộp bài — Ngày 2 Object Detection

## Hình thức và tên repo

**Bài cá nhân.** Mỗi học viên tự tạo và nộp một repo bài làm.

Repo đề bài dùng chung Level 2: `K5-L2-Day02-ObjectDetection`.

Tên repo bài nộp:

```text
K5-L2A-DAY02-HoVaTen-MSSV-ObjectDetection
```

Nếu lớp dùng tên riêng, thay `L2` bằng `L2A` hoặc `L2B`. Ví dụ: `K5-L2B-DAY02-NguyenVanAn-2A20260000-ObjectDetection`. Viết tên không dấu, không khoảng trắng, phân cách bằng dấu `-`; ngày dùng `DAY02`.

## Cấu trúc bài nộp

```text
dataset/labels/train/                       50 file YOLO do học viên gán
GUIDELINE_MINI.md                           định nghĩa lớp và ít nhất 3 ca mơ hồ
outputs/annotation_evaluation.json          kết quả kiểm tra nhãn sau khi nhận gold
report/individual/MSSV_HoTen.md             báo cáo cá nhân đã đổi tên
report/individual/assets/eval_predictions/ 10 ảnh eval kèm box dự đoán (từ Colab)
report/individual/assets/yolo26_runs/      log train và ảnh chỉ số model (từ Colab)
notebooks/day2_annotation_to_yolo.ipynb     notebook của bài
```

Đổi `MSSV_HoTen.md` thành tên thật, ví dụ `20241234_NguyenVanAn.md`. Báo cáo ghi link Colab đã chạy, precision/recall/mAP của model và đường dẫn ảnh đánh giá trên 10 ảnh `eval`. Không chép số minh họa hoặc kết quả của người khác thành kết quả của mình.

Giữ các file script, ảnh và cấu hình gốc trong bản làm việc để chạy lại; không đổi tên ảnh hoặc ID lớp. Không thêm gold train, tài liệu giảng viên, token, mật khẩu hay checkpoint `.pt` vào repo bài nộp.

`outputs/` được bỏ qua mặc định để tránh commit toàn bộ log/checkpoint. Khi chốt bài, chỉ thêm JSON cần nộp:

```bash
git add dataset/labels/train GUIDELINE_MINI.md report/individual
git add -f outputs/annotation_evaluation.json
git status
```

Kết quả Colab: giải nén `day2_colab_outputs.zip` rồi đặt **cả 2 folder** `eval_predictions/` và `yolo26_runs/` vào `report/individual/assets/`, dẫn link tương đối từ báo cáo (xem mục dưới). Giữ link Colab chạy được và cấp quyền xem cho người chấm theo thông báo lớp; không công khai gold train hoặc dữ liệu nội bộ.

## Kết quả Colab: xử lý `day2_colab_outputs.zip`

Chạy xong cell cuối, notebook tạo **`outputs/day2_colab_outputs.zip`** (thấy trong file tree của Colab). Bên trong gồm:

```text
day2_colab_outputs/
  eval_predictions/
    visuals/                   10 ảnh eval kèm box dự đoán của model
  yolo26_runs/
    vehicle_annotation/        log train theo epoch, có file results.csv
    vehicle_eval/              các ảnh chỉ số đánh giá model (confusion matrix, PR curve, ...)
  model_eval_metrics.json      precision, recall, mAP50, mAP50–95 của best checkpoint
```

Cách xử lý: **nộp cả 2 folder đã giải nén** `eval_predictions/` và `yolo26_runs/` vào `report/individual/assets/`:

1. **Tải ZIP về và giải nén.** Ở Colab, tải `day2_colab_outputs.zip` về (hoặc giải nén trực tiếp trong file tree). Bên trong giải ra 2 folder:
   - `eval_predictions/visuals/` — 10 ảnh eval kèm box dự đoán của model.
   - `yolo26_runs/vehicle_annotation/` (có `results.csv`) và `yolo26_runs/vehicle_eval/` (ảnh chỉ số: confusion matrix, PR curve, ...).
2. **Đặt 2 folder vào `report/individual/assets/`** (giữ cấu trúc con như trong ZIP):
   ```text
   report/individual/assets/
     eval_predictions/
       visuals/...
     yolo26_runs/
       vehicle_annotation/
         results.csv
         weights/            (best.pt, last.pt — KHÔNG nộp, xem bước 3)
       vehicle_eval/
   ```
   Từ báo cá nhân, dẫn link tương đối tới các ảnh/chỉ số (ví dụ `assets/eval_predictions/visuals/drive_055.jpg`).
3. **Checkpoint `.pt` tự bị loại.** `.gitignore` đã chặn toàn bộ file `.pt`, nên khi bạn `git add report/individual`, `weights/best.pt` và `last.pt` **không** được commit — bạn không cần xóa tay. Kiểm tra nhanh trước khi push:
   ```bash
   git status -- report/individual/assets
   find report/individual/assets -name '*.pt'   # phải trả về rỗng sau khi add
   ```
   Nếu thấy `.pt` hiện trong `git status`, tức có file đã được track trước đó — báo giảng viên, không tự `git add -f`.
4. **Số liệu (bắt buộc):** copy 4 chỉ số từ `model_eval_metrics.json` (đã in ngay trong notebook) vào mục 5 của báo cáo.
5. **Commit:** `git add report/individual` (đã có trong lệnh chốt bài) sẽ đưa cả 2 folder assets (đã sạch `.pt`) lên GitHub. `.gitignore` bỏ qua `outputs/` nên ZIP gốc và log Colab không bị commit — chỉ nộp các ảnh/index đã giải nén ở `assets/`.

## Khóa nhãn và rework

Sau self-QC, commit và push bộ nhãn ban đầu theo mốc khóa của giảng viên. Chỉ nhận `gold_train_labels/` khi cả lớp đã khóa/nộp nhãn. Trước khi rework, giữ commit của bản ban đầu và lưu số liệu trước/sau trong báo cáo; không sửa gold hoặc viết lại lịch sử để thay bản đã khóa.

Không commit `gold_train_labels/` hoặc file ZIP chứa gold. Colab chỉ clone repo của bạn, nên repo là đầu dữ liệu duy nhất: không đưa gold train hay nội dung giảng viên vào repo.

## Nơi nộp và deadline

- Mỗi học viên tự nộp **link repo trên LMS bằng tài khoản cá nhân**.
- Deadline: **11:59 trưa ngày hôm sau buổi lab, UTC+7**. Ca ngày 11/11/2026 có hạn nộp **11:59 ngày 12/11/2026**.
- Gia hạn, nộp muộn và sửa bài sau deadline theo thông báo chính thức của Key Coach trên kênh lớp/LMS. Lab không tự đặt thêm mức trừ điểm ngoài rubric và thông báo đó.

## Kiểm tra trước khi nộp

- [ ] Repo bài làm đúng tên; báo cáo đã đổi thành `MSSV_HoTen.md`.
- [ ] Có 50 file nhãn YOLO, cùng basename với ảnh train; checker chạy đạt.
- [ ] Mini guideline có định nghĩa lớp, quy tắc box và ít nhất 3 ca mơ hồ có lý do.
- [ ] Có JSON đánh giá annotation; kết quả trước/sau rework được ghi rõ nếu có.
- [ ] Colab đã chạy; có link, metrics và cả 2 folder `assets/eval_predictions/` + `assets/yolo26_runs/` trong báo cáo.
- [ ] Nhãn train và JSON thực sự xuất hiện trên GitHub, không chỉ nằm trên máy.
- [ ] Không có gold train, tài liệu giảng viên, token, mật khẩu hoặc model `.pt` trong commit.
- [ ] Đã nộp link trên LMS trước deadline và người chấm xem được bài.
