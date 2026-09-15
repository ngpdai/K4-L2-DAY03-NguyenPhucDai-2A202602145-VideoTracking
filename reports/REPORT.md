# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: Nguyễn Phúc Đại
Ngày: 15/09/2026

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: Local Docker / Web UI |
| Thời gian gán `clip_02` (warm-up) | 25 phút |
| Thời gian gán `clip_01` | 60 phút |
| Số track đã vẽ trong `clip_01` | 7 |
| Số keyframe trung bình mỗi track | 4.5 |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Xe ô tô đằng xa bị xe khác che khuất (Occlusion): Xe bị che khuất tầm nhìn khoảng vài frame trước khi lộ diện lại. Bật thuộc tính occluded = true
2. Ranh giới giữa frame cuối cùng xe còn xuất hiện và frame xe biến mất hoàn toàn dễ bị lấn sang làm thừa bbox. Đặt Bbox ôm sát phần xe còn thấy ở mép ảnh
3. Quãng nội suy (Interpolation) giữa các Keyframe xa bị lệch (Drift): Xe rẽ góc hoặc thay đổi tốc độ khiến bbox bị trượt khỏi thân xe ở các frame giữa. Tua lại các frame giữa (midpoint) và bổ sung thêm keyframe để cố định bbox ôm sát vật thể.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `...`
- Lượt 2: `...`
- Lượt 3: `...`

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`...`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | e3b0c44298fc1c149afbf4c8996fb92427a |
| Thời điểm khóa | 15:45:00 - 15/09/2026 |
| Số row / frame / track trước khi mở reference | 1250 rows / 190 frames / 7 tracks |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold |0.742 | 0.810| 0.685|0.850 | 0.765|0.820 |0.840 |45 |60 |3 |
| Sau rework |0.845 | 0.880|0.815 |0.875 |0.865 |0.890 | 0.860|18 |22 | 0|

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): có 

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
|ID Switch |000085 |ID 04 -> ID 06 |Dùng tính năng Merge (M) trong CVAT để ghép 2 track bị tách làm 1 ID duy nhất |
|Bbox trôi (Drift) |000045 |ID 02 |Nắn lại bbox cho khít phần nhìn thấy của thân xe và tạo thêm 1 keyframe. |
|Bbox treo (Missing Outside) |000170 | ID 05|Đặt thuộc tính outside (O) tại frame 000170 để ngắt track khi xe đi ra khỏi hình. |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | Python 3.10 / ultralytics 8.4.145 / torch 2.1.0 / lap 0.5.13 |
| weights / hai tracker | yolo26n.pt / bytetrack.yaml vs botsort-reid.yaml |
| conf / IoU / imgsz / classes | conf=0.25 / IoU=0.45 / imgsz=960 / classes=[2, 5, 7] (COCO vehicles) |
| device | cuda:0 |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold |0.845 |0.880 |0.815 |0.875 |0.865 |0.890 |0.860 |18 | 22|0 |
| ByteTrack control vs gold |0.695 |0.750 | 0.648|0.810 |0.710 |0.740 |0.805 | 65| 85| 8|
| BoT-SORT + ReID vs gold |0.738 |0.755 |0.725 |0.815 |0.792 |0.755 |0.810 |60 |82 | 3|
| ReID vs bạn |0.752 |0.780 |0.730 |0.825 |0.801 |0.768 |0.820 | 52|70 |2 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

Trong kết quả gán nhãn, MOTA cao hơn IDF1 (Bản pre-gold: MOTA 0.820 vs IDF1 0.765).  Nếu MOTA cao mà IDF1 thấp, điều đó chứng tỏ mô hình/người gán nhãn tìm đối tượng rất tốt (Detection tốt) nhưng bị lỗi nhầm lẫn/thay đổi danh tính ID (Identity Switch/Fragmentation).  MOTA không phạt nặng lỗi ID vì công thức MOTA chỉ đếm số lần phát sinh ID Switch đúng 1 lần tại thời điểm xảy ra nhảy ID. Trong khi đó, IDF1 đánh giá sự nhất quán trên toàn bộ quãng đời (trajectory) của vật thể; khi 1 track bị cắt đôi, IDF1 sẽ phạt tới nửa quãng đời còn lại của track đó.  

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

So sánh: BoT-SORT + ReID cho kết quả vượt trội hơn ByteTrack control về khả năng duy trì identity: IDF1 tăng (0.792 vs 0.710), AssA tăng (0.725 vs 0.648) và IDSW giảm đáng kể (3 lỗi vs 8 lỗi).  Frame sequence minh họa: Tại chuỗi frame 000060 -> 000075, chiếc xe ID 02 đi qua sau cây cối và bị che hoàn toàn trong 15 frames. ByteTrack (chỉ dùng motion/IoU) bị mất dấu chuyển động nên tạo ra ID mới (IDSW) khi xe xuất hiện lại. Trong khi đó, BoT-SORT + ReID trích xuất đặc trưng hình ảnh (appearance feature) trước khi bị che nên đã nhận diện chính xác xe cũ và duy trì nguyên ID 02.  Lưu ý quan trọng: Sự chênh lệch này không cô lập hoàn toàn hiệu ứng nguyên nhân - kết quả (causal effect) của riêng ReID, vì ByteTrack và BoT-SORT có sự khác biệt trong cả kiến trúc triển khai thuật toán và cơ chế kết hợp thông tin (tracker implementation).  

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

DetA giữa ByteTrack (0.750) và ReID (0.755) hầu như không thay đổi đáng kể, lượng FP và FN giữa 2 phiên bản tương đương nhau (FP: 65 vs 60; FN: 85 vs 82).  Kết luận: Do cả 2 tracker đều dùng chung một bộ phát hiện vật thể (Detector input yolo26n.pt), phần lớn các lỗi FP/FN còn lại là do Detector bỏ sót xe mờ/nhỏ ở xa hoặc nhận diện nhầm nền. Sự cải thiện của ReID chủ yếu nằm ở phần Association (liên kết ID).  

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Frame / ID: Frame 000110 / ID 05.Hiện tượng: Chiếc xe bán tải bị xe khách vượt mặt che khuất 90% thân xe.  Vì sao bạn đúng: Người gán nhãn quan sát được 10% phần đuôi xe nhô ra ở mép xe khách và duy trì bbox khít cho ID 05 (bật occluded = true).  Vì sao ReID sai: Mức độ che khuất quá lớn khiến đặc trưng ReID thu được bị nhiễu bởi xe khách ở phía trước, dẫn đến việc ReID bỏ sót đối tượng (tăng FN) hoặc cắt ngắt track.  

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**
Frame / ID: Frame 000145 / ID 07.Vì sao: ReID tạo ra một track độc lập ở khu vực lề đường bên phải. Khi kiểm tra lại, người gán nhãn phát hiện có 1 chiếc xe ô tô con màu đen đỗ cố định trong bóng râm bị bỏ sót lúc gán nhãn thủ công.  Kết quả: ReID đã hỗ trợ phát hiện lỗi bỏ sót (FN) của người gán nhãn, giúp bổ sung thêm track cho chiếc xe đang đỗ này.  

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

Nếu phải thực hiện gán nhãn cho 10 clip tiếp theo, tôi sẽ điều chỉnh:Sửa trong GUIDELINE_MINI.md:  Quy định rõ ràng ngưỡng kích thước nhỏ nhất để gán nhãn.  Thêm điều khoản chi tiết về xử lý xe đỗ cố định bên đường và các ca xe bị che bởi chướng ngại vật cố định (cây cối, cột đèn).  Đổi trong quy trình làm việc:  Thực hiện thao tác gán nhãn dứt điểm từng đối tượng (track by track) từ đầu đến cuối clip thay vì gán theo từng frame.  Áp dụng nghiêm ngặt quy trình tự kiểm 3 lượt trước khi thực hiện export dữ liệu. 

## 7. Tệp đã nộp

- [ ] `annotations/clip_01/gt.txt`
- [ ] `annotations/clip_02/gt.txt`
- [ ] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [ ] `GUIDELINE_MINI.md` đã điền
- [ ] `outputs/eval_vs_gold.json`
- [ ] `outputs/model_bytetrack_clip_01.txt`
- [ ] `outputs/model_reid_clip_01.txt`
- [ ] `outputs/model_run_config.json`
- [ ] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [ ] `reports/REPORT.md` (file này)
