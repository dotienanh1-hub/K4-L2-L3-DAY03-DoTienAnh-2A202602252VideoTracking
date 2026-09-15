# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `ĐỖ TIẾN ANH`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | `...` phút |
| Thời gian gán `clip_01` | `...` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `...` |

Ba tình huống khó nhất khi gán clip này và cách tôi xử lý:
1. **Xe rời khỏi khung hình:**  
   Có trường hợp bbox vẫn tiếp tục xuất hiện sau khi xe đã rời khung. Cần đặt trạng thái `outside` đúng tại frame xe thực sự ra khỏi khung. Trong kết quả kiểm tra có ID 8 ở frame 169–172 và ID 4 ở frame 149–151.

2. **Bounding box bị trôi giữa các keyframe:**  
   Tại các frame 101–103 của track 6 và frame 168 của track 8, bbox có IoU thấp hơn so với gold. Cách xử lý là thêm keyframe ở vùng chuyển động để bbox bám sát xe hơn.

3. **Đối tượng bị che khuất và nguy cơ mất ID:**  
   Khi xe bị che hoặc các xe ở gần nhau, việc duy trì cùng một `track_id` khó hơn. Vì vậy cần theo dõi liên tục ID của từng xe qua các frame và kiểm tra lại các đoạn có khả năng xảy ra tách track hoặc đổi ID.


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
| GOLD reference | `annotations/clip_01/gt.txt` |
| Số bbox trong GOLD | **573 bbox** |
| Số frame trong GOLD | **190 frame** |
| Số track trong GOLD | **8 track** |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `Chưa có trong notebook/evidence hiện có` |
| Thời điểm khóa pre-gold | `Chưa có trong notebook/evidence hiện có` |

Bản GOLD `annotations/clip_01/gt.txt` được dùng làm ground truth/reference để đánh giá annotation và các tracker. File GOLD có 573 dòng bbox, bao phủ clip từ frame 1 đến frame 190 và có 8 track. 

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | `Chưa có evidence` | `Chưa có evidence` | `Chưa có evidence` | `Chưa có evidence` | `Chưa có evidence` | `Chưa có evidence` | `Chưa có evidence` | `Chưa có evidence` | `Chưa có evidence` | `Chưa có evidence` |
| Sau rework / Bạn vs GOLD | **0.852** | **0.839** | **0.867** | **0.884** | **0.986** | **0.972** | **0.872** | **13** | **3** | **0** |


Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**
- **HOTA:** 0.852
- **DetA:** 0.839
- **AssA:** 0.867
- **LocA:** 0.884
- **IDF1:** 0.986
- **MOTA:** 0.972
- **MOTP:** 0.872
- **FP:** 13
- **FN:** 3
- **IDSW:** 0
Sau khi đọc danh sách lỗi, tôi đã sửa cụ thể những gì? Ghi theo frame và ID:
| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox treo sau khi xe rời khung | 169–172 | 8 | Đặt `outside` đúng frame xe rời khung |
| Bbox treo sau khi xe rời khung | 149–151 | 4 | Đặt `outside` đúng frame xe rời khung |
| Bbox trôi | 101–103 | 6 | Thêm keyframe quanh vùng bbox bị trôi |
| Bbox trôi | 168 | 8 | Thêm keyframe quanh vùng bbox bị trôi |


## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:
| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | **3.13.15 / 8.4.145 / 2.11.0+cpu / 0.5.13** |
| Weights | **yolo26n.pt** |
| Tracker control | **ByteTrack** |
| Tracker treatment | **BoT-SORT + ReID** |
| Conf | **0.25** |
| IoU | **0.70** |
| imgsz | **960** |
| Classes | **[2, 5, 7]** |
| Device | **CPU** |
| persist | **True** |
Hai lần chạy đều sử dụng cùng detector `yolo26n.pt`; sự khác biệt nằm ở tracker/association.

### So sánh kết quả
| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bạn vs gold | **0.852** | **0.839** | **0.867** | **0.884** | **0.986** | **0.972** | **0.872** | **13** | **3** | **0** |
| ByteTrack control vs gold | **0.709** | **0.649** | **0.776** | **0.846** | **0.875** | **0.749** | **0.823** | **88** | **54** | **2** |
| BoT-SORT + ReID vs gold | **0.763** | **0.711** | **0.820** | **0.872** | **0.900** | **0.792** | **0.860** | **91** | **26** | **2** |
| ReID vs bạn | **0.764** | **0.705** | **0.830** | **0.872** | **0.894** | **0.780** | **0.860** | **91** | **36** | **1** |

## 5. Phân tích — năm câu hỏi

### 1. MOTA của bạn cao hơn hay thấp hơn IDF1?
MOTA của nhãn tay là **0.972**, thấp hơn IDF1 **0.986**.
Với ByteTrack:
- MOTA = **0.749**
- IDF1 = **0.875**
Với ReID:
- MOTA = **0.792**
- IDF1 = **0.900**
IDF1 phản ánh khả năng duy trì và khớp đúng danh tính track rõ hơn, trong khi MOTA tập trung nhiều vào các lỗi detection như FP, FN và ID switch.
Vì vậy MOTA có thể vẫn tương đối cao ngay cả khi có vấn đề về identity association. Khi đánh giá tracking cần xem thêm **IDF1, AssA và IDSW**.

### 2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào?
BoT-SORT + ReID cho kết quả tốt hơn ByteTrack:
- **IDF1:** 0.875 → 0.900, tăng **0.025**
- **AssA:** 0.776 → 0.820, tăng **0.044**
- **IDSW:** 2 → 2, **không thay đổi**
Điều này cho thấy ReID treatment giúp association giữa các frame tốt hơn, đặc biệt ở khả năng duy trì identity.
Một đoạn đáng chú ý là **frame 87–113**. ReID có ID switch ở frame 87 và 113, trong khi ByteTrack có ID switch ở frame 59 và 94.
Tuy nhiên, đây là **system comparison**, không phải phép đo causal effect riêng của ReID, vì ByteTrack và BoT-SORT có implementation/association khác nhau.

### 3. DetA, FP và FN đổi thế nào?
So với ByteTrack:
- DetA: **0.649 → 0.711**
- FN: **54 → 26**
- FP: **88 → 91**
ReID cải thiện DetA và giảm đáng kể FN, nhưng FP tăng nhẹ.
Điều này cho thấy lỗi còn lại bao gồm cả:
- **Detection:** thể hiện qua FP, FN và bbox có IoU thấp.
- **Association:** thể hiện qua IDSW và hiện tượng một xe bị chia thành nhiều ID.
Với ByteTrack có **30 trường hợp bbox lệch** và **3 trường hợp model bắt thiếu đoạn**.
Với ReID còn **9 trường hợp bbox lệch** và **1 trường hợp bắt thiếu đoạn**.

### 4. Một chỗ bạn đúng và ReID sai
Một số bbox do ReID tạo ra nhưng không khớp với track trong gold. Ví dụ:
- **ID 7:** frame 16–116
- **ID 27:** frame 106–121
- **ID 38:** frame 158–178
Các trường hợp này không khớp track tham chiếu nào.
Đây là dấu hiệu model có thể tạo **bbox thừa / false positive**, trong khi annotation tham chiếu không có đối tượng tương ứng.
Vì vậy không nên mặc định model đúng chỉ vì nó tạo ra một track liên tục.

### 5. Một chỗ ReID làm bạn xem lại annotation

Các frame **104, 109, 110, 113, 114** của ReID có bbox lệch so với gold, với IoU khoảng **0.52–0.58**.
Đây là những frame cần xem lại annotation, đặc biệt ở các đoạn xe chuyển động hoặc bị che khuất.
Tuy nhiên, model không phải đáp án để sửa annotation. Nếu bbox model khác annotation, cần xem trực tiếp frame và diễn biến của track để quyết định.
Nếu model tạo bbox nhưng không có track gold tương ứng, evidence hiện tại nghiêng về khả năng model tạo **false positive** hơn là tự động kết luận annotation sai.

## 6. Nếu phải gán thêm 10 clip nữa
Nếu gán thêm 10 clip, tôi sẽ bổ sung các quy tắc sau vào `GUIDELINE_MINI.md`:
- Khi xe rời khỏi khung hình phải đặt `outside` đúng frame cuối cùng mà xe còn xuất hiện.
- Tăng số keyframe ở những đoạn chuyển động nhanh hoặc bbox có nguy cơ bị trôi.
- Kiểm tra kỹ các đoạn xe bị che khuất để tránh đổi `track_id`.
- Kiểm tra lại các đoạn hai xe đi gần hoặc cắt nhau.
- Sau khi gán xong cần tua lại video theo từng ID để phát hiện track bị tách hoặc bbox bị treo.

Quy trình nên ưu tiên:
**Gán nhãn thủ công → tự kiểm → đánh giá → xem kết quả model**
Không nên dùng model làm đáp án trong lúc đang annotate.


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
