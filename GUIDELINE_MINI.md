# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Nguyễn Phúc Đại`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): `gán nhãn cả xe đỗ bên lề`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | `Duy trì tính liên tục của hành trình chuyển động (trajectory) đối với vật thể bị occlusion ngắn` |
| Xe bị che lâu hơn ngưỡng trên | Ngắt track cũ. Khi xe xuất hiện lại, gán ID mới. | Tránh gán sai identity (ID switch) khi vật thể khuất tầm nhìn quá lâu dẫn đến mất dấu. |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | Đảm bảo tính nhất quán theo quy chuẩn tracking tiêu chuẩn của bài tập |
| Hai xe cắt nhau / chồng lên nhau | Xe ở phía trước giữ nguyên bbox và ID. Xe bị che phía sau thu hẹp bbox ôm phần nhìn thấy được | Giữ đúng quy tắc chỉ annotate pixel/phần nhìn thấy thực tế (visible region).   |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: `...` |
| Xe đang đỗ, không di chuyển | Giữ nguyên bbox và ID từ frame bắt đầu đến frame kết thúc xuất hiện trong clip. |
| Keyframe đặt dày ở đâu | Đặt dày keyframe ở các đoạn xe chuyển hướng (rẽ/quay đầu), thay đổi tốc độ đột ngột, hoặc khi xe bắt đầu/kết thúc bị che khuất. |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: clip_01
- Tình huống: ô tô con đi song song xe bus
- Quyết định: gán nhãn khi mui xe phía trước lộ rõ
- Lý do: không thể gán nhãn chỉ dựa vào ánh đèn thông qua kính

### Ca 2
- Clip / frame / ID: clip_01
- Tình huống: xe van đi theo sau xe bus
- Quyết định: gán nhãn ngay khi có bộ phận lộ ra
- Lý do: đặc tính thân xe hình hộp chữ nhật

### Ca 3
- Clip / frame / ID: clip_02
- Tình huống: 2 học sinh chở nhau đi bằng xe điện
- Quyết định: không gán nhãn
- Lý do: không phải phương tiện 4 bánh

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Bổ sung rõ ngưỡng kích thước tối thiểu cho xe ở quá xa (tối thiểu $10 \times 10$ pixels) để tránh tình trạng một số thành viên gán nhãn cho các điểm ảnh mờ chưa rõ dạng xe
- Lập quy tắc thống nhất cho xe đang đỗ bên đường: bắt buộc phải tạo track dài liên tục chứ không gán từng frame đơn lẻ để tránh bị nhảy ID.
