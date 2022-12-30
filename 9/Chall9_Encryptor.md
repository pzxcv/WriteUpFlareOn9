# Encryptor
![Screenshot](/pic/9.1.png)

__Challenge bao gồm 1 PE64 exe + SuspiciousFile.txt.Encrypted__

*usage: flareon path [path ...]*

Flow chương trình rất đơn giản, chỉ cần trỏ đường dẫn tới file cần mã hóa và file mã hóa được tạo ra có đuôi *.Encrypted*.

Khi phân tích file đã bị mã hóa (kết hợp phân tích mã nguồn), có được cấu trúc chương trình mã hóa:
- Đoạn dữ liệu bị mã hóa (dạng hex)
- 4 Đoạn giá trị Hex 256 bytes lưu dưới dạng ASCII, phân cách bằng 0xA.
![Screenshot](/pic/9_2.png)

Đây là một challenge thiên về crypto, sử dụng một số thuật toán khá khó để dịch ngược. Chỉ có thể xác định được flow chung của chương trình, còn những hàm sâu nhất phải sử dụng thêm *chiêu trò* mới có thể xác định được.

Flow chung chương trình gồm:
- Đọc file ban đầu
- Thực hiện các phép tính trước mã hóa (1)
- Bắt đầu mã hóa (2)
- Thực hiện các phép tính trong và sau mã hóa rồi ghi ra file mã hóa (3)

Dựa vào thông tin mà PEid cung cấp ở hình 1 (list các số *nguyên tố*), kết hợp với các đoạn số có kích thước bằng nhau khi phân tích file mã hóa, thì gần như chắc chắn RSA có xuất hiện trong challenge này, dù chưa biết ở dạng nào.

Khi tiến hành phân tích tĩnh, xác định được thêm 1 thuật toán mã hóa được sử dụng, đó là *Salsa20/Chacha* với string  __"expand 32-byte k"__.

Khi tiến hành phân tích động, với các hàm không xác định được chức năng thì tiến hành biện pháp *thử - sai*, rồi lấy ra kết quả. Trong trường hợp này ngoại trừ phần mã hóa Salsa2-/Chacha ra thì thử bằng các input của RSA với kết quả có thể tính được, sau đó đối chiếu. Từ đó xác định được thuật toán là RSA thuần, sử dụng số nguyên tố dưới dạng hex. (https://vi.wikipedia.org/wiki/RSA_(m%C3%A3_h%C3%B3a))
