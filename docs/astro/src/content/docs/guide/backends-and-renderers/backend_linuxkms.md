---
<!-- Copyright © SixtyFPS GmbH <info@slint.dev> ; SPDX-License-Identifier: MIT -->
title: LinuxKMS Backend
description: LinuxKMS Backend
---

<!-- cSpell: ignore linuxkms libinput libseat libudev libgbm libxkbcommon xkbcommon noseat -->

LinuxKMS后端仅在Linux上运行，无需依赖Wayland或X11等窗口系统。它通过以下库和接口直接渲染到屏幕，并响应触摸、鼠标和键盘输入。

- 通过KMS/DRI实现OpenGL。
- 通过Vulkan KHR显示扩展实现Vulkan。
- 使用DRM dumb缓冲区进行软件渲染。
- 使用libinput/libudev处理来自鼠标、触摸屏或键盘的输入事件。
- 使用libseat无需root权限即可访问GPU和输入设备。（可选）

在编译过程中，使用 pkg-config 来确定以下必需的系统库的位置：

| pkg-config 包名称 | 基于 Debian 的发行版中的包名 |
|-------------------------|--------------------------------------|
| `gbm`                   | `libgbm-dev`                         |
| `xkbcommon`             | `libxkbcommon-dev`                   |
| `libudev`               | `libudev-dev`                        |
| `libseat`               | `libseat-dev`                        |

:::note{Note}
如果目标系统上没有 `libseat`，则不要选择 `backend-linuxkms`，而是选择 `backend-linuxkms-noseat`。此版本的 LinuxKMS 后端无需安装 libseat，但作为交换，需要以具有访问所有输入和 DRM/KMS 设备文件权限的用户身份运行应用程序；通常这是 root 用户。
:::

LinuxKMS 后端支持不同的渲染器。可以通过 `SLINT_BACKEND` 环境变量显式选择使用的渲染器。

| 渲染器名称 | 所需的图形 API | 选择渲染器的 `SLINT_BACKEND` 值 |
|---------------|------------------------|-----------------------------------------------------------------------------|
| FemtoVG       | OpenGL ES 2.0          | `linuxkms-femtovg`                                                          |
| Skia          | OpenGL ES 2.0, Vulkan  | `linuxkms-skia-opengl`, `linuxkms-skia-vulkan`, 或 `linuxkms-skia-software` |
| Software      | 无                     | `linuxkms-software`                                                         |

:::note{注意}
该后端仍处于实验阶段。后端尚未在多种设备上进行广泛测试，且存在[已知问题](https://github.com/slint-ui/slint/labels/a%3Abackend-linuxkms)。
:::

:::note{注意}
支持鼠标作为输入设备，但鼠标光标的渲染仅适用于 Skia 和 FemtoVG 渲染器，不适用于 Slint 软件渲染器。
:::

## 使用 OpenGL 或 Skia 软件选择显示

FemtoVG 使用 OpenGL，而 Skia（除非启用 Vulkan）也使用 OpenGL。Linux 的直接渲染管理器（DRM）子系统用于配置显示输出。Slint 默认选择第一个连接的显示器，并将其配置为其首选分辨率（如果可用）或最高分辨率。设置 `SLINT_DRM_OUTPUT` 环境变量以选择特定的显示器。要获取可用输出的列表，将 `SLINT_DRM_OUTPUT` 设置为 `list`，运行程序并观察输出。

例如，在带有内置屏幕（eDP-1）和外部连接显示器（DP-3）的笔记本电脑上，输出可能如下所示：

```
DRM 输出列表请求：
eDP-1 (已连接: true)
DP-1 (已连接: false)
DP-2 (已连接: false)
DP-3 (已连接: true)
DP-4 (已连接: false)
```

将 `SLINT_DRM_OUTPUT` 设置为 `DP-3` 将在第二个显示器上渲染。

要选择特定的分辨率和刷新率（模式），设置 `SLINT_DRM_MODE` 变量。将其设置为 `list` 并运行程序以获取可用模式的列表。例如，程序输出可能如下所示：

```
DRM 模式列表请求：
索引: 0 宽度: 3840 高度: 2160 刷新率: 60
索引: 1 宽度: 3840 高度: 2160 刷新率: 50
索引: 2 宽度: 3840 高度: 2160 刷新率: 30
索引: 3 宽度: 2560 高度: 1440 刷新率: 59
索引: 4 宽度: 1920 高度: 1080 刷新率: 60
索引: 5 宽度: 1680 高度: 1050 刷新率: 59
...
```

将 `SLINT_DRM_MODE` 设置为 `4` 以选择 1920x1080@60。

## 使用 Vulkan 选择显示

当启用 Skia 的 Vulkan 功能时，Skia 将尝试使用 Vulkan 的 KHR 显示扩展直接渲染到连接的屏幕。Slint 默认选择第一个连接的显示器，并将其配置为最高可用分辨率和刷新率。设置 `SLINT_VULKAN_DISPLAY` 环境变量以选择特定的显示器。要获取可用输出的列表，将 `SLINT_VULKAN_DISPLAY` 设置为 `list`，运行程序并观察输出。

例如，在带有内置屏幕（索引 0）和外部连接显示器（索引 1）的笔记本电脑上，输出可能如下所示：

```
Vulkan 显示列表请求：
索引: 0 名称: 显示器
索引: 1 名称: 显示器
```

将 `SLINT_VULKAN_DISPLAY` 设置为 `1` 将在第二个显示器上渲染。

要选择特定的分辨率和刷新率（模式），设置 `SLINT_VULKAN_MODE` 变量。将其设置为 `list` 并运行程序以获取可用模式的列表。例如，程序输出可能如下所示：

```
Vulkan 模式列表请求：
索引: 0 宽度: 3840 高度: 2160 刷新率: 60
索引: 1 宽度: 3840 高度: 2160 刷新率: 50
索引: 2 宽度: 3840 高度: 2160 刷新率: 30
索引: 3 宽度: 2560 高度: 1440 刷新率: 59
索引: 4 宽度: 1920 高度: 1080 刷新率: 60
索引: 5 宽度: 1680 高度: 1050 刷新率: 59
...
```

将 `SLINT_VULKAN_MODE` 设置为 `4` 以选择 1920x1080@60。

## 配置键盘

默认情况下，键盘布局和型号假定为美国型号和布局。设置以下环境变量以配置对不同键盘的支持：

* `XKB_DEFAULT_LAYOUT`：以逗号分隔的布局（语言）列表，包含在键映射中。有关接受的语言代码列表，请参阅 [xkeyboard-config(7)](https://manpages.debian.org/testing/xkb-data/xkeyboard-config.7.en.html) 中的布局部分。
* `XKB_DEFAULT_MODEL`：用于解释键的键盘型号。有关接受的型号代码列表，请参阅 [xkeyboard-config(7)](https://manpages.debian.org/testing/xkb-data/xkeyboard-config.7.en.html) 中的型号部分。
* `XKB_DEFAULT_VARIANT`：以逗号分隔的变体列表，每个布局一个，用于配置布局特定的变体。有关接受的变体代码列表，请参阅 [xkeyboard-config(7)](https://manpages.debian.org/testing/xkb-data/xkeyboard-config.7.en.html) 中布局部分括号内的值。
* `XKB_DEFAULT_OPTIONS`：以逗号分隔的选项列表，用于配置与布局无关的键组合。有关接受的选项代码列表，请参阅 [xkeyboard-config(7)](https://manpages.debian.org/testing/xkb-data/xkeyboard-config.7.en.html) 中的选项部分。

## 显示旋转

如果显示器的默认方向与用户界面的期望方向不匹配，则可以设置 `SLINT_KMS_ROTATION` 环境变量，指示 Slint 在渲染时进行旋转。支持的值是旋转角度：`0`、`90`、`180` 和 `270`。

请注意，此变量仅旋转渲染输出。如果您使用的是连接到同一显示器的触摸屏，则可能需要配置它，以对生成的触摸事件也应用旋转。有关配置 libinput 的 `LIBINPUT_CALIBRATION_MATRIX`，请参阅 [libinput 文档](https://wayland.freedesktop.org/libinput/doc/latest/device-configuration-via-udev.html#static-device-configuration-via-udev) 以获取有效值列表。通常可以通过将这些值写入 `/etc/udev/rules.d` 下的规则文件来设置。

以下示例配置 libinput 为任何连接的触摸屏应用 90 度顺时针旋转：

```bash
echo 'ENV{LIBINPUT_CALIBRATION_MATRIX}="0 -1 1 1 0 0"' > /etc/udev/rules.d/libinput.rules
udevadm control --reload-rules
udevadm trigger
```


