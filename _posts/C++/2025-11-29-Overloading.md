---
title: "OVERLOADING"
date: 2025-11-29 23:17:05 +0800
categories: [C++]
tags: [C++]
---

# NẠP CHỒNG TOÁN TỬ

Nạp chồng (overloading) toán tử thường được người lập trình sử dụng rất nhiều trong lập trình hướng đối tượng. Vậy nó có gì thú vị các bạn cùng tìm hiểu nhé!

Nạp chồng toán tử (overload operator) là bạn định nghĩa lại toán tử đã có trên kiểu dữ liệu người dùng tự định nghĩa để dễ dàng thể hiện các câu lệnh trong chương trình.

Trong C++ có các loại toán tử đơn, đôi, nhập xuất.

Bảng các toán tử có thể nạp chồng trong **C++**

![alt text](/assets/C++/Overloading/operator.png)

## Operator
Toán tử đơn là toán tử một ngôi, nó có thể là toán tử trước, toán tử sau ví dụ ++ , --,...

Toán tử đôi là các toán tử hai ngôi: + , - , * , / …

Toán tử nhập xuất là các toán tử chuẩn hóa theo thư viện istream (toán tử nhập >>) , ostream (toán tử xuất <<).

Các bạn cùng mình xét ví dụ dưới đây để hiểu rõ hơn nhé:

![alt text](/assets/C++/Overloading/example1.png)

![alt text](/assets/C++/Overloading/example2.png)

![alt text](/assets/C++/Overloading/example3.png)

![alt text](/assets/C++/Overloading/example4.png)

![alt text](/assets/C++/Overloading/example5.png)

**Kết quả:**

![alt text](/assets/C++/Overloading/result1.png)

**Giải thích:**
Trong phần ví dụ trên mình có hướng dẫn các bạn nạp chồng toán tử ++ tiền tố , ++ hậu tố, toán tử + một ngôi, toán tử - hai ngôi , toán tử so sánh hai ngôi, toán tử nhập >>, toán tử xuất <<

Cú pháp cho nạp chồng toán tử một ngôi:  

![alt text](/assets/C++/Overloading/synctax.png)

Đối với nạp chồng tiền tố, thì chúng ta cần phải cộng minute trước rồi mới làm việc, và khi cộng chúng ta nên chuẩn hóa thời gian lại.

**Ví dụ:**

![alt text](/assets/C++/Overloading/image21.png)

Đối với nạp chồng hậu tố, thì chúng ta cần phải thêm 1 biến ThoiGian T để lưu giá trị thời gian thực hiện tại lại để sau khi làm xong câu lệnh chứa toán tử ++ hậu tố rồi mới tăng giá trị minute lên thêm 1 đơn vị. 

**Ví dụ:**

![alt text](/assets/C++/Overloading/image22.png)

Đối với toán tử + một ngôi, xét câu lệnh T3 = T1 + T2 ở khối lệnh nạp chồng, ta cần phải khai báo thêm thực thể T để chứa các giá trị của thực thể T3, this->hour chính là giá trị hour của thực thể T1 , Ts.hour chính là giá trị hour của thực thể T2 và tương tự với minute.

Xét ví dụ cụ thể như trên:

Tạo thực thể T1(8,59) và thực thể T2(10,24), khai ta thực hiện thành công phép gán T3 = T1 + T2 tức là chúng ta đã nạp chồng thành công toán tử + và kết quả T3.

**Ví dụ:**

![alt text](/assets/C++/Overloading/image23.png)

Xét tiếp đến khối lệnh: 

![alt text](/assets/C++/Overloading/image24.png)

- Khi này T3.minute hiện tại đang là 23 nên nó sẽ thực hiện khối lệnh trong else và in ra dòng "Hau to true" qua đó có thể thấy chúng ta đã overloading toán tử ++ hậu tố thành công.
- Tương tự kiểm tra đối với overloading toán tử ++ tiền tố.
- Tiếp đến overloading toán tử 2 ngôi, đối với toán tử hai ngôi chúng ta cần phải nạp chồng nó thông qua hàm friend (hàm friend có thể truy cập tới các biến thành viên của lớp kể cả private). Tại sao chúng ta lại phải overloading nó thông qua hàm friend? Do toán tử 2 ngôi có 2 tham số truyền vào nên nó không thể dùng con trỏ this mà phải dùng hàm friend để truy cập đến biến thành viên của lớp.

**Cú pháp overloading cho toán tử 2 ngôi:**

![alt text](/assets/C++/Overloading/image25.png)

Ở trên mình có nạp chồng toán tử - 2 ngôi theo đúng cú pháp như trên các bạn tham khảo và nạp chồng tương tự với các toán tử +,*,/ nhé.

![alt text](/assets/C++/Overloading/image26.png)


Overloading các toán tử so sánh chúng ta chỉ cần áp dụng đúng như cú pháp mình vừa nêu trên, và mình có làm ví dụ với toán tử so sánh < các bạn tham khảo và áp dụng tương tự với các toán tử so sánh còn lại.

Mình có giải thích một chút là tại sao mình lại hay dùng tham chiếu & cho các đối số vì, khi ta truyền tham trị cho biến là ta đang dùng bản copy của nó đối với hàm, làm như này mất khá nhiều bộ nhớ, để giảm tải việc mất quá nhiều bộ nhớ chúng ta nên dùng trực tiếp nó thông qua địa chỉ, nếu bạn không muốn biến truyền vào bị thay đổi dữ liệu khi ra bên ngoài thì có thể thêm từ khóa const trước đối số truyền vào.

Ví dụ: `friend bool operator < (const ThoiGian& T1, const ThoiGian& T2)` như này thì các dữ liệu thành viên của các đối số sẽ không bị thay đổi khi ra ngoài. Tiếp đến là toán tử nhập, cú pháp:

![alt text](/assets/C++/Overloading/image27.png)

Trong C++ ta có thư viện chuẩn iostream được kêt hợp giữa 2 thư viện chuẩn nhập (istream) và xuất (ostream) vì vậy kiểu trả về ở nạp chồng toán tử nhập sẽ là istream còn toán tử xuất sẽ là ostream. 

**Ví dụ ở trên:**

![alt text](/assets/C++/Overloading/image28.png)

Ở đây các bạn phải bắt buộc truyền tham chiếu cho ThoiGian& T, vì khi ra khỏi chương trình thì đối số được nhập vào phải thay đổi theo main nên các bạn hết sức lưu ý với trường hợp nạp chồng toán tử nhập >> này. Và bắt buộc chúng ta phải thay cin thành is nhé (is là tên do mình đặt các bạn hoàn toàn có thể thay thế nó thành nên nào các bạn thích).

Nạp chồng toán tử xuất cũng tương tự với nạp chồng toán tử nhập, cú pháp:

![alt text](/assets/C++/Overloading/image29.png)

**Ví dụ ở trên:**

![alt text](/assets/C++/Overloading/image30.png)

Ở đây các bạn không cần thiết phải truyền tham chiếu cho ThoiGian& T vì khi ra khỏi hàm T không thay đổi nên đây là điểm khác biệt giữa nhập và xuất tuy nhiên mình vẫn khuyên khích các bạn truyền tham chiếu hơn vì lí do như mình đã kể trên. Và cout cũng phải bắt buộc thay bằng os.







