# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Dao Quang Tuan
- MSSV: 2A202603194
- Email: daotuan6325@gmail.com
- Lớp: K5-L2A
- Link repo bài làm: https://github.com/DaoTuan439/K5-L2A-DAY02-DaoQuangTuan-2A202603194-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/drive/1WQozlGw8W76sC9CVrqcyRMqp31k58hSy
  - Lần chạy Colab này dùng nhãn với thứ tự ID cũ (commit `992da19`). Sau khi đổi ID, Colab hết hạn mức GPU nên lần chạy cuối được thực hiện local trên CPU với cùng code và tham số của notebook (xem mục 5, mục 6).

## 2. Schema và mini guideline

Chi tiết ở [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md).

- Bốn lớp: `car`, `van`, `truck`, `bus`. SUV và bán tải có capo dài là `car`; xe tải nhẹ đầu phẳng có thùng hở là `truck`; minivan thân kín, đầu ngắn là `van`; xe buýt nối toa là một `bus`.
- Box là hình chữ nhật thẳng, ôm sát phần xe nhìn thấy; xe bị che/cắt khung vẫn gán nếu nhận ra lớp; hai xe chồng nhau vẽ hai box.
- Ba ca mơ hồ đã quyết định:
  - `drive_040`: xe buýt nối toa nằm chéo — vẽ lại từ polygon thành một box thẳng, lớp `bus`.
  - `drive_050`: hai minivan gán `van`; SUV nâu chồng lên xe trắng vẽ hai box `car` riêng.
  - `drive_018`: xe đỗ ven đường gán `car` theo README.
- Cần giảng viên làm rõ: (1) thứ tự ID lớp của gold khác README; (2) gold không gán xe đỗ ven đường trong khi README yêu cầu gán.

## 3. Annotation và self-QC

- Số file nhãn train: 50 file (`dataset/labels/train/drive_001.txt` … `drive_050.txt`), tổng 718 box: car 571, van 45, truck 22, bus 80.
- Kết quả checker và lỗi đã xử lý: `python tools/check_yolo_labels.py` → `ĐẠT cấu trúc: 50/50 file nhãn hợp lệ`. Trên Windows cần đặt `PYTHONIOENCODING=utf-8` để script in được tiếng Việt (lỗi hiển thị, không phải lỗi nhãn).
- Công cụ gán nhãn: CVAT (local), xuất định dạng Ultralytics YOLO Detection 1.0.
- Danh sách ảnh đã rà bằng mắt: `drive_018`, `drive_028`, `drive_040`, `drive_050` _(bổ sung thêm các ảnh đã tự rà để đủ 10)_.
- Lỗi phát hiện khi rà và cách sửa:
  - `drive_040`: xe buýt nối toa ban đầu vẽ bằng polygon xoay → vẽ lại bằng box chữ nhật thẳng; xe buýt xanh góc trên trái bị gán nhầm `car`.
  - `drive_050`: một box gộp SUV nâu và xe trắng phía sau → cần tách hai box.
  - `drive_018`, `drive_028`: box lỏng (box xe trắng góc dưới phải `drive_028` rộng hơn thân xe nhiều).
- Commit khóa nhãn ban đầu: `992da19`.

## 4. Gold QC và rework

| Chỉ số              | Trước sửa (`992da19`) | Sau sửa — đổi ID lớp (`643c52a`) |
| ------------------- | --------------------: | -------------------------------: |
| TP                  |                   348 |                              429 |
| FP                  |                   370 |                              289 |
| FN                  |                   305 |                              224 |
| Precision           |                0.4847 |                           0.5975 |
| Recall              |                0.5329 |                           0.6570 |
| F1                  |                0.5077 |                           0.6258 |
| mean_iou_of_matches |                0.7143 |                           0.7265 |
| passed              |                 false |                            false |

Kết quả hiện tại: [`outputs/annotation_evaluation.json`](../../outputs/annotation_evaluation.json).

**Rework đã làm — đổi thứ tự ID lớp:**

- Lỗi đã kiểm: khi so khớp box không xét lớp, phần lớn box sai lớp lệch theo một quy luật cố định: 46 box gold `van` bị chấm là `bus`, 18 box gold `bus` bị chấm là `truck`, 20 box gold `truck` bị chấm là `van`.
- Kiểm chứng trên ảnh: trong `drive_040` gold ghi xe buýt nối toa là `3`, xe tải ben đỏ là `2`; trong nhãn eval `drive_058`, sáu xe buýt bên trái là `3`, xe tải trắng bên phải là `2`. Như vậy gold và nhãn eval dùng `0 car, 1 van, 2 truck, 3 bus`, khác README/`data.yaml` gốc (`0 car, 1 truck, 2 bus, 3 van`).
- Quyết định: đổi ID 50 file nhãn (truck 1→2, bus 2→3, van 3→1) và tên lớp trong `data.yaml` cho khớp gold/eval. Không đổi vị trí box hay lớp của vật thể nào. Không sửa gold. Nhãn gốc giữ ở commit `992da19`.

**Phần chưa đạt và nguyên nhân đã kiểm:**

Sau khi đổi ID, F1 vẫn 0.63 < 0.80. Phân tích box chưa khớp:

- Khoảng 226 box thừa không có box gold nào gần, chủ yếu là `car` nhỏ. Rà bằng mắt `drive_018` và `drive_028` (overlay box của mình và gold): phần lớn là **xe đỗ ven đường/bãi đỗ** mà gold không gán, trong khi README ghi "xe đang đỗ vẫn là `car`". Đã ghi vào mini guideline để hỏi giảng viên.
- Khoảng 161 xe gold có mà mình bỏ sót, chủ yếu là `car` nhỏ ở xa; nhiều nhất ở `drive_041`, `drive_024`, `drive_026`, `drive_027`.
- 42 box lệch/lỏng (IoU 0.1–0.5 so với gold), ví dụ box xe trắng góc dưới phải `drive_028`.
- Lỗi lớp còn lại sau đổi ID: 10 xe gold `van` mình gán `car`, 7 xe gold `car` mình gán `van` — đúng ranh giới mơ hồ van/car ghi trong guideline.

Vòng rework tiếp theo (chưa thực hiện): chờ giảng viên trả lời về xe đỗ ven đường; thu box lỏng; bổ sung xe bị sót trên làn chính.

## 5. Kết quả YOLO

- Runtime: chạy local trên CPU (Windows, Anaconda), Ultralytics 8.4.120, Torch 2.13.0+cpu. Thời gian train 50 epoch khoảng 22 phút.
- Tham số thay đổi so với notebook:
  - `device='cpu'` thay vì GPU — Colab báo hết hạn mức GPU.
  - `batch=-1` (AutoBatch chỉ dùng cho GPU) nên Ultralytics tự dùng batch 16.
  - Thêm `workers=0` để tránh lỗi dataloader đa tiến trình trên Windows.
  - Các tham số còn lại giữ nguyên: `yolo26n.pt`, `epochs=50`, `imgsz=640`, `seed=20260909`.
- Đường dẫn best checkpoint trong phiên chạy: `Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt` (best ở epoch 42; file `.pt` không nộp).

| Chỉ số trên 10 ảnh eval | Giá trị |
| ----------------------- | ------: |
| Precision               |  0.3057 |
| Recall                  |  0.3042 |
| mAP50                   |  0.2900 |
| mAP50–95                |  0.1194 |

Nguồn: [`assets/model_eval_metrics.json`](assets/model_eval_metrics.json).

Theo từng lớp:

| Lớp   | Số box eval | Precision | Recall | mAP50 | mAP50–95 |
| ----- | ----------: | --------: | -----: | ----: | -------: |
| car   |          94 |     0.425 |  0.630 | 0.512 |    0.199 |
| van   |           5 |     0.100 |  0.200 | 0.041 |    0.013 |
| truck |           2 |     0.000 |  0.000 | 0.050 |    0.008 |
| bus   |          12 |     0.697 |  0.387 | 0.558 |    0.258 |

Ảnh và biểu đồ:

- Ảnh dự đoán 10 ảnh eval: [`assets/eval_predictions/visuals/`](assets/eval_predictions/visuals/)
- Log train theo epoch: [`assets/yolo26_runs/vehicle_annotation/results.csv`](assets/yolo26_runs/vehicle_annotation/results.csv)
- Confusion matrix: [`assets/yolo26_runs/vehicle_eval/confusion_matrix.png`](assets/yolo26_runs/vehicle_eval/confusion_matrix.png)
- PR curve: [`assets/yolo26_runs/vehicle_eval/BoxPR_curve.png`](assets/yolo26_runs/vehicle_eval/BoxPR_curve.png)

Lần chạy Colab trước với ID cũ cho precision 0.410, recall 0.230, mAP50 0.272, mAP50–95 0.202; kết quả đó nằm ở commit `643c52a` và không dùng làm kết quả chính vì ID lớp lệch với nhãn eval.

Trả lời bốn câu hỏi cuối notebook:

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   `truck` có recall 0 (0/2 box eval), sau đó là `van` (0.20). Ví dụ [`drive_058.jpg`](assets/eval_predictions/visuals/drive_058.jpg): chiếc xe tải trắng bên phải, cạnh taxi vàng, không có box dự đoán nào. Cùng ảnh, hàng sáu xe buýt bên trái chỉ có một chiếc được phát hiện (`bus 0.34`). Nguyên nhân chính là dữ liệu train mất cân bằng: chỉ 22 box `truck` và 45 box `van` so với 571 box `car`.

2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   mAP50–95 (0.119) thấp hơn nhiều so với mAP50 (0.290), tức model tìm được vị trí xe nhưng box chưa khớp sát ở ngưỡng IoU cao. Cần kiểm độ bám sát của box: box có lấn mặt đường/bóng đổ không, cạnh box với xe bị che hoặc cắt khung có nhất quán không. Dữ kiện từ nhãn train hỗ trợ: mean IoU so với gold chỉ 0.73 và có 42 box lỏng/lệch, nên model học theo box chưa sát.

3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Thứ tự ID lớp: README ghi `1 truck, 2 bus, 3 van` nhưng nhãn eval dùng `1 van, 2 truck, 3 bus`. Ở lần chạy Colab đầu, model học theo ID cũ nên dự đoán đúng xe buýt vẫn bị tính sai lớp; sau khi đổi ID, bus đạt mAP50 0.558.
   - Ranh giới `van`/`car`: trong gold QC có 10 xe gold `van` bị mình gán `car` và 7 trường hợp ngược lại; ở eval, confusion matrix có 1 xe `car` bị dự đoán là `van` và 1 là `truck`, `van` chỉ đạt recall 0.20.
   - Confusion matrix cho thấy lỗi lớn nhất là bỏ sót (cột `background`: 67 car, 5 van, 2 truck, 12 bus không được phát hiện ở ngưỡng mặc định) hơn là nhầm lớp, phù hợp với việc train chỉ có 50 ảnh và nhãn còn thiếu xe nhỏ ở xa.

4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   Nếu train trên ảnh eval, model đã thấy đáp án nên chỉ số đo trên chính các ảnh đó sẽ cao ảo, không phản ánh khả năng trên ảnh mới. Trong notebook, `data.yaml` đặt `val: images/eval`: 10 ảnh này được dùng để tính metrics sau mỗi epoch, chọn `best.pt` và tính precision/recall/mAP cuối cùng. Vì `best.pt` được chọn dựa trên chính tập này nên kết quả vẫn hơi lạc quan so với một tập test hoàn toàn độc lập.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm               | Lỗi quan sát                                                 | Cách đã thử                                                   | Kết quả/còn thiếu                                              |
| ---------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------- | -------------------------------------------------------------- |
| Gán nhãn CVAT                | Vẽ xe buýt nối toa bằng polygon xoay                         | Vẽ lại bằng công cụ Rectangle                                 | Đã sửa                                                         |
| Self-QC                      | `check_yolo_labels.py` lỗi `UnicodeEncodeError` trên Windows | Chạy với `PYTHONIOENCODING=utf-8`                             | Đạt 50/50                                                      |
| Gold QC (10/11)              | F1 0.51; lỗi lớp lệch theo quy luật cố định                  | So khớp không xét lớp, đối chiếu ảnh `drive_040`, `drive_058` | Phát hiện gold dùng thứ tự ID khác README; đổi ID, F1 lên 0.63 |
| Gold QC                      | Nhiều FP là xe đỗ ven đường                                  | Overlay box mình và gold trên `drive_018`, `drive_028`        | Gold không gán xe đỗ, mâu thuẫn README; chờ giảng viên         |
| Train lại sau đổi ID (10/11) | Colab "Cannot connect to GPU backend" do hết hạn mức         | Chạy local cùng code notebook                                 | Chạy được trên CPU                                             |
| Train local                  | `OMP: Error #15` (libiomp5md.dll nạp hai lần, Anaconda)      | Đặt `KMP_DUPLICATE_LIB_OK=TRUE`                               | Train xong 50 epoch                                            |

## 7. Tự đánh giá theo rubric

| Tiêu chí                          | Tự đánh giá                                                                                             |
| --------------------------------- | ------------------------------------------------------------------------------------------------------- |
| Hoàn thành và định dạng YOLO (10) | Đạt: 50 file, checker 50/50.                                                                            |
| Đầy đủ vật thể (20)               | Chưa tốt: recall so với gold 0.66, còn sót xe nhỏ ở xa.                                                 |
| Độ bám sát box (20)               | Trung bình: mean IoU 0.73 (đạt ngưỡng 0.70) nhưng còn 42 box lỏng/lệch.                                 |
| Phân lớp nhất quán (15)           | Sau đổi ID, lỗi lớp còn ít (chủ yếu van/car); không gán vật thể ngoài schema.                           |
| Mini guideline (10)               | Có định nghĩa lớp, quy tắc box, 3 ca mơ hồ có lý do.                                                    |
| Gold-set QC (15)                  | Chưa đạt F1 0.80; đã có một vòng rework (đổi ID) được giải thích; vòng sửa box chưa làm.                |
| Pipeline YOLO và phân tích (10)   | Đã chạy (Colab lần 1, local CPU lần 2), không dùng eval để train, có metrics, ảnh eval và nhận xét lỗi. |

## 8. Công cụ AI và tài liệu tham khảo

- Công cụ AI: Claude Code.
  - Dùng để: giải thích yêu cầu lab và định dạng YOLO; góp ý chọn lớp cho vài xe mơ hồ;chạy lại notebook trên máy local;

- Tài liệu: tài liệu trong repo bài (README, GUIDE, RULES, RUBRIC, SUBMISSION); tài liệu Ultralytics YOLO; CVAT. Không dùng nguồn nhãn ngoài repo.
