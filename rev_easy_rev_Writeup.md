# REV_EASSY_REV

### Đầu tiên khi chạy thử file thì chỉ có duy nhất 1 dòng là == Vaulted Flag Checker == và ta được yêu cầu nhập flag, nhập ngẫu nhiên cái gì đó vào thì nó hiện wrongflag! và không có gì xảy ra#
### Sau đó tiến hành dùng IDA để mở file, thấy rằng code có vẻ ngắn nên ta đọc sơ qua code và dùng tính năng chuyển sang mã giả của IDA để đọc qua pseudoCode.
### Một số chỗ dễ chú ý đầu tiên là char v55[128] này sẽ là vị trí flag mà ta nhập vào, ta sẽ đổi tên nó thành 'input_string'.
<img width="640" height="243" alt="image" src="https://github.com/user-attachments/assets/c9e4010d-cf58-454f-b78d-c7d62bd0720b" />.
### Sau đó là một đống phép tính xử lý khá là lang mang và cả filename cũng là một mục cần để ý nên ta để breakpoint để lát sau chạy debug để xem chương trình làm gì với nó.
### Tiếp đến là ta thấy chương trình gọi system_clock_gettime 2 lần và ở giữa là một đoạn mã tính toán gì đó có lẽ là vô nghĩa, có vẻ nó nhằm mục đích đo thời gian thực thi để phát hiện debug vì ngay bên dưới có một phép tính so sánh thời gian giữa 2 lần gettime đó với 0x11E1A300.
<img width="737" height="287" alt="image" src="https://github.com/user-attachments/assets/1b2f4054-738a-4f87-800f-853ce5242f71" />.
### Tiếp đó là phần tính toán gì đó khá là nhiều và rối nên ta tiến thẳng tới đoạn thông báo 'Wrong flag' và 'Correct', ta thấy điều kiện chỉ có if(v43)? Mà v43 ngay trên đó là addr(input_string). Chuyển sang x86 ta thấy lệnh đó là 'call rbx', nó gọi vào một thanh ghi nên rõ ràng là có một hàm ẩn tại vị trí đó.
### Ta liền đặt breakpoint tại đó và khi chương trình dừng tại đó mà ta step nó sẽ nhảy đến một cụm(?) gì đó và nó có mã giả như sau: 
 <img width="908" height="322" alt="image" src="https://github.com/user-attachments/assets/5c20ec6e-492a-43f1-ad84-a449d3180ea2" />.
### Đến đây em cứ tưởng là đã xong bài vì thấy cách mà nó xử lý flag nên viết một script python để tìm flag dựa trên nội dung của hàm đó và 27 byte từ off_7FFFF7FF1040 .
 <img width="962" height="198" alt="image" src="https://github.com/user-attachments/assets/908b4712-771e-4e77-a169-9d8f2180de12" />.
### Nhưng đời không như là mơ nên là flag đã không xuất hiện. 
 <img width="415" height="448" alt="image" src="https://github.com/user-attachments/assets/793da3a8-b2f1-413d-9cca-7677b3bd2085" />.
 <img width="405" height="51" alt="image" src="https://github.com/user-attachments/assets/6f0a2cce-6179-4cfe-8a8c-8a063e450f24" />.

### Quay lại với những dấu hiệu đã tìm được lúc nãy, đầu tiên là filename, ta sẽ đặt breakpoint ở đó và xem nó chứa cái gì sau khi xử lý. 
 <img width="627" height="55" alt="image" src="https://github.com/user-attachments/assets/7b8524c1-d1af-4f79-a0bb-83e8a88b1fa8" />.
### Vậy là sau khi xử lý thì nó sẽ trở thành chuỗi '/proc/self/status' và cái input ban đầu của ta nhập vào cũng trở thành 'TracerPid'.
### Em không đọc hiểu được cách LABEL_20 xử lý chuỗi tracepid nhưng ở cuối có thấy v21 có vẻ là đang ôm trạng thái có phát hiện debug hay không.
 <img width="827" height="117" alt="image" src="https://github.com/user-attachments/assets/52626c19-1088-450e-9208-0bf9aff9e46d" />.
### Việc phát hiện được như vậy còn là nhờ ở dưới đó ta có thấy một biến ôm điều kiện khác là v30, nó là kết quả phép xor giữa v21 và điều kiện thời gian nên ta đoán được điều đó.
### Chương trình tiếp tục dùng v30 để tính v31 và v32 băng cách xoay bit và khi v30 khác 0 (tức là thõa 1 trong 2 điều kiện phát hiện debug) thì v31 và v32 sẽ bị sai lệch đi.
 <img width="1085" height="87" alt="image" src="https://github.com/user-attachments/assets/28de056b-ae51-41ac-adfe-7216755d6eb8" />
### Và ở vòng lặp cuối, v32 sẽ mã hóa sai lệch toàn bộ 91 byte shellcode sinh ra trên RAM nên lệnh tính v43 (call rbx) nhảy vào một vùng nhớ chứa rác rồi mảng dữ liệu đích off_7FFFF7FF1040 bị sai lệch nên lúc nãy ta không giải ra được.
### Giờ ta thử debug lại nhưng lần này đặt break point sau khi tính v30 xong để modify giá trị của v30 thành 0 (hay thanh ghi RBP) rồi mới nhảy tiếp lệnh call RBX vào hàm ẩn.
### Lúc này ta thấy 27 bytes tại vị trí off_7FFFF7FF1040 đã thay đổi so với ban đầu, ta sẽ đem dãy bytes này giải mã lại bằng script lúc nãy và cuối cùng kết quả thu đươc là flag.
 <img width="942" height="607" alt="image" src="https://github.com/user-attachments/assets/902b72d0-5518-4848-bdcd-8a2780baf2cb" />.
 <img width="536" height="373" alt="image" src="https://github.com/user-attachments/assets/d433ab3a-da01-4623-92bd-82ef2a07aa14" />.
 <img width="376" height="116" alt="image" src="https://github.com/user-attachments/assets/11093a84-7032-4e21-94cb-3fd5868f0f75" />.




 
 


 




