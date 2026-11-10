# GUIDELINE_MINI

# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.
> 

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Phương tiện giao thông cơ giới bốn bánh chủ yếu phục vụ chở người cá nhân/gia đình (dưới 9 chỗ) | Thân xe thấp
Không khoang chở hàng rời |
| `truck` | Phương tiện chuyên dùng chở hàng hóa, vật liệu. Bao gồm xe tải thùng, xe ben, xe bồn, xe đầu kéo container và xe bán tải | Cabin tách biệt với thùng hàng
Xe bán tải có thùng lửng phía sau |
| `bus` | Phương tiện giao thông công cộng hoặc dịch vụ vận chuyển hành khách quy mô lớn (từ 24 chỗ trở lên) | Thân dài hình hộp, nhiều hàng ghế, cửa sổ kính liên tiếp, có cửa lên xuống lớn |
| `van` | Phương tiện thân liền khối dạng hộp, kích thước trung bình giữa car và bus. Thường dùng chở khách hoặc chuyển phát cỡ nhỏ (10-16 chỗ) | Thân xe cao và vuông vức hơn car nhưng ngắn hơn bus. Khung vỏ liền khối không tách rời. |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: Các cạnh của box ôm khít 4 mép ngoài cùng của vật thể. Không bao gồm bóng đổ, ống xả, ăng ten hay các vật không cố định của vật thể.
- Vật thể bị che khuất: Nếu vật thể bị che khuất trên 80% diện tích hoặc không còn đủ đặc trưng nhận diện chắc chắn: Bỏ qua (không gán nhãn). Nếu bị che cắt làm hai phần bởi vật cản, vẽ 1 box duy nhất bao trọn cả hai phần của vật thể.
- Vật thể bị cắt bởi rìa ảnh: Đặt cạnh box sát mép viền của ảnh, không tự suy diễn hay vẽ box tràn ra ngoài không gian ảnh. Nếu phần vật thể xuất hiện trong ảnh nhỏ hơn 20% hoặc không còn đủ đặc trưng nhận diện chắc chắn: Bỏ qua (không gán nhãn)
- Vật thể rất nhỏ/mơ hồ: Bỏ qua các vật thể có kích thước quá nhỏ hoặc ở hậu cảnh quá xa, bị nhòe nét do chuyển động, không thể phân biệt bằng mắt thường. Chỉ gán nhãn nếu nhìn rõ đường viền kết cấu của vật thể

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_049` | Xe chỉ nhìn được phần đầu không thể nhận biết rõ là car hay van do không đầy đủ đặc trưng. | Label là car | Ưu tiên gán theo lớp mặc định là car |
| `drive_001` | Xe bị chia cắt bởi cột biển báo. Không thể phân biệt rõ giữa van và truck | Label là van | Do có dáng cao hình hộp và phần thân sau trùng màu với phần cabin nên khả năng cao là van |
| `drive_013` | Các vật thể chuyển động để lại bóng mờ lớn. Phân vân giữa việc có nên label các vật thể đó hay không | Vẫn label các vật thể | Do vẫn thể hiện rõ các đặc trưng của vật thể đó là gì |