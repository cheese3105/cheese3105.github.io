---
title: Làm màu cho terminal
tags: blog
---

Vào một ngày đẹp trời mình đã quá mệt mỏi với terminal trắng đen của Mac nên mình đã tìm cách để custom nó. 

![image](https://hackmd.io/_uploads/r1TDH2hUye.png)

Nhưng những mục tiêu mình cần là:
- Phân biệt được dòng gõ lệnh và output
- Output phải có sự khác biệt giữa file và directory

Theo những hướng dẫn mình tìm được trên mạng, mọi người sẽ cài **iTerm2**, **oh my zsh**, ...

## iTerm2

**iTerm2** là trình giả lập terminal cho macOS thay thế ứng dụng Terminal mặc định. Nó cung cấp các tính năng bổ sung mà Terminal cơ bản không có, chẳng hạn như hỗ trợ tab, giao diện thân thiện hơn với người dùng, khả năng chia nhỏ các ngăn terminal, hồ sơ và cải thiện hiệu suất.

Ở đây mình sử dụng `homebrew` để tải iTerm2 về:

```
brew install --cask iterm2
```

Sau khi tải về thì search trong Applications và khởi động iTerm2 lên 

![image](https://hackmd.io/_uploads/B1N0wn2Ukl.png)

Trong cũng không khác gì ha :smile: 

## zsh

Để làm đẹp cho terminal cái ta cần cài là **oh my zsh** hoặc **oh my posh**. Tuy nhiên điều kiện là terminal shell mặc định của ta phải là `zsh`

Để kiểm tra ta có thể sử dụng lệnh

```
% echo $0
-zsh
```

Nếu kết quả không phải là zsh ta có thể install bằng lệnh brew sau:

```
brew install zsh
```

Sau khi install xong ta cần gõ lệnh sau trên terminal để đặt zsh thành shell mặc định

```
% chsh -s $(which zsh)
```

## [Oh my zsh ](https://github.com/ohmyzsh/ohmyzsh/)
**![image](https://hackmd.io/_uploads/ryP_cnn8Jl.png)**

Đây là cái sẽ giúp cho terminal của trông màu mè hơn :3 

Các lệnh có thể dùng để cài



| Method| Command |
| -------- | -------- |
| curl   | `sh -c "$(curl -fsSL https://install.ohmyz.sh/)"`    |
| wget   | `sh -c "$(wget -O- https://install.ohmyz.sh/)"`    | 
| fetch   | `sh -c "$(fetch -o - https://install.ohmyz.sh/)"`    | 

:::danger
Lưu ý: Khi install oh my zsh sẽ backup file `~/.zshrc` sẵn có sang một file khác tên là `~/.zshrc.pre-oh-my-zsh` sau đó sẽ **ghi đè** file `~/.zshrc` của bạn

-> Cần phải chỉnh sửa, copy lại các lệnh trong file backup và bỏ vào `~/.zshrc` để tránh việc các cấu hình trước đó của bạn cho các công cụ khác như python có thể sẽ bị lỗi :v 
:::

Chưa kịp custom thì mình đã được gợi ý một công cụ khác để làm màu :smile: 

Nên nào có dịp custom bằng oh my zsh thì mình sẽ update thêm :3 


## [Oh my posh](https://ohmyposh.dev/)

![image](https://hackmd.io/_uploads/BJetahhUyl.png)

Install:

```
brew install jandedobbeleer/oh-my-posh/oh-my-posh
```

Lệnh này sẽ install 2 thứ:

- **oh-my-posh** - file thực thi, nằm trong `$(brew --prefix)/bin`
- **themes** - The latest Oh My Posh themes

Nếu muốn custom theme có sẵn, ta có thể tìm chúng trong `$(brew --prefix oh-my-posh)/themes`

![image](https://hackmd.io/_uploads/BJbzJanLyl.png)

Về chỗ chữa theme trên máy mình thì có phần sẽ khác với hướng dẫn đôi chút 

Các theme trên máy mình sẽ nằm ở 

```
% ls /opt/homebrew/Cellar/oh-my-posh/24.18.0/themes
1_shell.omp.json			larserikfinholt.omp.json
M365Princess.omp.json			lightgreen.omp.json
agnoster.minimal.omp.json		marcduiker.omp.json
agnoster.omp.json			markbull.omp.json
```

Nếu nghiên cứu thay đổi các file json ở đây thì ta có thể 100% custom theo ý muốn. Nhưng do trình đỗ mỹ thuật còn kém nên mình quyết định sẽ dùng theme có sẵn 

Để xem ảnh mẫu có theme ta có thể xem [ở đây](https://ohmyposh.dev/docs/themes)

Để có thể sử dụng, oh my posh cũng như chọn theme mong muốn ta cần thêm 2 dòng sau vào file `~/.zshrc`

```
eval "$(oh-my-posh init zsh)"
eval "$(oh-my-posh init zsh --config /opt/homebrew/Cellar/oh-my-posh/24.18.0/themes/illusi0n.omp.json)"
```

Dòng đầu tiên thì giữ nguyên, dòng thử 2 thì thay đổi tuỳ theo đường dẫn đến file json của theme

Thêm xong thì save file lại và khởi động lại terminal/iTerm2

Kết quả

![image](https://hackmd.io/_uploads/rJywGThUkl.png)

Kkkk đã có màu :3 

## Thêm màu cho Directory

Sau khi tìm hiểu thì việc phân biệt màu giữa directory và file sẽ do command `ls` quy định

https://superuser.com/questions/528228/how-can-i-configure-the-color-of-ls-directory-under-zsh

Theo như bài viết ta chỉ cần thêm 3 dòng sau vào file `~/.zshrc`

```
export CLICOLOR=1
export LSCOLORS=gxGxBxDxCxEgEdxbxgxcxd
alias ll="ls -alG"
```
Trong đó `gx` là màu mà ta quy định, chi tiết ta có thể gõ lệnh `man ls` và kéo xuống phần LSCOLORS để xem 

Thêm xong thì save file lại và khởi động lại terminal/iTerm2

Kết quả 

![image](https://hackmd.io/_uploads/S1ShN62I1e.png)

**THE END**
