---
title: "敏捷開發與 SOLID 原則"
subtitle: ""
excerpt: "敏捷開發 SOLID SRP OCP LSP DIP ISP"
layout: post
author: "blueskyson"
header-style: text
tags:
  - software engineering
---

## 敏捷開發

敏捷開發是一種用來應對快速變化的需求的開發方式，其強調軟體工程人員與業務人員面對面的溝通、頻繁交付可用的版本，使專案有彈性以因應不可預期的變化。常常聽到的 Scrum 與 Kanban 就是用來實敏節開發的框架。

敏捷開發最重要的四個方針：

- 個人與互動：重於流程與工具
- 可用的軟體：重於詳盡的文件
- 與客戶合作：重於合約協商
- 回應變化：重於遵循計劃

## Single-Responsibility Principle (SRP) 單一職責原則

**一個類別應只負責一個職責。**

如果一個類別承擔過多職責 (高耦合)，當需要修改或新增職責時，容易使得所有依賴此職責的地方都需要被修改，提高維護難度。透過分離職責來降低耦合，當需要新增、更改職責時，只需依照 interface 的定義來修改。

下方舉一個例子，首先定義一個訂單的資料結構，它可以透過影印機輸出。

```csharp
public class Order
{
    private string Id;
    private double Price;

    public void Print()
    {
        ...
    }
}
```

分析 Order 這個類別，它應該只專注處理地單的資訊，`Print()` 顯得超出它的職責了，假設今天有了新需求：透過 PDF 輸出，此時 `Order` 類別會變成：

```csharp
public class Order
{
    private string Id;
    private double Price;

    public void Print()
    {
        ...
    }

    public void PrintByPdfExporter()
    {
        ...
    }
}
```

這時所有呼叫 `PrintByPrinter` 的地方都必須跟著改寫成相容於 `PrintByPdfExporter` 的邏輯，有許多處程式都必須修正。

現在套用 SRP 原則，定義一個 interface 叫 IPrinter：

```csharp
public interface IPrinter
{
    void Print(string id, double price);
}

public class PdfExporter : IPrinter
{
    public void Print(string id, double price)
    {
        ...
    }
}

public class Printer : IPrinter
{
    public void Print(string id, double price)
    {
        ...
    }
}

public class Order
{
    private string Id;
    private double Price;
    private IPrinter Printer;

    public Order()
    {
        this.Printer = new PdfExporter();
        // this.Printer = new Printer();
    }

    public Print()
    {
        this.Printer.Print(this.Id, this.Price);
    }
}
```

如此一來，所有跟列印訂單有關的更改就限縮在個別的 IPrinter 實作中。

## Open-Closed Principle (OCP) 開放封閉原則

**對於擴展是開放的，對於修改是封閉的。**

在下方範例中，`DrawAllShapes` 根據 `ShapeType` 來決定要畫出什麼圖案。每擴展一種形狀，就要修改一次 `DrawAllShape` 的 `switch` 區塊，增加了維護程式碼的複雜度。

```csharp
enum ShapeType { circle, square };

struct Shape
{
    ShapeType Type;
} 

void DrawAllShapes(Shape* Shapes, int n)
{
    for (int i = 0; i < n; i++) {
        switch(Shapes[i].Type) {
        case square:
            DrawSquare(Shapes[i]);
            break;
        case circle:
            DrawCircle(Shapes[i]);
            break;
        }
    }
}
```

在下方範例，每擴展一種形狀，則完全不用更改 `DrawAllShape`，故符合 OCP 原則。

```csharp
Interface Shape
{
    void Draw();
} 

class Square : Shape
{
    void Draw() { /* draw square here */ }
}

class Circle : Shape
{
    void Draw() { /* draw circle here */ }
}

void DrawAllShapes(Shape* Shapes, int n)
{
    for (int i = 0; i < n; i++)
    {
        Shapes[i].Draw();
    }
}
```

## Liskov Substitution Principle (LSP) Liskov 替換原則

**子類別應該完全符合父類別的特性。**

在下方範例中，儘管邏輯上正方形可以被視為長方形的特例，但是在程式實作中，如果意外將正方形的物件視為長方形來使用，就有許多衍伸問題，例如正方形的 Width 和 Height 被誤設為不同的值。

```csharp
public class Rectangle
{
    protected int height;
    protected int width;

    public virtual int Height
    {
        get { return height; }
        set { height = value; }
    }

    public virtual int Width
    {
        get { return width; }
        set { width = value; }
    }
    
    public int Area
    {
        get { return width * height; }
    }
}

public class Square : Rectangle {
    public override int Height
    {
        get { return height; }
        set { width = height = value; }
    }

    public override int Width
    {
        get { return width; }
        set { width = height = value; }
    }
}
```

套用上面的 `Rectangle` 與 `Square`，以下為一個違反 LSP 而造成 Bug 的案例：

```csharp
// Program.cs
void stretch(Rectangle rect, int hValue, int vValue)
{
    rect.Width += hValue;
    rect.Height += vValue;
}

Rectangle rectangle = new Rectangle { Width = 5, Height = 5 };
Square square = new Square { Width = 5 };

Console.WriteLine(rectangle.Area);
Console.WriteLine(square.Area);
Console.WriteLine("---");
stretch(rectangle, 1, 1);
stretch(square, 1, 1);
Console.WriteLine(rectangle.Area);
Console.WriteLine(square.Area);
```

輸出為：

```non
25
25
---
36
49
```

很明顯的 `square` 的長寬增加 1 後，面積應該要是 36，卻被誤算為 49。

> *待續*
> ## Dependency-Inversion Principle (DIP) 依賴反向原則
> 
> **1. 高層模組不應該直接依賴低層模組，它們都應該依賴抽象層。**
> 
> **2. 抽象層不應該因為低層模組而更改，反而低層模組要按照抽象層來實作。**
> 
> 右圖一，高層的 Policy 依賴 Mechanism，Mechanism 依賴 Utility，因此 Policy 遞移性地依賴 Utility。如此設計的缺點是，當低層模組變動時，會直接影響所有遞移性依賴的高層模組，使高層模組很可能也要跟著變動。
> 右圖二，所有模組都反過來依賴一個抽象介面，低層模組實作介面、高層模組透過介面使用低層模組。
> 
> ## Interface Segregation Principle (ISP) 介面隔離原則
> 
> **不應強迫子類別去依賴它不使用的方法。**
> 
> 下左圖 IPets 的範例為了兼容 Dog 和 Cat 類別，所以定義了 bark() 方法，導致 Cat 類別必須也提供一個 bark() 的實作，導致了不必要的耦合。
> 下右圖把 IPetDogs 和 IPetCats 隔離出來，讓 Dog 與 Cat 各自實作 bark() 和 meow()。