## 4.1. Bảo vệ dữ liệu và chống ảo giác

### a) Ghi chú đã ẩn danh hoá

**GHI CHU DA AN DANH**

```text
Khán giả [TEN_KHACH], số điện thoại [SO_DIEN_THOAI_KHACH], số CCCD [SO_CCCD_KHACH], email [EMAIL_KHACH], đặt 3 vé gói SVT trận bán kết Cúp Sao Vàng ngày 18/12/2026 cho 2 người lớn và 1 trẻ em 9 tuổi. Khán giả đã chuyển khoản đặt cọc 1.500.000 đồng. Khán giả yêu cầu 3 ghế ngồi cạnh nhau gần lối đi.
```

### b) Hai câu lệnh chống ảo giác

**CAU LENH**

```text
Câu 1: Dựa trên file chính sách hoàn/hủy vé mà tôi gửi hãy trả lời câu hỏi trên của tôi. Nếu các thông tin không có trong chính sách thì không tự bịa chính sách mà trả lời trực tiếp là "CHÍNH SÁCH VỀ ĐIỀU KHOẢN NÀY KHÔNG TỒN TẠI"
Câu 2: Khi bạn trả lời cho tôi thì hãy ghi rõ câu trả lời lấy từ mục nào trong file chính sách hoàn/hủy đó.
```

## 4.2. Trích xuất tin nhắn đặt vé ra JSON

**PROMPT**

```text
[Context]: Bạn là một chuyên gia phân tích tin nhắn khách hàng, bạn có nhiệm vụ chuyển các tin nhắn thành định dạng JSON.
[Request]: Bạn hãy chuyển dữ liệu TN1, TN2, TN3 trong thẻ <tin_nhan> thành một mảng JSON 3 đối tượng, đúng thứ tự, đúng 6 trường như sau "ma_goi" , "ngay_su_kien" , "nguoi_lon" , "tre_em" , "loai_ve" , "ghi_chu". như tin nhắn mẫu
[Explantion]: Khi chuyển thành JSON bạn cần bám theo quy tắc chuẩn hóa trong thẻ <quy_chuan> . Bạn có thể xem qua thử 2 ví dụ mẫu mà tôi đã thực hiện trong thẻ ví dụ
<quy_chuan>
    BẢNG MÃ GÓI
    SVT – Sân Vận Động: vé trận đấu
    FZ – Fan Zone: khu cổ vũ
    VIP – Khán đài VIP

    QUY TẮC CHUẨN HOÁ (5 quy tắc)
    1. ma_goi lấy theo bảng mã gói.
    2. ngay_su_kien dạng YYYY-MM-DD; tin nhắn không ghi năm thì lấy năm 2026; không có ngày cụ thể thì null.
    3. Khách không nhắc trẻ em thì tre_em là 0 và mọi khách là người lớn.
    4. loai_ve chỉ nhận "thuong", "vip" hoặc null.
    5. ghi_chu chỉ ghi yêu cầu đặc biệt về chỗ ngồi, hỗ trợ di chuyển hoặc ăn uống; không có thì null.
    Câu trả lời không có chào hỏi, không có giải thích chỉ có dữ liệu trả về.
</quy-chuan>

<vi_du>
Tin nhắn mẫu 1:
Đặt giúp mình 2 vé VIP trận chung kết ngày 15/11/2026, 2 người lớn và 1 bé. Cần chỗ ngồi cạnh lối đi.
Kết quả đúng:
{"ma_goi": "VIP", "ngay_su_kien": "2026-11-15", "nguoi_lon": 2, "tre_em": 1, "loai_ve": "vip", "ghi_chu": "chỗ ngồi cạnh lối đi"}

Tin nhắn mẫu 2:
Cho mình một vé Fan Zone, ngày nào cũng được, mình đi một mình.
Kết quả đúng:
{"ma_goi": "FZ", "ngay_su_kien": null, "nguoi_lon": 1, "tre_em": 0, "loai_ve": "thuong", "ghi_chu": null}
</vi_du>
[Action]: Đọc dữ liệu của TN khách hàng mà tôi đưa => chuẩn hóa về đúng các trường dữ liệu => Trả đúng về định dạng JSON
[Tone]: Chuyên nghiệp, nghiêm túc, ngắn gọn, xúc tích
[Extras]: Định dạng đầu ra chỉ là dữ liệu JSON

<tin_nhan>
TN1: Mình muốn đặt gói Sân Vận Động trận tứ kết ngày 20/12 nha, 2 người lớn với 1 bé 8 tuổi, vé thường.

TN2: Cho 3 người đi Fan Zone trận ngày 05/01/2027 ạ. Một người cần hỗ trợ xe lăn.

TN3: Trận chung kết Cúp Sao Vàng tháng 12 còn vé không ạ? Nhà em 4 người lớn, có một người khiếm thị cần hỗ trợ tìm chỗ, lấy vé VIP.

</tin_nhan>

```

## 4.3. Báo giá vé bằng Chain-of-Thought

### a) Prompt

**PROMPT**

```text
Pha 1: 
Dựa vào thẻ <bang_gia>, thẻ <quy_tac_tinh_gia> và thẻ <yeu_cau_bao_gia> được cung cấp dưới đây, hãy đóng vai một chuyên gia tính toán chi phí để suy luận và tính tổng tiền thanh toán.
<bang_gia>
BẢNG GIÁ VÉ (đồng/người lớn)
SVT – Sân Vận Động: 1.200.000
FZ – Fan Zone: 800.000
VIP – Khán đài VIP: 2.500.000
</bang_gia>

<quy_tac_tinh_gia>
QUY TẮC TÍNH GIÁ
1. Trẻ em từ 5 đến 11 tuổi tính 70% giá người lớn. Trẻ dưới 5 tuổi miễn phí.
2. Nhóm từ 5 người lớn trở lên được giảm 5% trên tiền vé (tiền người lớn + tiền trẻ em).
3. Phụ thu khu ghế ưu tiên: 300.000 đồng/người. Phụ thu không được giảm giá.
4. Tổng tiền = tiền vé sau giảm + phụ thu.

</quy_tac_tinh_gia>

<yeu_cau_bao_gia>
YÊU CẦU BÁO GIÁ
Nhóm cổ động viên đi xem trận bán kết, mua gói SVT, gồm 5 người lớn, 2 trẻ em (8 tuổi và 10 tuổi) và 1 bé 3 tuổi. Có 1 người lớn chọn khu ghế ưu tiên.
</yeu_cau_bao_gia>

Bạn bắt buộc phải tính toán từng bước theo đúng thứ tự sau và ghi rõ phép tính toán học ở mỗi bước:
tiền người lớn → tiền trẻ em → tiền vé → giảm giá nhóm → phụ thu → tổng tiền

Yêu cầu định dạng: Dòng cuối cùng của kết quả bắt buộc phải ghi theo đúng định dạng sau (không thêm bất kỳ ký tự nào khác):
[RESULT: <tổng tiền> đồng]
```

## 4.4. Dây chuyền xử lý khiếu nại (Prompt Chaining)

### a) Prompt Trạm 1

**PROMPT**

```text
Nhiệm vụ của bạn là đọc nội dung email khiếu nại của khán giả và trích xuất thông tin sự cố. 

Nội dung email:
"Tôi là khán giả mua vé gói SVT trận Cúp Sao Vàng ngày 02/11/2026, đi 2 người lớn. Ban tổ chức cam kết khu khán đài có mái che nhưng trong 2 hiệp đầu chúng tôi bị xếp sang khu không mái, trời mưa và ghế ướt. Nhân viên soát vé rất nhiệt tình, tôi không có gì phàn nàn về nhân viên. Tôi đề nghị ban tổ chức giải quyết thỏa đáng."

Yêu cầu định dạng đầu ra:
Bạn chỉ được phép trả về đúng một đối tượng JSON, tuyệt đối không kèm theo bất kỳ văn bản chào hỏi, giải thích hay định dạng markdown nào khác ngoài JSON. Đối tượng JSON phải gồm chính xác 4 trường sau:
- "loai_su_co": Chỉ được chọn một trong các giá trị: "tran_gio", "sai_khu_vuc", "nhan_vien", "khac".
- "so_khach": (Số nguyên) Số lượng khách bị ảnh hưởng.
- "so_hiep_bi_anh_huong": (Số nguyên hoặc null) Số hiệp đấu bị ảnh hưởng.
- "tom_tat": (Chuỗi văn bản) Tóm tắt ngắn gọn khiếu nại, giới hạn tối đa không quá 20 từ.
```

### b) Hai điều cần kiểm tra ở JSON đầu ra của Trạm 1

1. Kiểm tra tính hợp lệ của cấu trúc (Cú pháp JSON): Đảm bảo đầu ra có thể parse được thành JSON chuẩn, không chứa văn bản rác ở đầu/cuối (ví dụ: "Đây là kết quả của bạn:") và có đủ đúng 4 keys bắt buộc {"loai_su_co", "so_khach", "so_hiep_bi_anh_huong", "tom_tat"}

2. Kiểm tra ràng buộc miền dữ liệu (Data constraints): Đảm bảo trường "loai_su_co" khớp chính xác 100% với một trong bốn từ khóa Enum được định nghĩa ("tran_gio" | "sai_khu_vuc" | "nhan_vien" | "khac") để Trạm 2 có thể ánh xạ đúng với chính sách đền bù. Đồng thời đếm số lượng từ trong trường "tom_tat" đảm bảo không quá 20 từ.

### c) Câu lệnh viết tiếp

**CAU LENH**

```text
Hãy tiếp tục viết email phản hồi bắt đầu ngay sau đoạn: '...Ban tổ chức Cúp Sao Vàng xin hoàn lại Quý khán giả khoản'. Tuyệt đối không sinh lại hay lặp lại những phần văn bản bạn đã viết trước đó.
```
