# Who are you?

## Challenge

Let me in. Let me iiiiiiinnnnnnnnnnnnnnnnnnnn http://mercury.picoctf.net:52362/

## Observe

Truy cập đến trang web thì thấy

![image](https://hackmd.io/_uploads/BJMhKDrTa.png)

Dựa vào hint từ trang web "Only people who use the official PicoBrowser are allowed on this site!", mình quyết định đổi header `User-Agent` thành `User-Agent: PicoBrowser`

Sau khi gửi request thì nhận thấy response đã được thay đổi

![image](https://hackmd.io/_uploads/HJIU5vB66.png)

Để cho biết request này được truy cập từ site nào ta có thể sử dụng header `Referer: http://mercury.picoctf.net:52362/`

![image](https://hackmd.io/_uploads/rkq2qPBT6.png)

Hint này gợi ý về thời gian nên mình đã google và biết trong request có thể gửi kèm thời gian bằng header `Date: 2018`

![image](https://hackmd.io/_uploads/HkRZsPSap.png)

Tiếp theo mình google với keyword *"http header no tracking"* và được kết quả là header `DNT: 1`

![image](https://hackmd.io/_uploads/S19FjwBaT.png)

Mình google về header liên quan đến địa điểm của request thì đa phần các trang web đều nhận biết request đó đến từ nước nào thông qua IP. Và để gửi kèm IP trong request ta có thể dùng header `X-Forwarded-For: 102.177.146.1`, với IP đi kèm là IP của Sweden mà mình google được.

![image](https://hackmd.io/_uploads/SJrH3wSaT.png)

Tiếp theo ta chỉ cần thay đổi header `Accept-Language: sv-SE`

![image](https://hackmd.io/_uploads/rk7i3wST6.png)

Đã tìm thấy flag 🥳
