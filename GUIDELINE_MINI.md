# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` |Phương tiện cơ giới 4 bánh kích thước nhỏ đến trung bình, thiết kế chủ yếu chở người (từ 4 đến 9 chỗ ngồi cá nhân/gia đình) | Sedan, SUV, Crossover, Hatchback (như Vios, Mazda 3, VF 8, SantaFe). Phân biệt với van: phần đuôi xe vuốt liền khối chở khách, không dùng thùng kín chở hàng chuyên dụng. |
| `truck` |Phương tiện cơ giới chuyên dụng chở hàng hóa nặng, có cấu trúc tách rời rõ ràng giữa cabin lái phía trước và thùng/khoang chở hàng phía sau. |Xe tải thùng, xe ben, xe bồn, xe đầu kéo container. Phân biệt: có khoảng hở hoặc khớp nối cơ học giữa cabin và thùng hàng. |
| `bus` |Phương tiện giao thông công cộng cỡ lớn chuyên chở khối lượng hành khách lớn (thường từ 16 chỗ ngồi trở lên). |Xe buýt nội đô, xe khách liên tỉnh, xe đưa đón học sinh. Phân biệt: thân xe dài, nhiều hàng ghế và nhiều ô cửa sổ kính dọc thân. |
| `van` |Xe tải nhỏ đa dụng có khoang hàng liền khối với khoang lái (không tách rời cabin), thường là xe dạng hộp (box-shaped) để chở hàng nhẹ hoặc kết hợp chở người.|Xe chở hàng/chuyển phát nhanh (van giao hàng Shopee/Lalamove, xe van greenSM). Phân biệt: thân xe kín dạng khối hộp, không có cabin tách rời như truck, và không phải chở khách số lượng lớn như bus. |

## Quy tắc chung

- **Box bám sát vật thể nghĩa là:** Bounding box phải bao bọc vừa khít các điểm ảnh (pixel) ngoài cùng của vật thể (Tight Bounding Box). Không vẽ quá rộng chứa nhiều nền trống và không được cắt lẹm vào vật thể. Đối với vật thể bị che khuất, box chỉ bao quanh phần nhìn thấy được (hoặc theo ước lượng nếu schema dự án yêu cầu ước lượng toàn bộ vật thể).
- **Vật thể bị che khuất:** Gán thuộc tính `occluded`. Kích hoạt (True hoặc chọn mức độ phù hợp) khi vật thể bị che bởi một vật thể khác nằm phía trước nó (ví dụ: xe máy sau cây, người đi bộ sau biển báo).
- **Vật thể bị cắt bởi rìa ảnh:** Gán thuộc tính `truncated`. Kích hoạt khi một phần của vật thể nằm ngoài ranh giới (biên) của khung hình camera.
- **Vật thể rất nhỏ/mơ hồ:** Gán thuộc tính `difficult` hoặc `ignore`. Áp dụng khi vật thể có kích thước quá nhỏ (ví dụ: dưới 15x15 pixel), bị nhòe chuyển động (motion blur), hoặc trong điều kiện thiếu sáng nghiêm trọng không thể phân biệt chắc chắn bằng mắt thường.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| :--- | :--- | :--- | :--- |
| `drive_001` | Xe ô tô bị khuất sau xe tải, chỉ nhìn thấy góc đầu xe | Vẽ box phần nhìn thấy, đặt `occluded: true`, `truncated: false` | Vật thể bị che khuất bởi vật thể khác trong ảnh |
| `drive_002` | Người đi bộ đang đi ra khỏi khung hình, bị khuất một nửa người | Vẽ box phần còn trong ảnh, đặt `occluded: false`, `truncated: true` | Vật thể bị cắt bởi rìa ảnh |
| `drive_003` | Đốm sáng nhỏ ở xa vào ban đêm, không rõ là đèn xe hay đèn đường | Không vẽ box (hoặc vẽ gán nhãn `ignore` / `difficult: true`) | Vật thể quá nhỏ/mơ hồ không đủ điều kiện phân loại chính xác |

