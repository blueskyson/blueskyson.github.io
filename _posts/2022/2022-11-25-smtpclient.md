---
title: "用 SmtpClient 從 Outlook 傳送郵件"
subtitle: ""
excerpt: "SmtpClient Outlook .NET csharp c#"
layout: post
author: "blueskyson"
header-style: text
tags:
  - csharp
  - dotnet
---

執行環境：Windows 11、NET 6.0

首先創建一個主控台應用程式，在 Program.cs 貼上以下程式碼，並將以下字串替換：

- sender@outlook.com: 要發送信件的 Outlook 郵箱。
- 123abc!: 要發送信件的 Outlook 密碼。
- receiver@gmail.com: 收信人的郵箱。

```csharp

```csharp
using System.Net.Mail;
using System.Net;

MailMessage message = new MailMessage();
message.From = new MailAddress("sender@outlook.com");
message.To.Add(new MailAddress("receiver@gmail.com"));
message.IsBodyHtml = true;
message.Subject = "My first smtp email!";
message.Body =
@"
<!DOCTYPE html>
<html>
<body>
    <h3>Hello Jack:</h3>
    <p>This is a Hello World email!</p>
</body>
</html>
";

try
{
    var client = new SmtpClient("smtp-mail.outlook.com", 587);
    client.Credentials = new NetworkCredential("sender@outlook.com", "123abc!");
    client.EnableSsl = true;
    client.Send(message);
}
catch (Exception e)
{
    Console.WriteLine(e);
}
```

注意第一次發送時，這封信可能會被認為是垃圾郵件，所以需要手動將其移動到收件箱中。信件效果如下：

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/smtpclient.png)