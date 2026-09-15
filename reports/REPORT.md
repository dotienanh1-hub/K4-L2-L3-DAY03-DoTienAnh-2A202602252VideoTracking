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
| Số track đã vẽ trong `clip_01` | `...` |
| Số keyframe trung bình mỗi track | `...` |

Ba tình huống khó nhất khi gán clip này và cách tôi xử lý:

1. `Khi có occlusion dài và nhiều xe đi sát nhau, tôi tăng số keyframe ở vùng giao cắt để xác định quỹ đạo của từng xe. Tôi kiểm tra trước và sau đoạn occlusion bằng hướng di chuyển, thời điểm xuất hiện và vị trí dự kiến, để tránh gắn nhầm ID.`
2. `Khi xe đổi hướng hoặc quay đầu trong vài frame, bbox bị lệch và IoU giảm rõ rệt. Tôi chốt thêm keyframe ở frame đầu và cuối của đoạn đổi hướng rồi đối chiếu với đường đi trước đó để giữ track không tách sai.`
3. `Khi xe ở mép khung hình hoặc bị che một phần, tôi dễ bỏ sót hoặc nhận nhầm với xe lân cận. Tôi xử lý bằng cách tua liên tiếp trước/sau, nhìn cả quỹ đạo và vị trí tương đối của xe, thay vì chỉ dựa trên một frame duy nhất.`

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `Kiểm tổng thể ID để phát hiện xe nào bị đổi ID, tách track hoặc tạo track mới không hợp lý.`
- Lượt 2: `Kiểm frame đầu và cuối của mỗi track để đảm bảo xe bắt đầu và kết thúc ở đúng vị trí, không bị kéo dài hoặc cắt ngắn sai.`
- Lượt 3: `Kiểm các frame giữa, nơi occlusion và overlap xảy ra nhiều nhất, để đảm bảo mỗi track giữ cùng ID trong suốt quãng đời.`

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.
Số lỗi tôi tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của tôi: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`Có những trường hợp hai người không đồng ý ở đoạn xe bị che hoặc đi sát nhau, đặc biệt khi bbox overlap trong vài frame. Chúng tôi sau đó căn cứ vào hướng chuyển động, thời điểm xe xuất hiện/rời khung và keyframe quanh frame giao cắt để thống nhất. Một quy tắc còn thiếu là cách xử lý rõ ràng khi xe bị che ngắn nhưng đã có hai track liền nhau, dễ dẫn đến đổi ID hoặc tách track.`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `...` |
| Thời điểm khóa | `...` |
| Số row / frame / track trước khi mở reference | `...` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | | | | | | | | | | |
| Sau rework | | | | | | | | | | |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có / chưa**

Sau khi đọc danh sách lỗi, tôi đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| ID switch | `...` | `...` | `Chuyển track về xe đúng và thêm keyframe ở các frame trước/sau để giữ continuity.` |
| Tách track | `...` | `...` | `Gộp các đoạn track bị chia thành một ID duy nhất, đảm bảo quỹ đạo trước và sau occlusion liên tiếp.` |
| Bbox trôi / thiếu keyframe | `...` | `...` | `Điều chỉnh bbox sát với phần xe thấy được và bổ sung keyframe quanh vị trí xe đổi hướng hoặc bị che.` |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `...` |
| weights / hai tracker | `...` |
| conf / IoU / imgsz / classes | `...` |
| device | `...` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | | | | | | | | | | |
| ByteTrack control vs gold | | | | | | | | | | |
| BoT-SORT + ReID vs gold | | | | | | | | | | |
| ReID vs bạn | | | | | | | | | | |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`MOTA có thể cao hơn IDF1 nếu nhiệm vụ chủ yếu phát hiện xe đúng nhưng lại sai ở mức duy trì identity trên nhiều frame. Điều này cho thấy nhãn của tôi có thể đúng ở mức detection nhưng sai ở mức association. MOTA không phạt nặng lỗi ID vì nó chủ yếu dựa trên FP/FN và số lần switch; còn IDF1 lại đánh giá toàn bộ quãng đời của track nên càng nhiều frame bị đổi ID thì điểm số càng giảm.`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`ByteTrack control và BoT-SORT + ReID treatment đều dùng cùng detector đầu vào, nhưng đây là hai implementation khác nhau. Vì vậy, nếu có thay đổi ở IDF1, AssA và IDSW, ta chỉ có thể nói treatment khác biệt trên clip này, chứ không thể khẳng định ReID là nguyên nhân duy nhất. Ở đoạn occlusion, khi hai xe đi sát nhau, control có thể nhầm ID hoặc tách track, trong khi treatment có thể giữ đúng hơn nhờ appearance cue. Tuy nhiên, sự khác biệt đó không chứng minh mối nhân quả độc lập của ReID vì cấu trúc tracker đã khác.`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`Nếu DetA giảm và FP/FN tăng, lỗi chủ yếu nằm ở detector vì bbox không được phát hiện đúng hoặc phát hiện nhầm. Nếu DetA tương đối ổn nhưng IDF1 và AssA thấp, lỗi chủ yếu nằm ở association: xe được phát hiện nhưng gắn nhầm ID hoặc bị tách track. Với clip này, các trường hợp lỗi phổ biến nhất là ở đoạn xe che nhau hoặc ở mép khung hình, trong đó mức độ mơ hồ về vị trí và hình dạng khiến association dễ sai hơn detection.`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`Một ví dụ điển hình là frame `...` với `ID ...`: ReID treatment gắn nhầm xe đó sang một ID khác vì vùng overlap và appearance rất gần nhau. Tuy nhiên, nếu xem quỹ đạo trước/sau, ta thấy xe đó đi theo hướng nhất quán, không có dấu hiệu đổi hướng hoặc nhảy track. Do đó, annotation của tôi hợp lý hơn ở đoạn này.`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`Một trường hợp khác là frame `...` với `ID ...`: ReID treatment giữ track tốt hơn sau một đoạn occlusion, trong khi nhãn của tôi ban đầu đã cắt track quá sớm. Khi xem lại dữ liệu trước/sau occlusion, tôi nhận ra cần bổ sung keyframe và sửa lại nhãn để giữ ID liên tục. Đây là bằng chứng cho thấy model có thể giúp xác định các đoạn annotation cần xem lại, dù không phải lúc nào cũng đúng.`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

`Nếu phải gán thêm 10 clip nữa, tôi sẽ làm rõ thêm quy tắc xử lý occlusion dài, overlap và xe đi sát nhau. Tôi cũng sẽ chuẩn hóa quy trình: xem toàn bộ clip trước, xác định track chính, chốt keyframe ở đầu/cuối của mỗi track, rồi mới chạy kiểm tra ID và kiểm chéo. Điều này sẽ giảm số lượng lỗi đổi ID và tách track ở giai đoạn cuối.`

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
