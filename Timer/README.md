# WRITE UP

Challenge: Timer.apk

<img src="./media/image1.png" style="width:5.66798in;height:2.85969in" alt="Graphical user interface, application Description automatically generated" />

Cài đặt ứng dụng và mở lên thì thấy đây là một application cho ta chờ đợi. Có lẽ hết thời gian thì sẽ nhả flag. Mà bao giờ thì không biết. Sử dụng JD-GUI để xem mã giả:

<img src="./media/image2.png" style="width:6.5in;height:3.59514in" alt="Graphical user interface, text, application Description automatically generated" />

MainActivity khởi tạo các giá trị beg, k, now,t. Có hàm check isPrime():  
<img src="./media/image3.png" style="width:6.5in;height:3.925in" alt="Timeline Description automatically generated with medium confidence" />

Ở hàm OnCreate(), localHandler sẽ gọi postDelayed với class Runnable(). Trong đây, **this.t** sẽ được set bằng thời gian thực:

<img src="./media/image4.png" style="width:6.5in;height:3.62361in" alt="Text Description automatically generated" />

Để đọc được flag, **this.beg** phải &lt;= **this.t**, nhưng không dễ như vậy. **this.beg** được gán = 200000 + currentTime:

<img src="./media/image5.png" style="width:6.48877in;height:1.39566in" alt="Text Description automatically generated" />

Bên cạnh đó, ở bên dưới mỗi khi if trả về false, chương trình lại đệ quy gọi lại postDelayed(). Lúc này giá trị **this.k** được modify **this.k += 100** đồng thời biến count làm **this.k -= 1.** Mà giá trị **this.k** là input stringFromJNI2() – hàm để get flag ra:

<img src="./media/image6.png" style="width:6.5in;height:1.86389in" alt="Graphical user interface Description automatically generated with medium confidence" />

Giờ thì ta cần modify nó để bypass cái if này và gán k bằng giá trị phù hợp sau khi chạy qua 200000 giây:

<img src="./media/image7.png" style="width:6.5in;height:1.28333in" />

MainActivity sẽ gọi is2() (isPrime()) để kiểm tra thời gian có phải là prime hay không trước khi vào vòng for biến đổi giá trị k và thực hiện đệ quy bên trong đó. Do đó, ta sử dụng python để tìm ra giá trị k cuối cùng khi chạy qua đoạn mã giả bên trên. Ta tìm được **k = 1616384 (0x18aa00):**

<img src="./media/image8.png" style="width:5.20045in;height:0.84174in" alt="Text Description automatically generated" />

<img src="./media/image9.png" style="width:6.5in;height:3.52431in" alt="A screenshot of a computer Description automatically generated" />

Đoạn cần modify là đoạn bên dưới đây. Ta sẽ modify giá trị của k ngay khi khai báo với giá trị hex 0x18aa00. Sử dụng apktool để build soure ra smali. Ta sẽ modify trong MainActivity.smali:

<img src="./media/image10.png" style="width:5.00043in;height:1.20844in" alt="Text Description automatically generated" />

<img src="./media/image11.png" style="width:6.5in;height:2.72708in" alt="A screenshot of a computer Description automatically generated" />

const/4 v0, 0x0 ( int k = 0) const/4, 0x18aa00 ( int k = 1616384)

Tiếp theo, ta cần modify &lt;= ở if so sánh để đọc được flag thành &gt;. Tuy vậy, khi move vào runnable(), ta chỉ thấy smali invoke-virtual đến postDelayed(Ljava/lang/Runnable;j)Z:

<img src="./media/image12.png" style="width:6.5in;height:1.66042in" alt="Graphical user interface, text Description automatically generated" />

Check lại source, ta thấy còn 1 file alf MainActivity$1.smali, mở lên thì đó là **Ljava/lang/Runnable:**

<img src="./media/image13.png" style="width:6.5in;height:1.63333in" alt="Text Description automatically generated" />

Tìm kiếm chuỗi “AliCTF” vì bên dưới nó sẽ là đoạn if cần modify:

<img src="./media/image14.png" style="width:6.5in;height:0.75in" alt="Graphical user interface, text, application Description automatically generated" />

<img src="./media/image15.png" style="width:6.5in;height:2.24583in" alt="A screenshot of a computer Description automatically generated" />

Ở cuối dòng 52, MainActivity gọi if-gtz (nếu lớn hơn hoặc bằng) so sánh v0 với :cond\_0 (ở đây là so sánh this.beg với this.now). Do đó, ta sẽ modify trực tiếp v0 trước khi gọi if-gtz. Ta đổi từ **if-gtz** **if-ltz** (nhỏ hơn):

<img src="./media/image16.png" style="width:5.95545in;height:1.60339in" alt="Text Description automatically generated" />

Rebuild lại app:

<img src="./media/image17.png" style="width:5.268in;height:2.64976in" alt="Text Description automatically generated" />

Generate keytool:

<img src="./media/image18.png" style="width:6.5in;height:2.59167in" alt="Text Description automatically generated" />

Align lại file apk:

<img src="./media/image19.png" style="width:6.5in;height:1.91875in" alt="Text Description automatically generated" /> <img src="./media/image20.png" style="width:6.5in;height:1.87431in" alt="Text Description automatically generated" />

Sign cho app vừa aligned:

<img src="./media/image21.png" style="width:6.5in;height:0.60347in" />

Mở app lên và ta get được flag:

<img src="./media/image22.png" style="width:3.06503in;height:3.25182in" alt="A picture containing text, monitor, screen, mounted Description automatically generated" />

\- Flag: \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
