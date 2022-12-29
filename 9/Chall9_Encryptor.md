# Encryptor
![Screenshot](/pic/9.1.png)

__Challenge bao gồm 1 PE64 exe + SuspiciousFile.txt.Encrypted__

*usage: flareon path [path ...]*

Flow chương trình rất đơn giản, chỉ cần trỏ đường dẫn tới file cần mã hóa và file mã hóa được tạo ra có đuôi *.Encrypted*.

Khi phân tích file đã bị mã hóa (kết hợp phân tích mã nguồn), có được cấu trúc chương trình mã hóa:
- Đoạn dữ liệu bị mã hóa (dạng hex)
- 4 Đoạn giá trị Hex 256 bytes lưu dưới dạng ASCII.
