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

Ở đoạn mã pha (1) như hình dưới, các hàm lần lượt tạo p và q, sau đó tính n, phi sau đó tính d. Ở hàm *sub_4016CC* thì thực hiện tính ra *c = m^e mod n* và lưu vào param đầu. Thực tế thì ở pha (1) thì biến __byte_409060__ là không có giá trị gì dù có trong file mã hóa. Lưu ý rằng giá trị e_like ban đầu (trước hàm sub_401B46) được gán bằng 0x10001, nhưng sau khi thực thi nó mang giá trị __*d*__ (phần này rất quan trọng).

![Screenshot](/pic/9_3.png)

Ở pha (2) và (3), chương trình mới lấy giá trị ra rồi tiến hành mã hóa với Salsa20/Chacha. Ngoài input ra thì để mã hóa, thuật toán này cần 2 param nữa là *key* 32 byte và *nonce* 12 byte, 2 param này được tạo ra bằng hàm random số __SystemFunction036__. Sau khi được mã hóa để tính *c* như pha (1) (output v9) thì ghi ra file mã hóa ở 256 bytes cuối cùng.
![Screenshot](/pic/9_4.png)

Các qword_409100, byte_4050A0 và byte_409060 lần lượt là n được tính từ pha 1, một hằng số trong chương trình và cuối cùng là con số "vô dụng" được nhắc từ trước. Và dĩ nhiên hằng số trên cũng "vô dụng" luôn. Và lý do sẽ được giải thích trong qua trình mã hóa chi tiết phía dưới.

***Tổng hợp lại chi tiết quá trình mã hóa***
- Pha (1) là quá trình tạo cặp khóa công khai (n,e) và vô tình tính luôn d (gán vào biến e_like). Lúc này tạm gọi d này là d1. Các giá trị m tính ra và hằng số byte_4050A0 ở param cuối hàm sub_4016CC là vô nghĩa vì param cuối đóng vai trò là số n, nhưng hằng số này không được sử dụng trong mã hóa lúc sau.
- Pha (2) và (3) sử dụng n tính từ (1) làm n chính trong việc tính c, với đầu vào là chuỗi giá trị key và nonce. Trong trường hợp này gọi là n2 và c2. Hàm tính c2 lúc này đã lấy n2 bằng n1, d1 để làm e, tạm gọi là e2=d1. Theo quy tắc của RSA, sẽ tồn tại cặp d,e sao cho d * e (trùng) (1 mod *phi*). Và do phép nhân có tính giao hoán, nên có thể đổi vai trò của 2 số này cho nhau được. Vì thế đã khiến khóa d vốn cần giữ bí mật thì lại bị lộ ngay từ ban đầu. Để giải mã, dùng công thức m=c^d mod n; Và d2 lúc này chính là e1 ban đầu (e1 * d1 tương ứng e2 * d2 do cùng n).
- Các giá trị hex trong file mã hóa lần lượt là giá trị của hằng số byte_4050A0 vô dụng, n được tính từ pha (1), byte_409060 vô dụng và n2 quan trọng.

**Vì thế quá trình giải mã như sau**
- Từ n2 có được và d2 bị lộ (bằng 0x10001), tính được m2, vốn là cặp key và nonce của thuật toán đối xứng Salsa20/Chacha
- Dùng cặp key và nonce và đoạn input bị mã hóa ở đầu file để giải mã văn bản gốc, từ đó có được flag.

Việc thực hiện giải mã có rất nhiều cách khác nhau, có thể thực hiện lại việc mã hóa, hoặc đơn giản hơn là đưa dữ liệu thẳng vào chương trình. Việc giải mã ra flag ban đầu sử dụng thuật toán mã hóa đối xứng, do vậy với thuật toán bất đối xứng, sau khi tự tính ra cặp key và nonce, sau đó patch vào debugger để chương trình tự giải mã ra data. Kết quả thu được là nội dung đoạn text có chứa flag.

![Screenshot](/pic/9_5.png)
