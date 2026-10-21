# Tài liệu về câu lệnh Windows: CMD, BAT, PowerShell và hộp thoại Run

Phạm vi: các lệnh dùng trên Windows 11 (bản đang kiểm tra là Windows 11 Home Single Language, 10.0.26200), gồm:

- Hộp thoại Run (Win + R): mở nhanh ứng dụng, thư mục, công cụ quản trị.
- CMD (Command Prompt, `cmd.exe`): shell cũ, vẫn rất phổ biến.
- File batch (`.bat`, `.cmd`): tập lệnh chạy bằng CMD.
- PowerShell (`powershell.exe` cho Windows PowerShell 5.1, `pwsh.exe` cho PowerShell 7+): shell mạnh hơn, làm việc với đối tượng.

Ghi chú về nguồn: các điều kiện đánh dấu **[Đã kiểm tra]** được xác nhận trực tiếp trên máy này bằng PowerShell 7.6.6 ngày 2026-10-21. Các nội dung còn lại dựa trên kiến thức nền về Windows và cần đối chiếu thêm với tài liệu Microsoft khi cần chắc chắn.

---

## PHẦN A. HỘP THOẠI RUN

### 1. Cách mở
- Nhấn `Win + R`, nhập lệnh, rồi nhấn Enter.
- Nhấn `Ctrl + Shift + Enter` thay vì Enter để chạy với quyền quản trị (có hộp thoại UAC).
- Hộp thoại Run không hiển thị kết quả của lệnh console. Lệnh như `ipconfig` sẽ mở cửa sổ CMD, chạy xong thì đóng. Muốn xem kết quả, dùng CMD hoặc PowerShell.

### 2. Ứng dụng và công cụ hay dùng

| Lệnh nhập vào Run | Tác dụng | Ghi chú |
|---|---|---|
| `notepad` | Mở Notepad | **[Đã kiểm tra]** có trong System32 |
| `calc` | Mở Calculator | **[Đã kiểm tra]** có trong System32 |
| `mspaint` | Mở Paint | Trên Windows 11, Paint là ứng dụng Store nên lệnh có thể không hoạt động nếu chưa cài. Không kiểm tra được trên máy này (**[Đã kiểm tra]**: không có trong System32) |
| `snippingtool` | Mở Snipping Tool | Trên Windows 11, phím tắt `Win + Shift + S` thường nhanh hơn. Không có trong System32 trên máy này |
| `charmap` | Bảng ký tự | **[Đã kiểm tra]** |
| `osk` | Bàn phím ảo | **[Đã kiểm tra]** |
| `cmd` | Mở Command Prompt | **[Đã kiểm tra]** |
| `powershell` | Mở Windows PowerShell 5.1 | **[Đã kiểm tra]** có App Path |
| `pwsh` | Mở PowerShell 7+ | Chỉ có nếu đã cài PowerShell 7. **[Đã kiểm tra]** có App Path |
| `wt` | Mở Windows Terminal | Có nếu đã cài Windows Terminal. Không có trên máy này |
| `msedge` | Mở Microsoft Edge | **[Đã kiểm tra]** có App Path |
| `chrome` | Mở Google Chrome | Chỉ hoạt động nếu đã cài Chrome. **[Đã kiểm tra]** có App Path |
| `code` | Mở VS Code | Chỉ hoạt động nếu VS Code được thêm vào PATH khi cài. Không có App Path trên máy này |
| `explorer` | Mở File Explorer | Lệnh chạy đúng. Nằm trong thư mục Windows, không phải System32 |
| `write` | Mở WordPad | Không có trong System32 trên máy này. Nên dùng tìm kiếm Start thay thế |

### 3. Công cụ quản trị và hệ thống

| Lệnh | Tác dụng | Ghi chú |
|---|---|---|
| `taskmgr` | Task Manager | **[Đã kiểm tra]** |
| `control` | Control Panel | **[Đã kiểm tra]**. Thêm tham số để mở trực tiếp mục, ví dụ `control printers` |
| `msconfig` | System Configuration (khởi động, dịch vụ, boot) | **[Đã kiểm tra]** |
| `services.msc` | Quản lý dịch vụ | **[Đã kiểm tra]** |
| `devmgmt.msc` | Device Manager | **[Đã kiểm tra]** |
| `diskmgmt.msc` | Quản lý ổ đĩa | **[Đã kiểm tra]** |
| `compmgmt.msc` | Computer Management | **[Đã kiểm tra]** |
| `eventvwr.msc` | Event Viewer (nhật ký sự kiện) | **[Đã kiểm tra]** |
| `taskschd.msc` | Task Scheduler | **[Đã kiểm tra]** |
| `resmon` | Resource Monitor | **[Đã kiểm tra]** |
| `perfmon` | Performance Monitor | **[Đã kiểm tra]** |
| `msinfo32` | Thông tin hệ thống chi tiết | **[Đã kiểm tra]** |
| `dxdiag` | Công cụ chẩn đoán DirectX | **[Đã kiểm tra]** |
| `winver` | Hiển thị phiên bản Windows | **[Đã kiểm tra]** |
| `sysdm.cpl` | Thuộc tính hệ thống (tên máy, Environment Variables, khôi phục) | **[Đã kiểm tra]** |
| `ncpa.cpl` | Kết nối mạng (Network Connections) | **[Đã kiểm tra]** |
| `appwiz.cpl` | Gỡ cài đặt ứng dụng (Programs and Features) | **[Đã kiểm tra]** |
| `desk.cpl` | Thuộc tính màn hình | **[Đã kiểm tra]** |
| `firewall.cpl` | Windows Defender Firewall | **[Đã kiểm tra]** |
| `optionalfeatures` | Bật hoặc tắt tính năng Windows | **[Đã kiểm tra]** |
| `cleanmgr` | Disk Cleanup | **[Đã kiểm tra]** |
| `regedit` | Registry Editor | Có trong thư mục Windows, không phải System32. **Sửa registry có thể làm hỏng hệ thống. Sao lưu trước khi sửa.** |
| `mstsc` | Remote Desktop Connection (client) | **[Đã kiểm tra]**. Dùng để kết nối máy khác |
| `certmgr.msc` | Quản lý chứng chỉ | **[Đã kiểm tra]** |
| `mmc` | Khung Microsoft Management Console trống | **[Đã kiểm tra]** |
| `gpedit.msc` | Local Group Policy Editor | **Không có trên Windows Home.** **[Đã kiểm tra]**: máy này là CoreSingleLanguage (Home), không có file này |
| `secpol.msc` | Local Security Policy | **Không có trên Windows Home.** **[Đã kiểm tra]** |

### 4. Đường dẫn thư mục và trang cài đặt

| Lệnh | Mở gì |
|---|---|
| `%temp%` | Thư mục tạm của người dùng hiện tại |
| `%appdata%` | Thư mục AppData\Roaming |
| `%localappdata%` | Thư mục AppData\Local |
| `%userprofile%` | Thư mục người dùng (C:\Users\<tên>) |
| `%programdata%` | Thư mục ProgramData |
| `shell:startup` | Thư mục khởi động cùng Windows (của người dùng hiện tại) |
| `shell:common startup` | Thư mục khởi động cho mọi người dùng |
| `shell:appsfolder` | Danh sách tất cả ứng dụng (kể cả Store) |
| `shell:sendto` | Thư mục "Gửi tới" |
| `ms-settings:` | Mở trang Settings |
| `ms-settings:windowsupdate` | Windows Update |
| `ms-settings:display` | Cài đặt hiển thị |
| `ms-settings:appsfeatures` | Ứng dụng đã cài |
| `ms-settings:network-status` | Trạng thái mạng |
| `ms-windows-store:` | Microsoft Store |

Ghi chú: các lệnh `ms-settings:` và `shell:` là URI do Windows đăng ký. Tên trang có thể khác giữa các bản Windows. Nếu một lệnh không mở được, thử mở Settings rồi tìm kiếm mục đó.

### 5. Chạy với quyền quản trị

Trong hộp thoại Run, nhập lệnh rồi nhấn `Ctrl + Shift + Enter`. Windows sẽ hỏi UAC. Một số công cụ (ví dụ `regedit`, `diskmgmt.msc`) cần quyền này để sửa đổi.

---

## PHẦN B. CMD (COMMAND PROMPT)

### 1. Mở CMD
- Mở Start, gõ `cmd`, rồi chọn Command Prompt.
- Hoặc nhập `cmd` trong Run.
- Mở với quyền quản trị: nhấp chuột phải và chọn "Run as administrator" (hoặc gõ `cmd` rồi nhấn `Ctrl + Shift + Enter`).

### 2. Khái niệm nền
- Dấu nhắc `C:\Users\admin>` cho biết thư mục hiện tại.
- Lệnh có dạng `tên_lệnh [tham_số] [đối_số]`. Tham số thường bắt đầu bằng `/` (kiểu Windows) hoặc `-`.
- Biến môi trường viết trong `%`, ví dụ `%USERPROFILE%`. Trong batch dùng `%biến%`, ở dấu nhắc dùng cùng cú pháp `%biến%`.
- Kết hợp lệnh: `&` chạy lần lượt, `&&` chạy tiếp nếu lệnh trước thành công, `||` chạy tiếp nếu lệnh trước thất bại, `|` đưa đầu ra vào lệnh sau.
- Chuyển hướng: `>` ghi đè file, `>>` ghi thêm vào cuối file, `2>` chuyển lỗi.

### 3. Lệnh điều hướng và file

| Lệnh | Tác dụng | Ví dụ |
|---|---|---|
| `dir` | Liệt kê thư mục | `dir /a` (kể cả file ẩn), `dir /s *.log` (tìm đệ quy) |
| `cd` | Đổi thư mục | `cd /d D:\VIBECODE` (`/d` để đổi cả ổ đĩa) |
| `mkdir` (`md`) | Tạo thư mục | `mkdir backup` |
| `rmdir` (`rd`) | Xóa thư mục | `rmdir /s /q old` (**xóa không hỏi lại**) |
| `copy` | Sao chép file | `copy a.txt b.txt` |
| `xcopy` | Sao chép thư mục | `xcopy src dst /e /i /y` |
| `robocopy` | Sao chép và đồng bộ thư mục (mạnh hơn) | `robocopy src dst /mir` (**`/mir` xóa file ở đích không có ở nguồn**) |
| `move` | Di chuyển, đổi tên | `move a.txt sub\` |
| `del` (`erase`) | Xóa file | `del /q temp.txt` (**không vào Recycle Bin**) |
| `ren` | Đổi tên | `ren a.txt b.txt` |
| `type` | In nội dung file | `type readme.txt` |
| `more` | In theo trang | `type log.txt \| more` |
| `findstr` | Tìm chuỗi trong file | `findstr /i "error" app.log` |
| `where` | Tìm vị trí file thực thi | `where python` |
| `fc` | So sánh hai file | `fc a.txt b.txt` |
| `tree` | Hiển thị cây thư mục | `tree /f` |

### 4. Lệnh hệ thống

| Lệnh | Tác dụng | Ví dụ |
|---|---|---|
| `systeminfo` | Thông tin hệ thống | `systeminfo \| findstr /i "Boot Time"` |
| `tasklist` | Liệt kê tiến trình | `tasklist /fi "imagename eq chrome.exe"` |
| `taskkill` | Kết thúc tiến trình | `taskkill /pid 1234 /f` (**`/f` ép buộc**) |
| `sc` | Quản lý dịch vụ | `sc query Spooler`, `sc start Spooler` |
| `net start` / `net stop` | Bật/tắt dịch vụ | `net stop "Print Spooler"` |
| `schtasks` | Quản lý tác vụ lên lịch | `schtasks /query /fo table` |
| `shutdown` | Tắt hoặc khởi động lại | `shutdown /r /t 0` (**khởi động lại ngay**), `shutdown /a` (hủy lệnh đang chờ) |
| `wmic` | Truy vấn WMI (cũ) | Microsoft đã đánh dấu ngừng dùng. Nên dùng PowerShell với `Get-CimInstance`. Không kiểm tra được khả năng có sẵn trên mọi bản Windows 11 |
| `set` | Liệt kê biến môi trường | `set path` |
| `setx` | Đặt biến môi trường vĩnh viễn | `setx MY_HOME "D:\apps"` (cần mở cửa sổ mới để thấy) |
| `echo` | In ra màn hình | `echo Xin chao` |
| `cls` | Xóa màn hình | `cls` |
| `title` | Đổi tiêu đề cửa sổ | `title Công việc` |
| `pause` | Tạm dừng, chờ nhấn phím | `pause` |

### 5. Mạng

| Lệnh | Tác dụng | Ví dụ |
|---|---|---|
| `ipconfig` | Xem địa chỉ IP, DNS, gateway | `ipconfig /all`, `ipconfig /flushdns` |
| `ping` | Kiểm tra kết nối | `ping -n 4 8.8.8.8` |
| `tracert` | Theo dõi đường đi gói tin | `tracert example.com` |
| `nslookup` | Tra cứu DNS | `nslookup example.com` |
| `netstat` | Kết nối và cổng đang mở | `netstat -ano \| findstr :8080` |
| `netsh` | Cấu hình mạng và tường lửa | `netsh wlan show profiles` |
| `curl` | Gửi yêu cầu HTTP (có trên Windows 10/11 bản mới) | `curl -I https://example.com` |

### 6. Sửa lỗi hệ thống

| Lệnh | Tác dụng | Ghi chú |
|---|---|---|
| `sfc /scannow` | Kiểm tra và sửa file hệ thống | Chạy với quyền quản trị |
| `DISM /Online /Cleanup-Image /CheckHealth` | Kiểm tra kho ảnh hệ thống | Chạy với quyền quản trị. Có thể dùng `/ScanHealth` và `/RestoreHealth` khi cần |
| `chkdsk C: /f` | Kiểm tra và sửa lỗi ổ đĩa | Ổ hệ thống thường cần khởi động lại để chạy |

---

## PHẦN C. FILE BATCH (.BAT VÀ .CMD)

### 1. Khái niệm
- File batch là file văn bản có đuôi `.bat` hoặc `.cmd`, chạy từng dòng bằng CMD.
- `.bat` và `.cmd` đều chạy bằng CMD và có cú pháp gần như giống nhau. Có khác biệt nhỏ về cách xử lý lỗi và một số hành vi mặc định. Với người mới, cứ dùng `.bat` là đủ. Chi tiết khác biệt chưa được đối chiếu trong tài liệu này.
- Dùng encoding phù hợp. Nếu có tiếng Việt, lưu file UTF-8 có thể làm CMD hiển thị sai. Có thể thêm `chcp 65001` ở đầu để đổi sang UTF-8, nhưng cần thử lại trên máy của bạn.

### 2. Mẫu bat hoàn chỉnh kèm giải thích từng dòng

Đoạn sau sao lưu một thư mục vào thư mục đích có ngày giờ. Tên file: `backup.bat`.

```bat
@echo off
rem Tắt hiển thị lệnh khi chạy, chỉ in kết quả
setlocal enabledelayedexpansion
rem Cho phép biến thay đổi bên trong vòng lặp (dùng !biến!)

set "SRC=D:\VIBECODE"
rem Đặt biến SRC là thư mục nguồn, dấu "" bao quanh để an toàn với khoảng trắng

set "DEST=D:\Backup"
rem Thư mục đích

for /f "tokens=1-3 delims=/ " %%a in ("%date%") do set "TODAY=%%c-%%b-%%a"
rem Tách ngày từ %date% (ví dụ 21/10/2026) thành năm-tháng-ngày, rồi lưu vào TODAY
rem Lưu ý: định dạng %date% phụ thuộc cài đặt vùng, cần kiểm tra lại trên máy

if not exist "%DEST%" mkdir "%DEST%"
rem Nếu thư mục đích chưa có thì tạo mới

robocopy "%SRC%" "%DEST%\%TODAY%" /e /r:1 /w:2 /np
rem robocopy sao chép thư mục nguồn sang đích con theo ngày
rem /e: kể cả thư mục rỗng; /r:1: thử lại 1 lần nếu lỗi; /w:2: chờ 2 giây; /np: không in tiến trình %

set "RC=%ERRORLEVEL%"
rem robocopy trả mã thoát, dưới 8 nghĩa là thành công hoặc có thay đổi nhỏ

if %RC% GEQ 8 (
    echo Sao luu that bai, ma loi %RC%
) else (
    echo Sao luu xong vao %DEST%\%TODAY%
)
rem Kiểm tra mã lỗi và in thông báo tương ứng

endlocal
rem Kết thúc phạm vi biến cục bộ
pause
rem Chờ nhấn phím trước khi cửa sổ đóng
```

Bảng ký hiệu trong mẫu trên:

| Cú pháp | Ý nghĩa |
|---|---|
| `@echo off` | Không in từng lệnh; `@` ở đầu dòng ẩn chính dòng đó |
| `rem` hoặc `::` | Chú thích |
| `set "TEN=giá_trị"` | Gán biến; dấu `"` giúp xử lý khoảng trắng và ký tự đặc biệt |
| `%TEN%` | Lấy giá trị biến |
| `setlocal` / `endlocal` | Biến chỉ tồn tại trong phạm vi giữa hai lệnh |
| `!TEN!` | Lấy giá trị biến khi chạy trong vòng lặp (cần `enabledelayedexpansion`) |
| `for /f ... in (...) do` | Vòng lặp đọc từng token của chuỗi |
| `%%a` | Biến vòng lặp trong file batch (ở dấu nhắc dùng `%a`) |
| `if ... ( ... ) else ( ... )` | Điều kiện |
| `%ERRORLEVEL%` | Mã thoát của lệnh vừa chạy |

### 3. Các mẫu thường dùng

```bat
@echo off
rem Kiểm tra và chạy lệnh rồi báo lỗi nếu thất bại
where python >nul 2>&1
rem Tìm python; >nul ẩn đầu ra; 2>&1 gộp lỗi vào cùng đầu ra (ẩn luôn)
if errorlevel 1 (
    echo Chua cai Python hoac khong co trong PATH
    exit /b 1
)
rem exit /b 1 thoát khỏi batch với mã lỗi 1
python --version
```

```bat
@echo off
rem Nhận tham số dòng lệnh: backup.bat ten_thu_muc
if "%~1"=="" (
    echo Thieu tham so
    exit /b 1
)
rem %~1 là tham số đầu tiên, bỏ dấu ngoặc kép bao quanh
echo Ban truyen: %~1
```

```bat
@echo off
rem Gọi lệnh khác và chờ nó xong
call other.bat arg1
rem call bắt buộc khi gọi file .bat từ bat khác, nếu không sẽ thoát luôn file hiện tại
goto :end
rem goto nhảy đến nhãn

:end
rem Nhãn bắt đầu bằng dấu :
echo Hoan tat
```

### 4. Lưu ý khi viết batch
- Tránh sửa file đang chạy.
- Lệnh `del` và `rd /s /q` không có thùng rác. Luôn kiểm tra biến đường dẫn trước khi chạy (ví dụ biến rỗng có thể làm lệnh xóa sai chỗ).
- Dùng `echo` trước để kiểm tra lệnh sẽ làm gì, ví dụ `echo del "%DEST%\*.tmp"`, rồi mới bỏ `echo`.

---

## PHẦN D. POWERSHELL

### 1. Khác biệt với CMD
- PowerShell làm việc với **đối tượng** (object), không chỉ văn bản. Lệnh `Get-Process` trả về danh sách tiến trình có thuộc tính như `Name`, `Id`, `CPU`.
- Lệnh có dạng **động từ-danh từ** (`Get-Service`, `Set-Location`). Dùng `Get-Command` để tìm lệnh.
- Có nhiều alias quen thuộc: `ls`, `dir`, `cd`, `cat`, `ps`, `kill`, `cp`, `mv`, `rm`. Một số alias khác với CMD, ví dụ `dir` và `cat` trong PowerShell không hoàn toàn giống CMD.
- Đường ống `|` đưa đối tượng sang lệnh tiếp theo.

### 2. Phiên bản và chính sách thực thi
- Windows PowerShell 5.1 (`powershell.exe`) đi kèm Windows.
- PowerShell 7+ (`pwsh.exe`) cài riêng. **[Đã kiểm tra]** máy này đang dùng PowerShell 7.6.6.
- Kiểm tra chính sách: `Get-ExecutionPolicy`. Mặc định trên Windows client thường là `Restricted` (cho Windows PowerShell) hoặc `RemoteSigned` (cho PowerShell 7). Nếu script `.ps1` không chạy được:

```powershell
# Chỉ đặt cho người dùng hiện tại, không cần quyền quản trị
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

Chỉ nới lỏng chính sách khi bạn hiểu rủi ro. Không tắt bảo vệ chỉ để chạy một script không rõ nguồn.

### 3. Lệnh cơ bản

| Lệnh | Tác dụng | Ví dụ |
|---|---|---|
| `Get-Help` | Xem hướng dẫn | `Get-Help Get-Process -Examples` |
| `Get-Command` | Tìm lệnh | `Get-Command *service*` |
| `Get-ChildItem` (`ls`, `dir`) | Liệt kê file | `Get-ChildItem -Recurse -Filter *.log` |
| `Set-Location` (`cd`) | Đổi thư mục | `Set-Location D:\VIBECODE` |
| `Get-Location` | Thư mục hiện tại | `Get-Location` |
| `New-Item` | Tạo file hoặc thư mục | `New-Item -ItemType Directory -Path .\backup` |
| `Copy-Item` (`cp`) | Sao chép | `Copy-Item a.txt b.txt` |
| `Move-Item` (`mv`) | Di chuyển | `Move-Item a.txt sub\` |
| `Remove-Item` (`rm`, `del`) | Xóa | `Remove-Item .\old -Recurse` (**không vào Recycle Bin**) |
| `Get-Content` (`cat`, `type`) | Đọc file | `Get-Content log.txt -Tail 20` |
| `Set-Content` / `Add-Content` | Ghi file | `Add-Content log.txt "dong moi"` |
| `Select-String` | Tìm chuỗi (như grep) | `Select-String -Path *.log -Pattern "error"` |
| `Test-Path` | Kiểm tra tồn tại | `Test-Path C:\Windows` |
| `Get-Process` (`ps`) | Tiến trình | `Get-Process chrome` |
| `Stop-Process` (`kill`) | Kết thúc tiến trình | `Stop-Process -Name notepad` (**kiểm tra tên trước**) |
| `Get-Service` | Dịch vụ | `Get-Service Spooler` |
| `Start-Service` / `Stop-Service` | Bật/tắt dịch vụ | `Stop-Service Spooler -Force` |
| `Get-CimInstance` | Truy vấn WMI/CIM (thay wmic) | `Get-CimInstance Win32_OperatingSystem` |
| `Get-ComputerInfo` | Thông tin máy | `Get-ComputerInfo -Property WindowsEditionId` |
| `Get-NetIPAddress` | Địa chỉ IP | `Get-NetIPAddress -AddressFamily IPv4` |
| `Test-NetConnection` | Kiểm tra cổng và kết nối | `Test-NetConnection example.com -Port 443` |
| `Invoke-WebRequest` / `Invoke-RestMethod` | Gửi yêu cầu HTTP | `Invoke-RestMethod https://api.github.com` |
| `Get-EventLog` / `Get-WinEvent` | Nhật ký sự kiện | `Get-WinEvent -LogName System -MaxEvents 20` |
| `Start-Process` | Mở chương trình | `Start-Process notepad` hoặc `Start-Process cmd -Verb RunAs` |
| `Get-ScheduledTask` | Tác vụ lên lịch | `Get-ScheduledTask \| Where-Object State -eq Ready` |

### 4. Đường ống và lọc

```powershell
Get-Process |
    Where-Object { $_.CPU -gt 10 } |
    Sort-Object CPU -Descending |
    Select-Object -First 5 Name, Id, CPU
```

Giải thích từng dòng:

| Dòng | Ý nghĩa |
|---|---|
| `Get-Process \|` | Lấy danh sách tiến trình và đưa vào đường ống. Dấu `\|` là ký tự đường ống (cần escape khi viết trong bảng Markdown) |
| `Where-Object { $_.CPU -gt 10 } \|` | Giữ tiến trình có CPU lớn hơn 10. `$_` là đối tượng hiện tại, `-gt` là "lớn hơn" |
| `Sort-Object CPU -Descending \|` | Sắp xếp giảm dần theo CPU |
| `Select-Object -First 5 Name, Id, CPU` | Lấy 5 dòng đầu, chỉ in ba thuộc tính |

Toán tử so sánh thường gặp: `-eq` (bằng), `-ne` (khác), `-gt` (lớn hơn), `-lt` (nhỏ hơn), `-like` (khớp mẫu có `*`), `-match` (regex), `-contains`.

### 5. Biến, vòng lặp, hàm

```powershell
# Biến bắt đầu bằng $
$src = "D:\VIBECODE"
$dest = "D:\Backup"

# Vòng lặp foreach: lặp qua từng file .md
foreach ($f in Get-ChildItem -Path $src -Filter *.md) {
    # $f là đối tượng FileInfo của từng file
    Write-Host "Tim thay: $($f.Name)"
    # $($f.Name) chèn thuộc tính vào chuỗi có nháy kép
}

# Hàm với tham số
function Backup-Folder {
    param(
        [Parameter(Mandatory)] [string] $Source,
        # Tham số bắt buộc, kiểu chuỗi
        [string] $Target = "D:\Backup"
        # Giá trị mặc định
    )
    Copy-Item -Path $Source -Destination $Target -Recurse -Force
    # -Recurse: sao chép đệ quy; -Force: ghi đè và bao gồm file ẩn
}
Backup-Folder -Source $src
```

Ký hiệu: `#` là chú thích, `$tên` là biến, `"..."` cho phép nội suy biến, `'...'` là chuỗi nguyên văn, `@( )` là mảng, `@{ }` là bảng băm (hashtable).

### 6. Mẫu script .ps1 an toàn
```powershell
# Kiểm tra ổ đĩa và in cảnh báo nếu còn ít dung lượng
[CmdletBinding()]
# Cho phép dùng tham số chung như -Verbose
param(
    [int] $WarnPercent = 15
    # Ngưỡng cảnh báo, mặc định 15%
)

Get-CimInstance Win32_LogicalDisk -Filter "DriveType=3" |
    # Chỉ lấy ổ đĩa cố định (DriveType=3)
    ForEach-Object {
        $freePct = [math]::Round($_.FreeSpace / $_.Size * 100, 1)
        # Tính phần trăm trống, làm tròn 1 chữ số
        if ($freePct -lt $WarnPercent) {
            Write-Warning "$($_.DeviceID) chi con $freePct% trong"
        } else {
            Write-Host "$($_.DeviceID) con $freePct% trong"
        }
    }
```

Chạy script: `.\kiem-tra-o-dia.ps1` hoặc `.\kiem-tra-o-dia.ps1 -WarnPercent 20`. Khi đã đổi chính sách thực thi như mục D.2.

### 7. Lệnh nguy hiểm, cần xác nhận trước
- `Remove-Item -Recurse -Force` xóa thư mục không qua thùng rác.
- `Stop-Process -Name` có thể dừng tiến trình quan trọng.
- `Format-Volume` và `Clear-Disk` xóa dữ liệu ổ đĩa.
- Dùng `-WhatIf` để xem trước: `Remove-Item .\old -Recurse -WhatIf`.
- Dùng `-Confirm` để hỏi lại: `Remove-Item .\old -Confirm`.

---

## PHẦN E. SO SÁNH NHANH CMD VÀ POWERSHELL

| Việc cần làm | CMD | PowerShell |
|---|---|---|
| Liệt kê file | `dir` | `Get-ChildItem` (`ls`) |
| Xem nội dung file | `type a.txt` | `Get-Content a.txt` |
| Tìm chuỗi | `findstr "x" a.txt` | `Select-String "x" a.txt` |
| Biến môi trường | `set VAR=1` | `$env:VAR = "1"` |
| Đọc biến | `echo %VAR%` | `echo $env:VAR` |
| Kiểm tra file | `if exist a.txt` | `Test-Path a.txt` |
| Tiến trình | `tasklist` | `Get-Process` |
| Kết thúc tiến trình | `taskkill /im x.exe /f` | `Stop-Process -Name x -Force` |
| Thông tin máy | `systeminfo` | `Get-ComputerInfo` |

Quy tắc chung: lệnh ngắn, nhanh và đã có sẵn trong nhiều hướng dẫn cũ thì dùng CMD. Lệnh cần xử lý dữ liệu, script phức tạp, hoặc cần đối tượng thì dùng PowerShell.

---

## PHẦN F. XỬ LÝ SỰ CỐ

| Triệu chứng | Nguyên nhân thường gặp | Cách kiểm tra và xử lý |
|---|---|---|
| Lệnh không được nhận diện | Chưa cài, chưa có trong PATH | `where lệnh` (CMD) hoặc `Get-Command lệnh` (PowerShell). Thêm đường dẫn vào PATH nếu cần |
| Run báo "Windows cannot find ..." | Tên sai hoặc ứng dụng chưa cài | Thử `shell:appsfolder` để tìm tên đúng, hoặc tìm trong Start |
| Lệnh chạy nhưng không thấy gì | Chạy trong Run nên cửa sổ đóng ngay | Mở CMD và chạy lệnh đó, hoặc thêm `& pause` (CMD) |
| Script `.ps1` không chạy | Execution policy | `Get-ExecutionPolicy -List`, chỉnh theo mục D.2 |
| Tiếng Việt hiển thị lỗi trong CMD | Encoding | Thử `chcp 65001`, hoặc lưu file UTF-8 có BOM, hoặc chạy trong Windows Terminal |
| `Access is denied` | Thiếu quyền quản trị | Chạy với Run as administrator (`Ctrl + Shift + Enter` trong Run) |
| `The process cannot access the file because it is being used by another process` | File đang bị khóa | Dùng `tasklist`/`Get-Process` tìm tiến trình, hoặc `Resource Monitor` (`resmon`) xem file nào đang mở |
| Lệnh `ERRORLEVEL` luôn là 0 | Thiếu `if errorlevel` hoặc dùng sai `%ERRORLEVEL%` trong khối `( )` | Dùng `!ERRORLEVEL!` với `enabledelayedexpansion`, hoặc tách lệnh ra ngoài khối |
| Biến rỗng trong vòng lặp `for` | Dùng `%biến%` trong khối thay đổi | Dùng `!biến!` và `setlocal enabledelayedexpansion` |
| `wmic` không chạy | Đã đánh dấu ngừng dùng | Dùng `Get-CimInstance` trong PowerShell |
| Mạng chậm hoặc không ra ngoài | DNS, cache | `ipconfig /flushdns`, `ipconfig /release` rồi `ipconfig /renew`, kiểm tra bằng `ping` và `nslookup` |
| Windows bị lỗi file hệ thống | File hỏng | `sfc /scannow`, sau đó `DISM` theo mục B.6 |

Nguyên tắc khi gỡ lỗi: chạy lệnh ở chế độ chỉ đọc trước (`dir`, `Get-ChildItem`, `Test-Path`), xem đầu ra, rồi mới thay đổi. Với lệnh xóa, dùng `-WhatIf` hoặc `echo` trước.

---

## PHẦN G. THUẬT NGỮ

| Thuật ngữ | Giải thích ngắn |
|---|---|
| Run | Hộp thoại mở nhanh bằng `Win + R` |
| CMD | Command Prompt, shell văn bản cũ của Windows |
| Batch | Tập lệnh `.bat` hoặc `.cmd` chạy bằng CMD |
| PowerShell | Shell và ngôn ngữ script dựa trên đối tượng |
| Cmdlet | Lệnh PowerShell có dạng động từ-danh từ |
| Alias | Tên ngắn của lệnh, ví dụ `ls` cho `Get-ChildItem` |
| PATH | Danh sách thư mục Windows tìm lệnh thực thi |
| Environment variable | Biến môi trường, ví dụ `%USERPROFILE%` |
| Execution policy | Chính sách quy định script PowerShell nào được chạy |
| UAC | User Account Control, hộp thoại xác nhận quyền quản trị |
| ERRORLEVEL | Mã thoát của lệnh CMD |
| Pipeline | Chuỗi lệnh nối bằng `\|` |
| Snap-in (`.msc`) | Công cụ quản trị đóng gói trong MMC |
| Control panel applet (`.cpl`) | Công cụ cài đặt dạng cửa sổ con của Control Panel |
| App Paths | Khóa registry cho phép Run tìm ứng dụng theo tên |
| URI scheme | Tiền tố như `ms-settings:` hoặc `shell:` để mở trang hoặc thư mục đặc biệt |

---

## PHẦN H. NGUỒN VÀ ĐIỂM CẦN XÁC MINH

### H.1. Đã kiểm tra trực tiếp trên máy (2026-10-21)
- Phiên bản Windows: Windows 11 Home Single Language (CoreSingleLanguage), build 10.0.26200.
- PowerShell 7.6.6.
- Các file lệnh có trong System32: notepad, calc, cmd, control, msconfig, taskmgr, services.msc, devmgmt.msc, diskmgmt.msc, compmgmt.msc, eventvwr.msc, taskschd.msc, resmon, perfmon, msinfo32, dxdiag, sysdm.cpl, ncpa.cpl, appwiz.cpl, desk.cpl, firewall.cpl, optionalfeatures, lusrmgr.msc, mstsc, charmap, osk, cleanmgr, winver, msra, certmgr.msc, mmc, chkdsk, sfc.
- App Paths có đăng ký: powershell, pwsh, msedge, chrome.
- Không có: gpedit.msc, secpol.msc (đúng với Home); mspaint, snippingtool, wt, code không có trong System32 hay App Paths trên máy này.
- regedit và explorer nằm trong thư mục Windows, không phải System32.

### H.2. Chưa kiểm tra, cần đối chiếu
- Chưa chạy thử các mẫu bat và PowerShell trong tài liệu. Các ví dụ là mã mẫu.
- Hành vi của `%date%` phụ thuộc cài đặt vùng và cần kiểm tra trên máy.
- Hành vi chính xác của `wmic` trên mọi bản Windows 11 chưa được xác nhận.
- Mặc định execution policy trên từng bản Windows chưa kiểm tra đầy đủ.
- Các URI `ms-settings:` có thể thay đổi giữa các bản.
- Paint, Snipping Tool và Windows Terminal có thể cài hoặc không, tùy máy.

### H.3. Nguồn tham khảo nên đọc thêm
- Tài liệu Microsoft Learn về Windows Command Line (cmd) và PowerShell (learn.microsoft.com).
- Tài liệu về `about_Execution_Policies`, `about_Operators`, `about_Environment_Variables` trong PowerShell.
- Tài liệu về `robocopy` và `DISM` trên Microsoft Learn.
