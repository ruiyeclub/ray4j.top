---
title: 创建Telegram机器人🤖API发送请求
date: 2024-04-17 15:14:49
tags:
---
## 申请TG机器人

![image-20240417152031003](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/image-20240417152031003.png)

1. 打开 Telegram 应用程序然后搜索`@BotFather`
2. 单击开始
3. 单击菜单 -> /newbot 或输入`/newbot`并点击发送
4. 按照说明进行操作，直到我们收到这样的消息

```
Done! Congratulations on your new bot. You will find it at t.me/new_bot.
You can now add a description.....

Use this token to access the HTTP API:
63xxxxxx71:AAFoxxxxn0hwA-2TVSxxxNf4c
Keep your token secure and store it safely, it can be used by anyone to control your bot.

For a description of the Bot API, see this page: https://core.telegram.org/bots/api
```

5. 这是我们的机器人令牌`63xxxxxx71:AAFoxxxxn0hwA-2TVSxxxNf4c`（确保我们不会将其分享给任何人）。



## 获取私人聊天的聊天ID

1. 搜索并打开我们新的 Telegram 机器人

2. 单击“开始”或发送消息

3. 在浏览器中打开此网址

```
https://api.telegram.org/bot{our_bot_token}/getUpdates
```

- 请注意，我们需要在令牌前加上 `bot`
- 例如：`https://api.telegram.org/bot63xxxxxx71:AAFoxxxxn0hwA-2TVSxxxNf4c/getUpdates`

或者通过@raw_info_bot：

![image-20240417151949471](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/image-20240417151949471.png)

4. 我们会看到这样的 json

```
{
  "ok": true,
  "result": [
    {
      "update_id": 83xxxxx35,
      "message": {
        "message_id": 2643,
        "from": {...},
        "chat": {
          "id": 21xxxxx38,
          "first_name": "...",
          "last_name": "...",
          "username": "@username",
          "type": "private"
        },
        "date": 1703062972,
        "text": "/start"
      }
    }
  ]
}
```

5. 检查`result.0.message.chat.id`，这是我们的聊天 ID：`21xxxxx38`

6. 让我们尝试post发送一条消息`https://api.telegram.org/bot63xxxxxx71:AAFoxxxxn0hwA-2TVSxxxNf4c/sendMessage?chat_id=21xxxxx38&text=test123`

7. 当我们正确设置机器人令牌和聊天 ID 时，消息`test123`应该会到达我们的 Telegram 机器人聊天中。



## 获取频道的聊天 ID

获取群聊 ID 的最简单方法是通过 Telegram 桌面应用程序。

1. 在桌面应用程序中打开 Telegram

2. 将我们的 Telegram 机器人添加到聊天组中

3. 发送消息至聊天组

4. 右键单击该消息，然后单击

```
Copy Message Link
```

- 我们将得到一个像这样的链接：`https://t.me/c/194xxxx987/11/13`
- 图案：`https://t.me/c/{group_chat_id}/{group_topic_id}/{message_id}`
- 这是我们的聊天 ID：`194xxxx987`

5. 要在 API 中使用群聊 ID，我们需要在其前面加上 number 前缀`-100`，如下所示：`-100194xxxx987`

6. 现在让我们尝试发送消息`https://api.telegram.org/bot63xxxxxx71:AAFoxxxxn0hwA-2TVSxxxNf4c/sendMessage?chat_id=-100194xxxx987&text=test123`

或者使用python：

```python
import requests

response = requests.post(
    url='https://api.telegram.org/bot{0}/{1}'.format(token, method),
    data={'chat_id': 12345, 'text': 'hello friend'}
).json()
```

7. 当我们正确设置机器人令牌和聊天 ID 时，消息`test123`应该会到达我们的群聊中。



## 获取群聊中某个主题的聊天 ID

为了向 Telegram 群组上的特定主题发送消息，我们需要获取主题 ID。

1. 与上面的步骤类似，我们点击 后`Copy Message Link`，会得到一个类似： 的链接`https://t.me/c/194xxxx987/11/13`，所以群组 Topic ID 为`11`。

2. 现在我们可以像这样使用它（参见`message_thread_id`）：`https://api.telegram.org/bot783114779:AAEuRWDTFD2UQ7agBtFSuhJf2-NmvHN3OPc/sendMessage?chat_id=-100194xxxx987&message_thread_id=11&text=test123`

3. 当我们正确设置机器人令牌和聊天 ID 时，消息`test123`应该到达我们的群聊主题内。



参考链接：https://stackoverflow.com/questions/31197659/how-to-send-request-to-telegram-bot-api![](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/image-20240417151949471.png)



## 更新内容

将消息推送至群组

1. 将机器人拉入群组，发送`/{message}@bot`消息
2. 可以通过`https://api.telegram.org/bot{token}/getUpdates`获取群组id

![image-20240829100959057](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/image-20240829100959057.png)

这里群组id为-4583**9819