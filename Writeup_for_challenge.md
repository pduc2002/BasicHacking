**1. Crack M3**

Chúng ta sẽ sử dụng ByteCode Viewer để xem nội dung có trong MainActivity.class:

![alt text](<Images/Challenge/image (1).png>)
*Hình 1: Nội dung trong MainActivity.class*

Dựa vào kết quả từ điều kiện của var4, chúng ta sẽ tìm chuỗi input hợp lệ sao cho var4 trả về **true**.

Đầu tiên, var5 sẽ là input dựa trên nội dung dòng lệnh, còn var6 sẽ lấy từ chuỗi something, các bước để tìm nó như sau:

![alt text](<Images/Challenge/image (2).png>)
*Hình 2: Vào lớp R$string.class*

Tìm tới “something”:

➔ var6 là chuỗi “**no\_one\_can\_escape\_from\_me**”

![alt text](<Images/Challenge/image (3).png>)

Theo đó, vòng lặp for đầu tiên có nội dung như sau:

for(int var2 = 0; var2 < var5.length(); ++var2) {

var5.setCharAt(var2, (**char**)(var5.charAt(var2) + "something\_that\_nobody\_can\_touch".charAt(var2 % 31) ^ var6.charAt(var2 % var6.length())));

}

![alt text](<Images/Challenge/image (4).png>)

➔ Thay đổi giá trị của từng phần tử của var5 bằng cách lấy vị trí tương ứng (var2) ban đầu + vị trí var2 trong chuỗi “something\_that\_nobody\_can\_touch” sau đó XOR với phần tử var2 trong chuỗi var6 vừa tìm ở trên.

Các phép chia dư trong vòng for nhằm để nó không vượt ra ngoài giới hạn phần tử của các chuỗi.

Tiếp đến, chúng ta có:

Dễ dàng nhận thấy chuỗi var5 (chuỗi input) buộc phải dài 41 kí và đồng thời câu điều kiện if buộc phải không thoả mãn để tự để giá trị var4 = true, nghĩa là:

var5.charAt(var3) = (var8 ^ (new int[]{130, 96, 129, 40, 7, 253, 245, 36, 212, 199, 227, 87, 135, 195, 41, 87, 159, 156, 89, 154, 56, 188, 132, 161, 238, 9, 236, 9, 98, 231, 223, 209, 104, 207, 41, 149, 64, 154, 144, 60, 169})[var3])

var8 sẽ được random kiểu Int với seed 105 và sau đó đem AND 255, chúng ta không cần quá quan tâm vào nó bởi nó sẽ được mang vào nguyên code giải ngược lấy input.

Vậy là chúng ta có các phép tính cần phải chuyển đổi để tính ra var5:

int var5 = (arrayNum[i] ^ ((char)generator.nextInt() & 255));

var5 = ((var5 ^ var6.charAt(i % var6.length())) - "something\_that\_nobody\_can\_touch".charAt(i % 31));

Cuối cùng đưa nó vào vòng for với giới hạn phần tử của mảng số nguyên [130,…,169]

Code hoàn chỉnh như sau:

import java.util.Random;

public class Main {

public static void main(String[] args) {

Random generator;

generator = new Random(105);

String results = new String("");

String var6 = "no\_one\_can\_escape\_from\_me";

int[] arrayNum = {130, 96, 129, 40, 7, 253, 245, 36, 212, 199, 227, 87, 135, 195, 41, 87, 159, 156, 89, 154, 56, 188, 132, 161, 238, 9, 236, 9, 98, 231, 223, 209, 104, 207, 41, 149, 64, 154, 144, 60, 169};

for (int i = 0; i < arrayNum.length; i++) {

int var5 = (arrayNum[i] ^ ((char)generator.nextInt() & 255));

var5 = ((var5 ^ var6.charAt(i % var6.length())) - "something\_that\_nobody\_can\_touch".charAt(i % 31));

results+= (char) var5;

}

System.out.println(results);

}

}

Chuỗi nhận được là: **flag{4ndr0id\_r3v\_5ucks55555555\_@$\#\&\#$^#$}**


![alt text](<Images/Challenge/image (5).png>)
*Hình 3: Lấy được flag*

**2. FlappyBird**

![alt text](<Images/Challenge/image (6).png>)

Với app thứ 2, khi tìm kiếm trong các lớp thì chúng ta có thể thấy được hàm getFlag nằm trong Champion.class, nó sẽ được gọi ngay khi activity này được gọi:

Và ChampionActivity sẽ được gọi khi người chơi đạt được 999999999 điểm, nội dung nó nằm ở gameView$2$1.class:

![alt text](<Images/Challenge/image (7).png>)

Thực hiện chỉnh sửa bằng cách dùng:

apktool d -f -r app-release\_chall.apk

Do nếu chỉ dùng d thì khi build sẽ xảy ra lỗi như dưới mặc dù chưa chỉnh sửa gì:

![alt text](<Images/Challenge/image (8).png>)
*Hình 4: Build lỗi*

Vậy nên cần sử dụng lệnh trước đó đã đề cập, sau đó truy cập vào tệp và thực hiện chỉnh sửa;

Thêm 1 chút vào chương trình onCreate của Champion.class, mục đích là để bật logcat lấy ngay flag mà không cần gõ lại kết quả trên màn hình:

![alt text](<Images/Challenge/image (9).png>)
*Hình 5: Tạo 1 log với nội dung là flag trong Champion.class*

Đổi 999999999 sang giá trị hex và tìm tới dòng 355, đổi nó thành 0x0 (0 điểm), cuối cùng build lại ứng dụng và kí (tên ứng dụng hơi sai do đang lấy tạm ứng dụng đã build thành công để viết lại các bước báo cáo):

![alt text](<Images/Challenge/image (10).png>)


![alt text](<Images/Challenge/image (11).png>)
*Hình 6: Kí cho tệp apk*

Chạy app và xem kết quả:

![alt text](<Images/Challenge/image (12).png>)
*Hình 7: Bị fake flag*

Tuy nhiên, để ý tới hàm gameInfo trong LogHelper, chúng ta thấy dòng Signature[]….

![alt text](<Images/Challenge/image (13).png>)
*Hình 8: Xem lại nội dung LogHelper.class*

Đây chính là bước giúp trích xuất danh sách các chữ ký (signatures) của gói ứng dụng (package) hiện tại và đem đi mã hoá ở các dòng lệnh sau, nó giúp cho việc xác định tính toàn vẹn của ứng dụng, nghĩa là việc sửa app đã bị phát hiện và chúng ta nhận được dòng chữ trên.

Vậy nên, việc build lại app cần có 1 bước sửa hàm này return thẳng về chữ kí gốc thay vì phải lấy lại thông tin bằng PackageManager.

Xem xuống vị trí lưu của log, chúng ta thấy nó được lưu trong log/app.logs, tìm tới vị trí lưu của nó nên chúng ta có thể vào đây và lấy nội dung để hàm gameInfo có thể return thẳng về chuỗi trên.


![alt text](<Images/Challenge/image (14).png>)
*Hình 9: Xem nơi lưu chuỗi được mã hoá cũ*

Chúng ta sửa code trong gameInfo() return chuỗi này, thực hiện decompile để sửa nội dung app đã được sửa score để gọi tới Champion avtivity:


![alt text](<Images/Challenge/image (15).png>)
*Hình 10: Sửa code*

Thực hiện các bước build và kí tương tự như trên và vào lại app:


![alt text](<Images/Challenge/image (16).png>)
*Hình 11: Build và kí*

Xem lại nội dung app vừa làm:


![alt text](<Images/Challenge/image (17).png>)
*Hình 12: Có thêm log xem flag*

gameInfo() luôn trả về chữ kí cũ:

![alt text](<Images/Challenge/image (18).png>)
*Hình 13: Kết quả chỉnh sửa trong LogHelper*


![alt text](<Images/Challenge/image (19).png>)
*Hình 14: Khi được 0điểm thì lập tức nhảy*


![alt text](<Images/Challenge/image (20).png>)
*Hình 15: Copy flag từ logcat*

**flag{ck0n\_vj3c\_kh0\_d3\_d4n\_d4u}**


