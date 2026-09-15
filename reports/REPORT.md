# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `Bùi Việt Nam`
Ngày: `2026-09-15`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT Track Mode, Rectangle label `vehicle` |
| Thời gian gán `clip_02` (warm-up) | 55 phút |
| Thời gian gán `clip_01` | 1h25' |
| Số track đã vẽ trong `clip_01` | 8 |
| Số keyframe trung bình mỗi track | |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Xe bị che hoặc chỉ còn một phần rất nhỏ. Tôi giữ nguyên ID nếu còn đủ bằng chứng nhận ra cùng một xe và thời gian che dưới 25 frame. Nếu chỉ thấy đốm sáng hoặc mảnh không xác định, tôi chưa gán bbox.
2. Hai xe chồng lên nhau hoặc cắt nhau. Tôi giữ hai ID riêng, mỗi xe một bbox ôm phần nhìn thấy được, đồng thời đặt keyframe dày hơn trước và sau đoạn cắt nhau.
3. Xe rời khung hình. Tôi kết thúc track bằng `outside` ở frame cuối cùng còn nhìn thấy xe, không kéo bbox nội suy vào các frame sau.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: Các ID không bị đổi trong quá trình theo dõi; kết quả đối chiếu cũng ghi nhận `IDSW = 0`.
- Lượt 2: Phát hiện cần kiểm tra lại điểm bắt đầu/kết thúc của một số track, đặc biệt track 5, 6, 4 và 8.
- Lượt 3: Phát hiện bbox lỏng ở các đoạn track 5, frame 82–86, và track 6, frame 101–106; cần đặt keyframe dày hơn và kiểm tra interpolation.

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`...`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `3b70d5e10475fb42df350c81ec330f6f49bb75a93683c4d0d87a4874de622ee2` |
| Thời điểm khóa | `2026-09-15T10:43:31.339830+00:00` |
| Số row / frame / track trước khi mở reference | `661 / 190 / 8` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.7310 | 0.7015 | 0.7652 | 0.8363 | 0.9238 | 0.8360 | 0.8157 | 91 | 3 | 0 |
| Sau rework | 0.7310 | 0.7015 | 0.7652 | 0.8363 | 0.9238 | 0.8360 | 0.8157 | 91 | 3 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có** theo file đánh giá hiện tại.

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox còn sau khi xe rời khung | 139–190 | 5 | Cần sửa trong CVAT bằng `outside` ở frame cuối còn thấy xe; log chưa xác nhận đã sửa |
| Bbox xuất hiện trước khi xe được xác định rõ | 83–100 | 6 | Cần dời frame bắt đầu về frame đầu tiên nhận ra rõ xe; log chưa xác nhận đã sửa |
| Bbox lỏng ở đoạn che/cắt nhau | 82–86 | 5 | Cần thêm keyframe và chỉnh bbox ôm phần nhìn thấy; log chưa xác nhận đã sửa |
| Bbox lỏng ở đoạn xe nhỏ/che khuất | 101–106 | 6 | Cần thêm keyframe và kiểm tra interpolation; log chưa xác nhận đã sửa |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cpu / 0.5.13` |
| weights / hai tracker | `yolo26n.pt / bytetrack.yaml / configs/trackers/botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `0.25 / 0.70 / 960 / [2, 5, 7]` |
| device / persist / clip | `cpu / true / 190 frame` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.7310 | 0.7015 | 0.7652 | 0.8363 | 0.9238 | 0.8360 | 0.8157 | 91 | 3 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.681 | 0.616 | 0.758 | 0.849 | 0.847 | 0.704 | 0.831 | 85 | 108 | 3 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA của tôi thấp hơn IDF1: MOTA = 0.8360 còn IDF1 = 0.9238. Điều này cho thấy identity nhìn chung khá nhất quán nhưng vẫn còn lỗi phát hiện và định vị, chủ yếu là 91 FP và 3 FN. MOTA gộp FP, FN và IDSW vào một công thức nên không đo trực tiếp độ đúng của toàn bộ chuỗi identity. Vì vậy một bản có ít ID switch nhưng nhiều bbox thừa vẫn có thể giữ IDF1 cao trong khi MOTA giảm.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

BoT-SORT + ReID cao hơn ByteTrack ở IDF1 (`0.900` so với `0.875`) và AssA (`0.820` so với `0.776`), tức giữ identity tốt hơn trong lần chạy này. IDSW của hai phương pháp đều là `2`, nên ReID không loại bỏ được toàn bộ lỗi đổi ID. ByteTrack đổi ID ở frame 59 và 94; ReID đổi ID ở frame 87 và 113. Ở các đoạn cần soi, nên xem chuỗi frame 87–113, đặc biệt quanh lúc xe bị che/cắt nhau. Đây là so sánh hệ thống: hai kết quả dùng hai implementation tracker khác nhau, nên chưa cô lập được causal effect riêng của ReID.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

So với gold, ByteTrack có DetA `0.649`, FP `88`, FN `54`; ReID có DetA `0.711`, FP `91`, FN `26`. ReID bắt được nhiều đoạn hơn, giảm FN `28` và tăng DetA `0.062`, nhưng FP tăng nhẹ `3`. Về association, cả hai đều có `IDSW = 2`; ReID có AssA `0.820` cao hơn ByteTrack `0.776`, nhưng vẫn còn lỗi tách track. Vì vậy lỗi còn lại là cả detector lẫn association: ByteTrack còn bỏ sót nhiều đoạn, còn cả hai tracker vẫn đổi/tách ID ở các đoạn khó. Với annotation thủ công, `IDSW = 0` nhưng FP `91` và FN `3`, nên ưu tiên chính là thời điểm bắt đầu/kết thúc track và bbox thừa.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Log đã có các điểm ReID và nhãn tay không đồng ý, nhưng chưa kèm ảnh frame nên chưa đủ bằng chứng để kết luận bên nào đúng. Các ứng viên cần kiểm tra trực quan là frame 105–106 (model có 2 bbox riêng, nhãn tay cũng có 2 bbox nhưng khác phạm vi), và frame 139 (model có 2 bbox trong khi nhãn tay có 2 bbox khác). Sau khi mở ảnh, ghi rõ ID model, ID nhãn tay và lý do hình học/che khuất; không kết luận chỉ từ metric.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

ReID và nhãn tay lệch nhiều ở frame 105, 106, 139; ngoài ra log xếp frame 91, 107–109 và 118 là các điểm bất đồng cao. ReID có 16 track trong khi nhãn tay có 8 track, đồng thời sinh các track không khớp gold như ID 7 (frame 16–116), ID 27 (106–121) và ID 38 (158–178). Đây là dấu hiệu cần xem lại model trước khi sửa annotation: các track dài nhưng không khớp reference có thể là false positive. Ngược lại, FN của ReID là 108 khi so với nhãn tay, nên cũng cần kiểm tra xem model đã bỏ sót xe thật ở những frame đó hay chưa.

## 6. Nếu phải gán thêm 10 clip nữa

Tôi sẽ giữ và làm rõ ba luật trong `GUIDELINE_MINI.md`: xe bị che dưới 25 frame thì giữ ID, xe đã ra khỏi khung thì khi quay lại mở track mới, và chỉ bắt đầu track khi đủ bằng chứng nhận ra xe bốn bánh. Tôi cũng sẽ ghi rõ frame bắt đầu/kết thúc của từng track ngay trong lúc gán, đặt keyframe dày ở đoạn che khuất hoặc giao cắt, rồi tua lại giữa mọi cặp keyframe xa nhau.

Về quy trình, tôi sẽ Save bằng nút trên CVAT và reload để kiểm tra trước khi chuyển việc; sau đó chạy validator, xem visualization, kiểm tra riêng frame đầu/cuối của từng track, rồi mới lock pre-gold. Sau khi có gold, tôi sẽ sửa trong CVAT và export lại thay vì chỉnh trực tiếp file MOT.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md`
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md`
- [x] `reports/REPORT.md`
