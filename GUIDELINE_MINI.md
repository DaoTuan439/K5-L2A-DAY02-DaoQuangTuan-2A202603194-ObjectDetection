# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

Thứ tự ID trong nhãn nộp: `0 car, 1 van, 2 truck, 3 bus` .

| Lớp     | Định nghĩa của bạn                                                                          | Ví dụ/điểm phân biệt                                                                                     |
| ------- | ------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| `car`   | Xe chở người cỡ nhỏ: sedan, hatchback, SUV, taxi, xe bán tải có mui động cơ dài phía trước. | Có nắp capo rõ ràng, thân thấp hoặc gầm cao nhưng ngắn. SUV vẫn là `car`, không phải `van`.              |
| `truck` | Xe chở hàng: xe tải thùng, xe ben, xe đầu kéo, xe tải nhẹ đầu phẳng có thùng hở.            | Cabin tách khỏi thùng hàng; xe tải nhẹ có cabin nằm trên bánh trước và thùng hở phía sau vẫn là `truck`. |
| `bus`   | Xe khách cỡ lớn chở nhiều người: xe buýt thành phố, xe buýt nối toa, minibus chở khách.     | Thân dài, nhiều cửa sổ đều nhau, cửa lên xuống bên hông. Xe buýt nối toa là một vật thể.                 |
| `van`   | Xe van/minivan chở người hoặc chở hàng, thân kín liền một khối.                             | Đầu xe ngắn hoặc gần phẳng, thân cao và vuông hơn `car`, nhỏ hơn hẳn `bus`.                              |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: hình chữ nhật thẳng (không xoay, không polygon), bốn cạnh chạm điểm ngoài cùng của thân xe nhìn thấy được, kể cả bánh xe và gương; không lấy thêm mặt đường, bóng đổ hay xe bên cạnh.
- Vật thể bị che khuất: vẫn gán nếu phần nhìn thấy đủ để nhận ra lớp; box chỉ bao phần nhìn thấy, không đoán phần bị che. Hai xe chồng nhau thì vẽ hai box riêng.
- Vật thể bị cắt bởi rìa ảnh: box chạm đúng rìa ảnh, không kéo ra ngoài và không đoán phần nằm ngoài khung.
- Vật thể rất nhỏ/mơ hồ: chỉ gán khi còn nhận ra được loại xe; nếu không xác định chắc lớp thì không tự đoán, ghi lại và hỏi giảng viên.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh         | Tình huống                                                                            | Quyết định cuối                                                                                             | Lý do theo schema                                                                                   |
| ----------- | ------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| `drive_040` | Xe buýt nối toa nằm chéo khung hình; lần đầu vẽ bằng polygon xoay theo thân xe.       | Vẽ lại bằng **một** box chữ nhật thẳng bao toàn bộ hai toa, lớp `bus`; chấp nhận box chồng lên xe bên cạnh. | YOLO detection chỉ nhận box thẳng 5 số; xe nối toa là một vật thể nên chỉ một box.                  |
| `drive_050` | Hai xe minivan ở giữa ảnh, dễ nhầm với `car`; xe SUV nâu chồng lên xe trắng phía sau. | Hai minivan gán `van`; SUV nâu và xe trắng vẽ hai box riêng, cùng lớp `car`.                                | Minivan thân kín, cao, đầu ngắn thuộc `van`; mỗi xe một box kể cả khi chồng nhau.                   |
| `drive_018` | Nhiều xe con đang đỗ ven đường, ngoài làn chính.                                      | Gán tất cả là `car`.                                                                                        | README ghi "xe đang đỗ vẫn là `car`". Gold lại không gán các xe này — đã ghi lại để hỏi giảng viên. |

## Điểm cần giảng viên làm rõ

- **Thứ tự ID lớp:** README/`data.yaml` gốc ghi `0 car, 1 truck, 2 bus, 3 van`, nhưng gold train và nhãn eval dùng `0 car, 1 van, 2 truck, 3 bus` (ví dụ `drive_058` eval: xe buýt là `3`). Đã đổi ID nhãn và `data.yaml` theo gold; bản nhãn gốc còn ở commit `992da19`.
