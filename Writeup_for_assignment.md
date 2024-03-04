

**1. Challenges 1**

**1.1.** Level 1 – Debug me

*Hình 1: level 1*

Level yêu cầu chúng ta tìm đọc log nên chúng ta sẽ đọc nội dung từ logcat:

*Hình 2: Xem logcat của ứng dụng bằng pid*

Phía dưới cùng:

*Hình 3: Flag*

➔ EVABS{logging\_info\_never\_safe}

**1.2. Level 2 – File Access**


*Hình 4: Gợi ý câu 2*

Theo như gợi ý, chúng ta sẽ đi tìm trong tệp assets khi decompile tệp .apk:


*Hình 5: Tìm được flag*

➔ EVABS{fil3s\_!n\_ass3ts\_ar3\_eas!ly\_hackabl3}

**1.3.** Level 3 - String

*Hình 6: Gợi ý level 3*


Gợi ý chỉ ra flag nằm trong tệp xml chứa các chuỗi -> có thể là tệp string.xml và nó luôn nằm trong thư mục values Sử dụng apktool của kali để decompile (ByteCode Viewer không xem được tệp values)

*Hình 7: Decompile*

Truy cập tới res/values/string.xml:

*Hình 8: Tìm kiếm bằng từ khoá "**api**"*

➔ EVABS{saf3ly\_st0red\_in\_Strings?}

**1.4.** Level 4 - Resources


*Hình 9: Level 4*

Resource đang nói đến cũng là tệp res, chúng ta tìm tới file có chứa “small toolkit” kia.


Theo như phân tích bằng BCV, tệp res\_raw.class (gồm cả $1) sẽ liên quan tới level này:

*Hình 10: Nội dung res\_raw$1.class*

Dựa theo tên lớp, truy cập vào tệp res/raw:

*Hình 11: Tìm thấy flag*



➔ EVABS{th!s\_plac3\_is\_n0t\_as\_s3cur3\_as\_it\_l00ks}

**1.5.** Level 5 – Shares and Preferences



*Hình 12: Level 5*


Thường thì dữ liệu các ứng dụng sẽ được lưu vào data/data/<app package> nên chúng ta sẽ mở các file trong đó để tìm mật khẩu.

*Hình 13: Lấy flag*

Hai tệp đầu là cache và code\_cache là 2 tệp thường thấy và log sẽ không ghi vào đây, chúng ta sẽ mở file shared\_prefs và lấy key.

➔ Flag: EVABS{shar3d\_pr3fs\_c0uld\_be\_c0mpromiz3d&#16;&#11;ޟw}

**1.6.** Level 6 – DB leak



*Hình 14: Level 6*

Level 6 yêu cầu chúng ta truy cập vào SQLite trong hệ thống, thực hiện truy cập lại vào data/data/com.revo.evabs:


*Hình 15: Xem giá trị có trong bảng*

Chúng ta có thể biết được nó nằm trong bảng CREDS dó khi phân tích mã nguồn, chúng ta thấy tài khoản Dr.l33t có quyền ADMIN:


*Hình 16: Phân tích*

➔ flag: EVABS{sqlite\_is\_not\_safe}

**1.7.** Level 7 - Export

*Hình 17: Yêu cầu level 7*

Dựa theo gợi ý, chúng ta sẽ tìm tới activity có gán nhãn là exported trong tệp Manifest.xml:

*Hình 18: Kết quả tại dòng 4*

Do nó được gán nhãn là exported=”true” nên chúng ta sẽ thử Hijacking tới nó:


*Hình 19: Không xuất hiện gì*

Tuy nhiên khi xem lại logcat, chúng ta thấy:

➔ flag: EVABS{exp0rted\_activities\_ar3\_harmful}

**1.8.** Level 8 - Decode

*Hình 20: Nội dung level 8*


Yêu cầu của level 8 sẽ là đọc code sau đó lấy các chuỗi trong đó giải mã mà ghép chúng lại với nhau sẽ được flag:

*Hình 21: Nội dung Decode.class*

Chuỗi có dấu = nên có thể là base64:


*Hình 22: Lấy được flag*

➔ EVABS{nev3r\_st0re\_s3ns!tiv3\_data\_1n\_7h3\_s0urcec0de}

**1.9.** Level 9 – Smali injection

*Hình 23: Level 9*

Level 9 yêu cầu chúng ta sửa code smali và sửa LAB\_OFF thành LAB\_ON để lấy flag.


Thực hiện sửa code:

*Hình 24: Sửa code smali*

Cuối cùng re-build và ký tệp apk:

*Hình 25: Rebuild apk*



Cài đặt lại ứng dụng và bấm nút Turn on, chúng ta được:

*Hình 26: Lấy được flag*

**1.10.** Level 10 – Intercept

**1.11.** Level 11 – Custom permission



*Hình 27: Level 11*



Với level 11, chúng ta được yêu cầu nhập vào chuỗi hợp lệ để chuyển sang action khác, theo dõi nội dung của code, chúng ta thấy có điểm đáng chú ý:

*Hình 28: Phần code giúp hiện flag*

Code trên đơn giản là nếu chúng ta nhập đúng chuỗi nó sẽ hiển thị ra flag và chuỗi đó chính là “cust0m\_p3rm”.

Tuy nhiên, khi nhập chuỗi trên, nó chuyển về chính trang này kèm thông điệp như dưới:

*Hình 29: Hiển thị thông điệp*

Vậy nên vấn đề là chúng ta cần tác động vào intent để flag hiện ra, code cụ thể như sau:

import frida

import sys

def onMessage(message, data):

print(message)

package = "EVABS"

jscode = """



Java.perform(function () {

send("[-] Starting hooks android.content.Intent.putExtra");

var intent = Java.use("android.content.Intent");

intent.putExtra.overload("java.lang.String", "java.lang.String").implementation = function(var\_1,

var\_2) {

send("[+] Flag: " + var\_2);

};

});

"""

process = frida.get\_usb\_device().attach(package)

script = process.create\_script(jscode)

script.on("message", onMessage)

print("[\*] Hooking", package)

script.load()

sys.stdin.read()

Dòng package chúng ta sẽ xác định bằng lệnh frida-ps -U và kết quả sẽ cho tên cùng với pid:



*Hình 30: Package sẽ là "EVABS"*

Thực hiện hooking với code trên và thực hiện lại bước nhập chuỗi “cust0m\_p3rm” để lấy flag:



*Hình 31: Lấy được flag*

**EVABS{always\_ver1fy\_packag3s}**

**1.12.** Level 12 - Intrusment

*Hình 32: Nội dung code level 12*



Chúng ta có thể thấy nội dung code sẽ có 2 biến a và b với kết quả được gán từ trước và x sẽ là kết quả của a nhân b, sau đó giá trị biến i sẽ được random với Random().nextInt(70) – giá trị I sẽ nằm trong khoản 0 đến 69.

Sau đó, kết quả sẽ được đưa vào lệnh if để so sánh x có lớn hơn i+150 hay không hay cũng chính so sánh a\*b với i+150. Nếu lớn hơn, thông điện sẽ được hiển thị trong log.

Do a và b trước đó chỉ gán là 25 và 2 nên kết quả chỉ là 50 không thể lớn hơn i+150 được, vậy nên chúng ta sẽ thay đổi giá trị của a hoặc b sao cho kết quả luôn lớn hơn khi kết quả i được random + 150.

*Hình 33: Sửa giá trị a*

Trong hình, chúng ta thay đổi giá trị a thành 0xDC tương ứng với 220 trong thập phân để giá trị sau khi nhân với b chắc chắn sẽ lớn hơn i+150 (max sẽ là 69+150 = 219).

Đóng gói và kí, chúng ta được:



*Hình 34: Tìm thấy flag*

**EVABS{a\_dynam1c\_h00k}**

**2. Challenges 2**

**2.1.** One.apk

*Hình 35: Level 1*

Xem log của ứng dụng:



*Hình 36: Xem log của app*

Sau đó chúng ta bấm nút HELLO, I AM A BUTTON, flag sẽ được hiển thị:

*Hình 37: Lấy được flag*

**picoCTF{a.moose.once.bit.my.sister}**



**2.2.** two.apk

Thực hiện mở tệp two.apk bằng BCV, chúng ta thấy code nằm trong phần MainActivity dùng để lấy flag như sau:

*Hình 38: Gọi tới getFlag*

Hàm getFlag có nội dung như dưới:

*Hình 39: Nội dung hàm getFlag*

Theo đó, hàm getFlag sẽ trả về kết quả dựa trên chuỗi nhập vào, nếu chuỗi nhập vào trùng với chuỗi giá trị chuỗi được gán cho “**2132427375**”, thì flag sẽ được hiển thị, ngược lại nó sẽ hiển thị NOPE:



*Hình 40: Chuỗi nhập vào không đúng*

Dữ liệu liên quan tới các chuỗi sẽ được tìm thấy trong lớp R$string, chúng ta thấy nó sẽ là **password**:



*Hình 41: Password được gán với 2131427375*

Tiếp đó, chúng ta sẽ tìm giá trị chuỗi được gán cho password trong tệp res/values/string.xml:

*Hình 42: Tìm thấy chuỗi cần nhập vào*



Chuỗi chúng ta sẽ nhập vào ứng dụng là opossum:

*Hình 43: Tìm thấy flag*

**picoCTF{pining.for.the.fjords}**



**2.3.** three.apk

Nội dung code getFlag trong level 3:

*Hình 44: Hàm getFlag*

Chúng ta cần thực hiện tính toán các giá trị j, m, I, k để từ đó đưa vào kết quả lấy các chuỗi với vị trí tương ứng:

return paramString.equals("".concat(arrayOfString[k]).concat(".").concat(arrayOfString[m]).concat(".").

concat(arrayOfString[j]).concat(".").concat(arrayOfString[k + j - m]).concat(".").concat(arrayOfString[3]).

concat(".").concat(arrayOfString[i])) ? sesame(paramString) : "NOPE";

Qua tính toán, chúng ta được như sau:

j = 0

m = 1

i = 1 + 1 - 0 = 2

k = 3 + 2 = 5

k+j-m = 5 + 0 - 1 = 4

Thứ tự các giá trị được gọi tới trong dòng code trên sẽ là 5\.1.0.4.3.2

Tương ứng với:

dismass.ogg.weatherwax.aching.nitt.garlick



Nhập chuỗi trên vào ứng dụng, chúng ta có flag:

**picoCTF{what.is.your.favourite.colour}**

**2.4.** four.apk

Nội dung hàm getFlag của level 4:



*Hình 45: Nội dung hàm getFlag*

Ta thấy hàm getFlag luôn được return về hàm nope(), vậy nên chúng ta sẽ thử thay đổi để nó gọi tới hàm yep:

*Hình 46: Thay đổi code trong hàm getFlag*

Sửa dòng 25 từ “nope” thành “yep”, sau đó rebuild và kí:



*Hình 47: Repatch apk*

Kết quả chúng ta được:

*Hình 48: Lấy được flag*

picoCTF{tis.but.a.scratch}

**2.5.** five.apk



*Hình 49: Nội dung hàm getFlag*

Hàm getFlag của câu 5 cũng tương tự câu 3 là chúng ta cần tính toán, nhưng câu 5 khi tìm được đúng chuỗi nó chỉ trả về thông điệp là “call it”, vậy nên chúng ta cần thực hiện sửa code sao cho nó gọi tới hàm cardamom():

*Hình 50: Thay đổi gọi về cardamom của lệnh if*

Thực hiện patch app và kí:



*Hình 51: Repatch apk*

Sau khi cài đặt lại ứng dụng, chúng ta tiếp tục với bước tính toán:

*Hình 52: Code sau khi đổi*

stringBuilder1:

• Ký tự đầu tiên (index 0): 'a' + 4 = 'e'

• Ký tự thứ hai (index 1): 'a' + 19 = 't'

• Ký tự thứ ba (index 2): 'a' + 18 = 's'

stringBuilder2:

• Ký tự đầu tiên (index 0): 'a' + 7 = 'h'

• Ký tự thứ hai (index 1): 'a' + 0 = 'a'

• Ký tự thứ ba (index 2): 'a' + 1 = 'b'



stringBuilder3:

• Ký tự đầu tiên (index 0): a' + 0 = 'a'

• Ký tự thứ hai (index 1): 'a' + 11 = 'l'

• Ký tự thứ ba (index 2): 'a' + 15 = 'p'

stringBuilder4:

• Ký tự đầu tiên (index 0): 'a' + 14 = 'o'

• Ký tự thứ hai (index 1): 'a' + 20 = 'u'

• Ký tự thứ ba (index 2): 'a' + 15 = 'p'

Nối các chuỗi trên theo thứ tự 3-2-1-4, chúng ta được keyword:

“**alphabetsoup**”

Nhập keyword và lấy flag:



*Hình 53: Lấy được flag*

**picoCTF{not.particulary.silly}**