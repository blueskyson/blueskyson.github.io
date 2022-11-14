---
title: "用 NUnit 和 Moq 對 ILogger 進行單元測試"
subtitle: ""
excerpt: "NUnit Moq ILogger unit test"
layout: post
author: "blueskyson"
header-style: text
tags:
  - nunit
  - csharp
  - dotnet
---

環境：.NET 6.0

首先創建一個主控台應用程式，新增一個 Foo.cs 內容如下：

```csharp
using Microsoft.Extensions.Logging;

namespace MyLog;

public class Foo
{
    private ILogger<Foo> _logger;
    public Foo(ILogger<Foo> logger)
    {
        _logger = logger;
    }

    public void PrintMessage()
    {
        _logger.LogInformation("Hello World!");
    }
}
```

接著建立一個 NUnit 專案，並新增一個 UnitTest.cs 內容如下：

```csharp
using Moq;
using Microsoft.Extensions.Logging;
using NUnit.Framework;
using MyLog;

namespace MyTest;

public class Tests
{
    [Test]
    public void Test1()
    {
        List<string> logMessages = new List<string>();
        var mockLogger = new Mock<ILogger<Foo>>();
        mockLogger.Setup(m => m.Log(
            It.IsAny<LogLevel>(),
            It.IsAny<EventId>(),
            It.IsAny<It.IsAnyType>(),
            It.IsAny<Exception>(),
            It.IsAny<Func<It.IsAnyType, Exception, string>>()!
        )).Callback(new InvocationAction(invocation => {
            var logLevel = (LogLevel)invocation.Arguments[0];
            var eventId = (EventId)invocation.Arguments[1];
            var state = invocation.Arguments[2];
            var exception = (Exception)invocation.Arguments[3];
            var formatter = invocation.Arguments[4];
                    
            var invokeMethod = formatter.GetType().GetMethod("Invoke");
            var logMessage = invokeMethod!.Invoke(formatter, new[] { state, exception });
            logMessages.Add((string)logMessage!);
        }));

        Foo foo = new Foo(mockLogger.Object);
        foo.PrintMessage();
        Assert.That("Hello World!", Is.EqualTo(logMessages[0]));
    }
}
```

然後執行測試，即可通過。