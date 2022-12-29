# Encryptor
![Screenshot](/pic/9.1.png)

__Challenge bao gồm 1 PE64 exe + SuspiciousFile.txt.Encrypted__

*usage: flareon path [path ...]*

Flow chương trình rất đơn giản, chỉ cần trỏ đường dẫn tới file cần mã hóa và file mã hóa được tạo ra có đuôi *.Encrypted*.

Khi phân tích file đã bị mã hóa (kết hợp phân tích mã nguồn), có được cấu trúc chương trình mã hóa:
- Đoạn dữ liệu bị mã hóa (dạng hex)
- 4 Đoạn giá trị Hex 256 bytes lưu dưới dạng ASCII.

Đây là một challenge thiên về crypto, sử dụng một số thuật toán khá khó để dịch ngược. Chỉ có thể xác định được flow chung của chương trình, còn những hàm sâu nhất phải sử dụng thêm *chiêu trò* mới có thể xác định được.

Flow chung chương trình gồm:
- Đọc file ban đầu
- Thực hiện các phép tính trước mã hóa (1)
- Bắt đầu mã hóa (2)
- Thực hiện các phép tính trong và sau mã hóa rồi ghi ra file mã hóa (3)
