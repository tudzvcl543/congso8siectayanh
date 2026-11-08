# Đề 24 – Hệ thống bán vé sự kiện và kiểm soát cổng


## 1. Sơ đồ Use Case

```mermaid
flowchart LR
    KH["Khách hàng"]
    QT["Quản trị viên"]
    NV["Nhân viên soát vé"]
    HT["Dịch vụ hệ thống"]

    subgraph CLIENT["APP CLIENT - FLUTTER"]
      C1([Duyệt, lọc và xem chi tiết sự kiện])
      C2([Chọn loại vé, số lượng, ghế])
      C3([Giữ chỗ 10 phút])
      C4([Áp mã giảm giá, xem tổng tiền])
      C5([Xác nhận đơn vé])
      C6([Nhận QR riêng cho từng vé])
      C7([Xem vé đã tải khi offline])
      C8([Chuyển nhượng bằng số điện thoại])
    end
    subgraph ADMIN["APP ADMIN - ANDROID JAVA + XML"]
      A1([Quản lý sự kiện])
      A2([Quản lý loại vé và khu, hàng, ghế])
      A3([Quản lý mã giảm giá và vé mời])
      A4([Theo dõi đã bán và còn lại])
      A5([Báo cáo doanh thu, lượt vào cổng, vé không dùng])
      A6([Tải danh sách vé hợp lệ vào SQLite])
      A7([Quét QR trực tuyến / ngoại tuyến])
      A8([Đánh dấu dùng tại máy và đồng bộ lô])
      A9([Nhận cảnh báo xung đột quét trùng])
    end
    subgraph SERVER["BACKEND - QUY TẮC NGHIỆP VỤ"]
      B1([Tự hết hạn giữ chỗ sau 10 phút])
      B2([Cấp mới QR và vô hiệu QR cũ])
      B3([Đối chiếu quét, phát hiện xung đột])
    end
    KH --- C1
    KH --- C2
    KH --- C3
    KH --- C4
    KH --- C5
    KH --- C6
    KH --- C7
    KH --- C8
    QT --- A1
    QT --- A2
    QT --- A3
    QT --- A4
    QT --- A5
    NV --- A6
    NV --- A7
    NV --- A8
    NV --- A9
    HT --- B1
    C8 -.-> B2
    A8 -.-> B3
    B3 -.-> A9
```

## 2. Sơ đồ cơ sở dữ liệu (ERD)

```mermaid
erDiagram
    NGUOI_DUNG {
        int id PK
        string so_dien_thoai
        string vai_tro
    }
    SU_KIEN {
        int id PK
        string ten
        text mo_ta
        string dia_diem
        decimal vi_do
        decimal kinh_do
        datetime bat_dau
        datetime ket_thuc
        string anh_bia
        string trang_thai
    }
    LOAI_VE {
        int id PK
        int su_kien_id FK
        string ten
        decimal gia
        int so_luong
        int da_ban
        datetime mo_ban_luc
        datetime dong_ban_luc
    }
    KHU_VUC {
        int id PK
        int su_kien_id FK
        string ten_khu
    }
    GHE {
        int id PK
        int khu_vuc_id FK
        string hang
        string so_ghe
        string trang_thai
    }
    GIU_CHO {
        int id PK
        int loai_ve_id FK
        int ghe_id FK
        int nguoi_dung_id FK
        datetime giu_den_luc
    }
    DON_VE {
        int id PK
        int nguoi_dung_id FK
        int su_kien_id FK
        decimal tong_tien
        string ma_giam_gia
        string trang_thai
    }
    VE {
        int id PK
        int don_ve_id FK
        int loai_ve_id FK
        int ghe_id FK
        string ma_qr
        string trang_thai
        int chu_so_huu_id FK
    }
    QUET_VE {
        int id PK
        int ve_id FK
        string cong
        datetime thoi_diem
        int nguoi_quet_id FK
        string ket_qua
        datetime dong_bo_luc
    }
    MA_GIAM_GIA {
        int id PK
        int su_kien_id FK
        string ma
        decimal phan_tram
        int so_luot_toi_da
        int da_dung
        datetime han_dung
    }
    CHUYEN_NHUONG {
        int id PK
        int ve_id FK
        int tu_nguoi_id FK
        int den_nguoi_id FK
        datetime thoi_diem
        string ma_qr_moi
    }

    SU_KIEN ||--|{ LOAI_VE : "co"
    SU_KIEN ||--o{ KHU_VUC : "gom"
    KHU_VUC ||--o{ GHE : "chua"
    SU_KIEN ||--o{ DON_VE : "phat_sinh"
    SU_KIEN ||--o{ MA_GIAM_GIA : "ap_dung"
    NGUOI_DUNG ||--o{ GIU_CHO : "giu"
    NGUOI_DUNG ||--o{ DON_VE : "dat"
    NGUOI_DUNG ||--o{ VE : "so_huu"
    NGUOI_DUNG ||--o{ QUET_VE : "thuc_hien"
    NGUOI_DUNG ||--o{ CHUYEN_NHUONG : "gui"
    NGUOI_DUNG ||--o{ CHUYEN_NHUONG : "nhan"
    LOAI_VE ||--o{ GIU_CHO : "giu_cho"
    GHE |o--o{ GIU_CHO : "giu_ghe_tuy_chon"
    DON_VE ||--|{ VE : "chua"
    LOAI_VE ||--o{ VE : "thuoc"
    GHE |o--o{ VE : "ve_ngoi_tuy_chon"
    VE ||--o{ QUET_VE : "lich_su_quet"
    VE ||--o{ CHUYEN_NHUONG : "lich_su_chuyen"
```


## 3. Bảng phân công

| Thành viên  | Phụ trách chính | Công việc cụ thể | Sản phẩm bàn giao |
|---|---|---|---|
| Trần Tuấn Tú - 241A010591 – Backend & CSDL | Thiết kế hệ thống, nghiệp vụ vé | Thiết kế CSDL và quan hệ; API sự kiện, giữ chỗ 10 phút, đơn vé; giao dịch chống bán quá số lượng, khóa ghế; mã giảm giá, chuyển nhượng và đổi QR; tạo dữ liệu mẫu | ERD, SQL migration/seed, API tài liệu và kiểm thử giao dịch |
| Nguyễn Thanh Tiến - 231A010705 – App Admin (quản trị) | Android Java + XML – quản lý | CRUD sự kiện, loại vé, sơ đồ khu/hàng/ghế, mã giảm giá và vé mời; dashboard đã bán/còn; báo cáo doanh thu và lượt vào cổng theo giờ, vé không dùng | Mã nguồn Admin, màn hình quản trị, báo cáo và demo |
| Đỗ Nhật Long - 241A010052 – Soát vé & đồng bộ | Android Java + XML – chế độ cổng, kết hợp Backend | Quét QR online/offline; tải trước danh sách vào SQLite; đánh dấu vé đã dùng và lưu log cục bộ; đồng bộ theo lô; phối hợp TV1 triển khai kiểm tra quét trùng hai cổng và trả thông báo xung đột | Module scanner, SQLite, API đồng bộ, kịch bản kiểm thử 2 cổng mất mạng |
| Lê Thiên Nga - 253A010012 – App Client (Flutter) | Giao diện người mua | Duyệt, lọc, xem chi tiết sự kiện; chọn loại vé, số lượng và ghế; đồng hồ giữ chỗ 10 phút; mã giảm giá/tổng tiền; đơn vé, QR riêng từng vé và lưu offline; chuyển nhượng qua số điện thoại | Mã nguồn Flutter, các luồng mua vé, lưu vé offline và demo |


## 4. Ghi chú kỹ thuật quan trọng

### Quy tắc triển khai phải kiểm thử

1. **Giữ chỗ 10 phút:** máy chủ xác định thời điểm hết hạn, không dùng bộ đếm phía Flutter làm nguồn sự thật. Giao dịch tạo đơn phải kiểm tra giữ chỗ còn hạn và khóa tồn kho/ghế để không bán trùng.
2. **QR một lần:** kiểm tra trạng thái vé bằng giao dịch nguyên tử phía máy chủ. `quet_ve.ket_qua` giữ các giá trị gốc `HOP_LE`, `DA_DUNG`, `SAI_SU_KIEN`.
3. **Quét ngoại tuyến:** mỗi máy Admin tải vé hợp lệ xuống **SQLite**, lưu log quét và đánh dấu đã dùng cục bộ. Khi có mạng, gửi lô có mã định danh riêng từng lần quét để thao tác đồng bộ có thể thử lại mà không ghi trùng.
4. **Hai cổng cùng offline:** không thể bảo đảm ngăn chặn tuyệt đối việc dùng trùng ngay tại thời điểm quét nếu hai máy không trao đổi dữ liệu. Máy chủ ghi nhận lượt hợp lệ đầu tiên theo quy tắc thống nhất và trả về mã lỗi nghiệp vụ **XUNG_DOT_QUET_TRUNG** cho lượt còn lại khi đồng bộ; không tự ý thêm giá trị này vào enum `quet_ve.ket_qua` của đề gốc.
5. **Chuyển nhượng:** QR cũ phải vô hiệu sau khi cấp QR mới trên server. Danh sách offline được tải trước có thể cũ nên phải cảnh báo, và xử lý xung đột khi đồng bộ.
6. **Điểm thiếu trong cấu trúc gốc:** các bảng có `nguoi_dung_id` nhưng đề chưa liệt kê bảng `nguoi_dung`, vì vậy ERD bổ sung **bảng phụ trợ tối thiểu** này. API lọc `chuDe` cũng cần có cột hoặc quan hệ chủ đề tương ứng; không tự sửa cấu trúc bảng `su_kien` được giao trước khi nhóm thống nhất.
7. **Dữ liệu mẫu:** tối thiểu **10 sự kiện × 4 loại vé**, trong đó **1 sự kiện có 300 ghế**, tổng **500 vé bán**, **350 lượt quét**. Có thể dùng loại vé thứ tư như Early Bird (bên cạnh Thường, VIP, Sinh viên), còn vé mời quản lý theo nghiệp vụ riêng.

