---
title: "C# 動態折線圖"
subtitle: ""
excerpt: "csharp c# dynamic line chart 動態 折線圖"
layout: post
author: "blueskyson"
header-style: text
mathjax: true
tags:
  - csharp
---

劃出類似 CPU 使用率的動態折線圖

![](https://raw.githubusercontent.com/blueskyson/image-host/master/csharplinechart.png)

|Form1 成員|用途|
|--------|---|
|Chart chart   |折線圖物件|
|int[] xvalue  |折線的 x 座標陣列|
|int[] yvalue  |折線的 y 座標陣列|
|int x_tick    |一次顯示多少座標點|

|Form1 函式|用途|
|----------|---|
|Form1()   |開啟視窗|
|Chart MyChart()|生成一個自訂的圖表|
|void InitXY()|把折線初始化為 0|
|Series getSeries()|把 `xvalue`, `yvalue` 轉換為 Series 物件|
|void timer_Tick(object sender, EventArgs e)|在 Timer 觸發時更新折線圖

創建一個名為 LineChart 的 .Net Framwork 專案，然後將以下程式碼貼到 `From1.cs` 即可。

```csharp
using System;
using System.Drawing;
using System.Windows.Forms;
using System.Windows.Forms.DataVisualization.Charting;

namespace LineChart
{
    public partial class Form1 : Form
    {
        Chart chart;
        int[] xvalue, yvalue;
        int x_tick;
        public Form1()
        {
            InitializeComponent();
            
            // Initialize a line chart
            chart = MyChart(600, 400, 0);
            InitXY();
            chart.Series.Add(getSeries());
            Controls.Add(chart);

            // Update when timer ticks
            Timer timer = new Timer();
            timer.Interval = 1000;      // 1 second
            timer.Tick += new EventHandler(timer_Tick);
            timer.Start();
        }

        public Chart MyChart(int width, int height, int value)
        {
            Chart chart = new Chart();
            chart.Width = width;
            chart.Height = height;
            chart.Location = new Point(0, 0);
            chart.Visible = true;
            
            ChartArea ca = new ChartArea("My Chart");
            ca.BackColor = Color.Azure;
            ca.BackGradientStyle = GradientStyle.HorizontalCenter;
            ca.BackHatchStyle = ChartHatchStyle.LargeGrid;
            ca.ShadowColor = Color.Purple;
            ca.ShadowOffset = 0;

            ca.AxisY.Enabled = AxisEnabled.True;
            ca.AxisY.LineColor = Color.LightBlue;
            ca.AxisY2.LabelStyle.Enabled = true;
            ca.AxisY2.LineColor = Color.LightBlue;
            ca.AxisY.Minimum = 0;                                   //Y axis Minimum value

            ca.AxisX.Title = value + " unit / sec";
            ca.AxisX.Interval = 5;
            ca.AxisX.LabelAutoFitMinFontSize = 5;
            ca.AxisX.LabelStyle.Angle = -20;
            ca.AxisX.LabelStyle.IsEndLabelVisible = true;           //show the last label
            ca.AxisX.IntervalAutoMode = IntervalAutoMode.FixedCount;
            ca.AxisX.IntervalType = DateTimeIntervalType.NotSet;
            ca.AxisX.TextOrientation = TextOrientation.Auto;
            ca.AxisX.LineWidth = 1;
            ca.AxisX.LineColor = Color.LightBlue;
            ca.AxisX.Enabled = AxisEnabled.True;
            ca.AxisX.ScaleView.MinSizeType = DateTimeIntervalType.Seconds;
            ca.AxisX.ScrollBar = new AxisScrollBar();

            chart.ChartAreas.Add(ca);
            chart.ChartAreas[0].Axes[0].MajorGrid.Enabled = false;  //x axis
            chart.ChartAreas[0].Axes[1].MajorGrid.Enabled = false;  //y axis
            chart.ChartAreas[0].AxisX.IsMarginVisible = false;
            return chart;
        }

        public void InitXY()
        {
            x_tick = 20;
            xvalue = new int[x_tick];
            yvalue = new int[x_tick];
            for (int i = 0; i < x_tick; i++)
            {
                xvalue[i] = i;
                yvalue[i] = 0;
            }
        }

        public Series getSeries()
        {
            Series series = new Series();
            series.ChartType = SeriesChartType.Line;
            series.BorderWidth = 2;
            series.Color = Color.OrangeRed;
            series.XValueType = ChartValueType.Int32;
            series.YValueType = ChartValueType.Int32;
            series.IsValueShownAsLabel = false;
            
            for (int i = 0; i < x_tick; i++) {
                series.Points.AddXY(xvalue[i], yvalue[i]);
            }

            return series;
        }

        public void timer_Tick(object sender, EventArgs e)
        {
            Random rand = new Random();
            for (int i = 1; i < x_tick; i++) {
                xvalue[i - 1] = xvalue[i];
                yvalue[i - 1] = yvalue[i];
            }

            int current_yvalue = rand.Next(100);
            xvalue[x_tick - 1]++;
            yvalue[x_tick - 1] = current_yvalue;

            chart.Visible = false;
            Controls.Remove(chart);
            chart = MyChart(600, 400, current_yvalue);
            chart.Series.Add(getSeries());
            this.Controls.Add(chart);
        }
    }
}
```