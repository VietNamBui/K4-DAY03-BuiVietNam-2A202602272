# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Bùi Việt Nam`
Clip: `clip_01`
---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): `...`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | khoảng mất quan sát ngắn vẫn có thể nối identity bằng vị trí, hướng chuyển động và đặc điểm hình dáng |
| Xe bị che lâu hơn ngưỡng trên | mở **track mới** khi xe hiện lại | sau hơn 2 giây không đủ bằng chứng chắc chắn để nối lại ID cũ; tránh gán nhầm identity |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | đã ra khỏi khung là track cũ kết thúc; không đoán rằng xe quay lại vẫn là cùng một track |
| Hai xe cắt nhau / chồng lên nhau | giữ ID riêng cho từng xe, mỗi xe một bbox; đặt keyframe dày trước, trong và sau lúc cắt nhau để không đổi ID | xe vẫn là hai đối tượng khác nhau dù bbox chồng lên nhau; theo dõi vị trí, hướng đi và đặc điểm hình dáng để tránh ID switch |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: **không đặt ngưỡng pixel cứng, nhưng phải nhìn ra được phần thân/đầu/đuôi có hình dạng liên tục của xe; chỉ thấy một đốm sáng hoặc mảnh không xác định thì chưa gán** |
| Xe đang đỗ, không di chuyển | vẫn giữ cùng ID và bbox ở mọi frame xe còn trong khung; đặt `outside` đúng frame xe rời khung | xe đứng yên vẫn là một track hợp lệ, không được bỏ vì không có chuyển động |
| Keyframe đặt dày ở đâu | đặt dày ở frame đầu/cuối track, trước và sau che khuất, lúc hai xe cắt nhau, xe rẽ/phanh, đổi hướng hoặc thay đổi kích thước nhanh; đoạn đi thẳng đều có thể đặt thưa hơn và luôn kiểm tra giữa hai keyframe | các điểm này làm interpolation dễ lệch hoặc dễ gây ID switch; đặt dày có chủ đích giúp bbox bám xe mà không tạo keyframe thừa |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `Clip01/49/VEHICLE 3`
- Tình huống: `Hiện thị một phần đèn và phần nhựa dài của đèn xe khách vô cùng nhỏ. Phải zoom to mới có thể thấy được. Không biết có nên cho một bounding box ở frame đó hay không?`
- Quyết định: Không`
- Lý do: `Phần hiện thị đó khá nhỏ, không phân biệt được rõ ràng nó có phải là của vehicle hay không`

### Ca 2
- Clip / frame / ID: `Clip01/58/VEHICLE 7`
- Tình huống: `Hiện thị một phần đóm sáng của đèn xe ô tô. Nhưng nó khá nhỏ nên không biết có nên cho một bounding box ở frame đó hay không?`
- Quyết định: `Không`
- Lý do: `Phần hiện thị là một đốm sáng, khó để phân biệt nó là ô tô hay một phương tiện, hay bất kỳ cái gì khác`

### Ca 3
- Clip / frame / ID: `Clip01/83/VEHICLE 6`
- Tình huống: `Một phần đuôi rất nhỏ của một phương tiện đang nhô ra từ xe khách. Nhưng nó khá nhỏ nên không biết có nên cho một bounding box ở frame đó hay không? `
- Quyết định: `Có`
- Lý do: `Dù phần đó nhỏ nhưng vẫn đủ để nhận ra đó là của một phương tiện khác xe máy, không phải xe hai bánh`

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- **Phải kết thúc track ngay khi xe rời khung:** kết quả đối chiếu gold cho thấy track 5 còn bbox từ frame 139 đến 190, track 4 còn bbox ở frame 149–151 và track 8 còn bbox ở frame 169–171. Khi xe không còn nhìn thấy, đặt `outside` tại frame cuối cùng còn thấy xe; không giữ bbox nội suy ở các frame sau.
- **Không bắt đầu track từ dấu hiệu quá mơ hồ:** track 6 có bbox từ frame 83 trong khi xe tham chiếu chỉ bắt đầu xuất hiện rõ từ frame 101. Chỉ bắt đầu khi nhận ra được xe bốn bánh, không bắt đầu từ một mảnh/đốm sáng không xác định. Vì vậy cần rà lại các frame 83–100 và áp dụng tiêu chí ở mục 3.
- **Ở đoạn che khuất hoặc hai xe cắt nhau, phải đặt keyframe dày và kiểm tra bbox:** các bbox của track 5 ở frame 82–86 và track 6 ở frame 101–106 có IoU thấp hơn mong muốn, dù ID vẫn đúng. Sau khi thêm keyframe, tua lại toàn bộ đoạn giữa hai keyframe để bbox ôm phần nhìn thấy được, không ôm thừa vùng nền hoặc xe bên cạnh.
