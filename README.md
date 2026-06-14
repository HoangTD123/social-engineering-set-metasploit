Giới thiệu

Đề tài "Tiển khai mô phỏng tấn công Social Engineering use SET Toolkit và Metasploit" xây dựng một môi trường thực thi mục tiêu:


Mô phỏng quy trình tấn công Social Engineering thực tế gồm 2 giai đoạn: Lừa đảo và Khai thác kỹ thuật (Reverse Shell) .
Đánh giá phản ứng của người dùng và hệ điều hành (Windows Defender) trước các kỹ thuật tấn công phổ biến.
Đánh giá khả năng phát hiện của các giải pháp bảo mật hiện có.
Đề xuất các biện pháp phòng vệ: MFA, đào tạo nhận thức, kiểm soát email, Đội Đỏ/Đội Xanh.



🏗️Mô hình hệ thống kiến ​​trúc kiến ​​trúc

<img width="761" height="670" alt="image" src="https://github.com/user-attachments/assets/4747b544-54be-4865-84f5-b96515cbf072" />




🛠️ Công nghệ & Công cụ sử dụng

Công cụđiểm đếnBộ công cụ kỹ thuật xã hội (SET)Mô phỏng tấn công phi kỹ thuật (lừa đảo, thu hoạch thông tin xác thực)Metasploit FrameworkSinh payload, lắng nghe và quản lý shell đảo ngượcmsfvenomTạo tải trọng windows/x64/meterpreter/reverse_tcp(đã được chuẩn bị, mã hóa)Apache2Máy chủ web phân phối trang lừa đảo và tải trọngKali LinuxHệ điều hành tấn công (Attacker)Windows 10 x64Hệ điều hành nạn nhân (Nạn nhân)


🚀 Hướng dẫn phát triển khai

Yêu cầu môi trường


Ubuntu Host (hoặc Kali Linux) với cài đặt Apache2
Metasploit Framework
Máy ảo Windows 10 x64 (ằm trong cùng mạng nội bộ/NAT)
Quyềnsudo


Bước 0 — Cấu hình mạng (nếu Kali chưa có IP)

Cách 1 — Xin IP từ DHCP (thử trước):

<img width="249" height="81" alt="image" src="https://github.com/user-attachments/assets/9d130572-7832-4e69-a3e1-e40bb11cceb5" />


Nếu inet 192.168.x.xxuất hiện → thành công, hãy sử dụng IP đó.

Cách 2 — Nếu DHCP không được cấp, hãy đặt IP tĩnh:

<img width="548" height="216" alt="image" src="https://github.com/user-attachments/assets/b06907f3-b9c7-43e7-8535-9afdaeb3d356" />




Bước 1 — Khởi động Apache & tạo web thư mục

<img width="448" height="222" alt="image" src="https://github.com/user-attachments/assets/951324e4-5852-45eb-a15e-58aea3113ae3" />


Kiểm tra quá trình chạy Apache (phải được tìm thấy Active: active (running)):

<img width="399" height="55" alt="image" src="https://github.com/user-attachments/assets/54ba0ed6-327b-4e77-96f8-56a7efcb7f5c" />

Bước 2 — Tạo trang lừa đảo (download.html)

Sử teedụng thay thế nanođể tránh lỗi trong một số trường môi trường:
<img width="1353" height="716" alt="image" src="https://github.com/user-attachments/assets/db505f35-9a4a-47d1-9d57-dbbe2d09017f" />


Kiểm tra tệp đã tạo:

ls -lh /var/www/html/fb/

Bước 3 — Tạo payload với msfvenom (được mã hóa)


👉 So sánh cách làm cũ:



⚠️ Thay đổi 192.168.80.135việc thực thi IP của Kali (kiểm tra bằng ip a).



<img width="725" height="254" alt="image" src="https://github.com/user-attachments/assets/480f465a-476b-4133-a256-798f2d714db9" />


Sao chép tải trọng vào web thư mục:

<img width="813" height="117" alt="image" src="https://github.com/user-attachments/assets/28c1ee69-8633-4d86-bc89-b711b4634e4c" />



✅ Tệp .exephải có khoảng kích thước 200–400KB . Nếu thấy ~7MB đang sử dụng cũ tải trọng (cấu hình sai).



Bước 4 — Khởi động Listener (Metasploit) với AutoMigrate


⚠️ Terminal 1 phải luôn mở trong suốt quá trình demo. Không đóng hoặc nhấn Ctrl+C.



<img width="224" height="75" alt="image" src="https://github.com/user-attachments/assets/caf2021a-a886-4cc8-ad0f-c669c3e4ba4b" />


Trong msfconsole, gõ từng dòng:
<img width="612" height="204" alt="image" src="https://github.com/user-attachments/assets/b5610a3d-27a5-4f00-8458-3261922ef47e" />


Phải thấy dòng:

[*] Started reverse TCP handler on 192.168.80.135:4567

Giải thích:


AutoRunScript migrate: tự động di chuyển tải trọng sang tiến trình khác ( explorer.exe) ngay khi có phiên → tải trọng không bị hủy khi nạn nhân đóng tệp .exe.
ExitOnSession false: handler tiếp tục lắng nghe ngay cả khi đã có session, phục vụ cho nhiều kết nối.


Bước 5 — Thử nghiệm trên máy Windows (Victim)


⚠️ Bắt buộc tắt Windows Defender trước khi chạy file. Đây là yêu cầu của lab môi trường.




Mở trình duyệt → truy cậphttp://192.168.80.135/fb/download.html
Tải tập tinsetup_windows_update.exe
Vào Windows Security → Bảo vệ khỏi virus và mối đe dọa → Quản lý cài đặt
Bảo vệ thời gian thực
Bảo vệ được cung cấp qua đám mây
Gửi mẫu tự động
Tắt Tamper Protection (quan trọng)
Chuột phải vào file → Chạy với tư cách quản trị viên



ℹ️ Sau khi chạy file, có thể thấy cửa sổ cmd flash nhanh rồi biến mất — đây là dấu hiệu bình thường , payload đang di chuyển sang tiến trình khác.



Bước 6 — Kiểm tra phiên

<img width="825" height="332" alt="image" src="https://github.com/user-attachments/assets/1af60e2a-cce1-428c-a4e5-bcfb524056d0" />

<img width="807" height="349" alt="image" src="https://github.com/user-attachments/assets/b1a80998-f1a1-412a-ac41-0aba390fa41e" />


🧩 Hậu khai thác (Sau khai thác)

<img width="761" height="435" alt="image" src="https://github.com/user-attachments/assets/cca0ed14-3b25-44a3-a35a-b1bb23f54063" />

Ví dụ tìm và đọc file:

<img width="535" height="110" alt="image" src="https://github.com/user-attachments/assets/62865649-2998-4f5b-bb3c-ab2cd1070791" />


