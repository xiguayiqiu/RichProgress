# RichProgress

RichProgress 是一个只使用 YScript 实现的进度显示库，不依赖 C、Go、FFI 或第三方模块。它提供多种定长进度条、终端动画、字节进度显示和多任务渲染。

## 模块结构

```text
RichProgress/
├── ysc.models
└── progress/
    ├── bar.ys       # 基础样式、自定义字符与百分比
    ├── animation.ys # spinner 与 pulse
    ├── color.ys     # ANSI 颜色、RGB 真彩色
    ├── format.ys    # 字节单位
    ├── multi.ys     # 多任务进度
    └── output.ys    # 终端动态输出
```

## 导入

在 YScript 项目中登记此库：

```sh
ysc mod get RichProgress/progress
```

在脚本中导入：

```yscript
package main
import ["RichProgress/progress"]

func main() {
    println(Bar(37, 100, 24))
}
```

项目库导入会把函数直接导入当前脚本，因此示例里直接调用 `Bar(...)`。`width` 表示条体宽度，不包括两侧边框和百分比文字。

## 进度条样式与自定义

```yscript
Bar(37, 100, 20)          // [███████░░░░░░░░░░░░░] 37%
Ascii(37, 100, 20)        // [#######-------------] 37%
Dots(37, 100, 20)         // (●●●●●●●○○○○○○○○○○○○○) 37%
Arrow(37, 100, 20)        // [======>.............] 37%
ColorBar(37, 100, 20, 32) // 绿色 ANSI 进度条
```

`CustomBar` 可自定义填充字符、空白字符和两侧边界：

```yscript
CustomBar(63, 100, 20, "▓", "·", "<", ">")
// <▓▓▓▓▓▓▓▓▓▓▓▓········> 63%
```

字符建议使用单个终端列宽的符号。`width` 指重复次数，不包含边界和百分比。

## 自定义颜色

```yscript
// ANSI 前景色、背景色和样式码；0 表示不设置该项
ColorText("完成", 32, 0, 1)                   // 绿色粗体
ColorText("警告", 33, 44, 1)                 // 黄色字、蓝色底、粗体
CustomColorBar(63, 100, 20, "▓", "·", 36, 0, 1)

// 24 位 RGB 真彩色，通道超出 0..255 时会自动夹紧
ColorRGB("自定义前景色", 40, 180, 120)
BackgroundRGB("自定义背景色", 30, 80, 160)
RGBBar(63, 100, 20, 40, 180, 120)
```

`ColorBar(current, total, width, color)` 保留简便接口，`color` 是 ANSI 前景色码，例如 `31` 红、`32` 绿、`33` 黄、`34` 蓝、`36` 青。

## 动画与动态输出

动画帧由调用者提供，适合放进已有循环或定时器中：

```yscript
println(SpinnerLine(3, "正在连接"))
println(Pulse(6, 18))
```

`PrintBar` 使用回车符覆盖当前终端行，`FinishBar` 输出最终状态并换行：

```yscript
let current = 0
while current <= 100 {
    PrintBar(current, 100, 30)
    current = current + 10
}
FinishBar(100, 100, 30)
```

## 字节与多任务

```yscript
println(ByteFormat(1572864))
println(ByteBar(1572864, 4194304, 24))

let tasks = [
    {"name": "下载", "current": 75, "total": 100},
    {"name": "校验", "current": 2, "total": 5},
]
println(Multi(tasks, 18))
```

`Percent(current, total)` 返回整数百分比。非正数 `total` 按已完成处理；进度值会限制在 `0..total` 范围内。对于未知总量的任务，使用 `Pulse(frame, width)` 或 `Spinner(frame)`。

## API

| 函数 | 作用 |
|---|---|
| `Bar(current, total, width)` | Unicode 块状进度条 |
| `Ascii(current, total, width)` | ASCII 兼容进度条 |
| `Dots(current, total, width)` | 圆点样式进度条 |
| `Arrow(current, total, width)` | 带移动指针的进度条 |
| `CustomBar(current, total, width, fill, empty, left, right)` | 自定义字符和边界 |
| `ColorBar(current, total, width, color)` | ANSI 彩色块状进度条 |
| `CustomColorBar(current, total, width, fill, empty, fg, bg, style)` | 自定义字符及 ANSI 前景、背景、样式 |
| `ColorText(text, fg, bg, style)` | 为任意文本设置 ANSI 颜色和样式 |
| `ColorRGB(text, r, g, b)` / `BackgroundRGB(text, r, g, b)` | 设置 24 位 RGB 前景色或背景色 |
| `RGBBar(current, total, width, r, g, b)` | RGB 真彩色进度条 |
| `Percent(current, total)` | 计算限制后的整数百分比 |
| `Spinner(frame)` / `SpinnerLine(frame, label)` | 旋转指示器 |
| `Pulse(frame, width)` | 往返脉冲动画 |
| `ByteFormat(size)` | 格式化为 B、KiB、MiB 或 GiB |
| `ByteBar(current, total, width)` | 进度条及已传输/总字节数 |
| `Multi(tasks, width)` | 渲染多个任务，返回换行分隔的文本 |
| `PrintBar(current, total, width)` | 回车覆盖式输出进度 |
| `FinishBar(current, total, width)` | 输出进度并换行 |
