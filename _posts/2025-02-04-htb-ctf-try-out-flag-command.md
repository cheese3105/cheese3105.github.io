---
title: HTB - CTF Try Out - Flag Command
tags: write-up HTB
---
# Mô tả
> Embark on the "Dimensional Escape Quest" where you wake up in a mysterious forest maze that's not quite of this world. Navigate singing squirrels, mischievous nymphs, and grumpy wizards in a whimsical labyrinth that may lead to otherworldly surprises. Will you conquer the enchanted maze or find yourself lost in a different dimension of magical challenges? The journey unfolds in this mystical escape!

# Quan sát 
Sau quan sát ta sẽ thấy đây là một web application giả lập một game với 4 lựa chọn. Mỗi lựa chọn sẽ đưa tới một diễn biến tiếp theo

![image](https://hackmd.io/_uploads/HJ6iFlytJx.png)

Sử dụng chức năng inspect của trình duyệt ta tìm thấy file **main.js** và có đoạn code sau:

```javascript
async function CheckMessage() {
    fetchingResponse = true;
    currentCommand = commandHistory[commandHistory.length - 1];

    if (availableOptions[currentStep].includes(currentCommand) || availableOptions['secret'].includes(currentCommand)) {
        await fetch('/api/monitor', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({ 'command': currentCommand })
        })
            .then((res) => res.json())
            .then(async (data) => {
                console.log(data)
                await displayLineInTerminal({ text: data.message });

                if(data.message.includes('Game over')) {
                    playerLost();
                    fetchingResponse = false;
                    return;
                }

                if(data.message.includes('HTB{')) {
                    playerWon();
                    fetchingResponse = false;

                    return;
                }

                if (currentCommand == 'HEAD NORTH') {
                    currentStep = '2';
                }
                else if (currentCommand == 'FOLLOW A MYSTERIOUS PATH') {
                    currentStep = '3'
                }
                else if (currentCommand == 'SET UP CAMP') {
                    currentStep = '4'
                }

                let lineBreak = document.createElement("br");


                beforeDiv.parentNode.insertBefore(lineBreak, beforeDiv);
                displayLineInTerminal({ text: '<span class="command">You have 4 options!</span>' })
                displayLinesInTerminal({ lines: availableOptions[currentStep] })
                fetchingResponse = false;
            });


    }
    else {
        displayLineInTerminal({ text: "You do realise its not a park where you can just play around and move around pick from options how are hard it is for you????" });
        fetchingResponse = false;
    }
}
```

# Cách giải

Thông qua source main.js ta biết được:
- Ứng dụng sẽ call đến API `/api/option` và gán giá trị JSON lấy được vào `availableOptions`
- Server sẽ check nếu command được nhập tồn tại trong `availableOptions` thì mới tiến hành gửi POST request tới server, lấy về `data.message`
- Để ý thấy có option `secret` nhưng không được thấy hiện trên giao diện

Như vậy, ta sẽ thử dùng Burp để bắt request response, hoặc truy cập vào tab Network của developer tool trên trình duyệt. Đọc được value của option `secret`. Gửi value dưới dạng command. Và thế là lấy được flag

![image](https://hackmd.io/_uploads/SJONKDyYyg.png)

**HẾT**
