### Báo cáo phân tích

#### 1\. Tại sao đường dẫn tuyệt đối gây lỗi khi đổi máy?

Đường dẫn tuyệt đối (như C:\\Users\\Admin\\Desktop\\Shopee-Lite\\assets\\logo.png) gắn chặt với cấu trúc ổ đĩa, tên người dùng (Admin) và hệ điều hành của máy gốc. Khi chuyển sang máy khác:

* Khác cấu trúc thư mục: Tên tài khoản Windows hoặc thư mục lưu trữ dự án trên máy mới không trùng khớp.  
* Hệ điều hành khác nhau: Phân cách thư mục trên Windows dùng \\, trong khi macOS/Linux dùng /.

Do đó, Python tìm file theo đúng địa chỉ cứng cũ và trả về lỗi FileNotFoundError.

#### 2\. Luồng IPO và nguyên nhân gây treo máy (RAM 98%)

\[INPUT\]                           \[PROCESS\]                 \[OUTPUT\]  
Storage (SSD/HDD)  \---\>  RAM & CPU Execution  \---\> Screen / Display  
(Ảnh logo, Code)         (Load dữ liệu vào bộ nhớ)  (Giao diện Shopee-Lite)

* Storage (Ổ cứng): Lưu trữ toàn bộ mã nguồn và file ảnh assets/ dưới dạng dữ liệu tĩnh, tốc độ truy xuất chậm hơn RAM.  
* RAM (Bộ nhớ trong): Khi khởi chạy ứng dụng, CPU gọi lệnh nạp dữ liệu từ Storage vào RAM để xử lý tức thì.  
* CPU: Đọc và thực thi các lệnh từ RAM.

Nguyên nhân gây tràn RAM (98%):

* Vòng lặp nạp file không kiểm soát: Mã nguồn chứa vòng lặp (như while True hoặc lặp render UI) đọc/nạp lại file ảnh từ Storage vào RAM liên tục mà không giải phóng bộ nhớ cũ.  
* Ảnh chưa tối ưu: Kích thước file ảnh quá lớn (dung lượng chưa nén) làm quá tải dung lượng RAM khả dụng.

### Giải pháp kỹ thuật

#### 1\. Viết lại cấu trúc đường dẫn tương đối

Dùng ký hiệu . (thư mục hiện tại) và .. (thư mục cha) kết hợp module os hoặc pathlib để đường dẫn tự điều chỉnh theo thư mục chạy thực tế.

Cấu trúc thư mục ví dụ:

Plaintext  
Shopee-Lite/  
├── assets/  
│   └── logo.png  
└── src/  
    └── main.py

Mã nguồn chuẩn hóa (chạy từ src/main.py):

BASE\_DIR \= Path(\_\_file\_\_).resolve().parent.parent  
LOGO\_PATH \= BASE\_DIR / "assets" / "logo.png"  
print(LOGO\_PATH)

#### 2\. 3 bước xử lý giảm lag dựa trên Task Manager

1. Bước 1 (Xử lý tức thời): Vào Task Manager $\rightarrow$ tab Processes $\rightarrow$ chọn tiến trình Python/Shopee-Lite chiếm nhiều tài nguyên nhất $\rightarrow$ bấm End Task để dừng ngắt tràn bộ nhớ.  
2. Bước 2 (Tối ưu Mã nguồn): Kiểm tra mã nguồn, đảm bảo file ảnh chỉ được nạp một lần duy nhất vào bộ nhớ đầu chương trình (chuyển biến lưu ảnh ra ngoài vòng lặp UI/main loop).  
3. Bước 3 (Tối ưu Tài nguyên Asset): Nén/resize toàn bộ file ảnh trong thư mục assets/ về độ phân giải và dung lượng phù hợp với giao diện hiển thị để giảm dung lượng RAM tiêu thụ khi nạp.

