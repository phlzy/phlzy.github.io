---
title: CPP-Qt-1
date: 2026-04-18
tag: [cpp, qt]
category: [Software Engineering]
math: true
---

最近用 Qt 做了一个几年前就想做的小东西，记录一些经验

<!--more-->



1

构建之后debug，出现系统错误，找不到  Qt6Widgets.dll，Qt6Cored.dll，Qt6Sqld.dll

> - 报错中的 `d` 后缀（如 `Qt6Widgetsd.dll`）代表你当前处于 Debug 构建模式。
> - Windows 寻找 DLL 的默认顺序是：**1. 所在的同级目录** -> **2. 系统的 PATH 环境变量**。
> - 之前删除了手动拷贝 DLL 的脚本，是因为在现代 IDE 中，手动拷贝不仅拖慢编译速度，且极易漏掉底层依赖（比如 Windows 平台插件 `qwindows.dll`）。
>
> 为了优雅地解决这个问题，你有两种标准且规范的做法。作为日常开发，**强烈推荐方案一**。
>
> ------
>
> ### 方案一：在 CLion 中配置运行环境变量（最优，编译最快）
>
> 这种方法不会把几百兆的 Qt 库复制到你的构建目录中，而是让 CLion 在启动你的程序时，临时将 Qt 的 `bin` 目录注入到系统的 `PATH` 中。
>
> 1. **打开运行配置**：在 CLion 顶部导航栏，找到绿色的运行按钮左侧的下拉菜单（写着 `sword_cutlass`），点击它，选择 **Edit Configurations...（编辑配置）**。
>
> 2. **定位变量设置**：在左侧选中你的 CMake Application -> `sword_cutlass`。在右侧的面板中，找到 **Environment variables（环境变量）** 这一行。
>
> 3. **添加路径**：点击输入框最右侧的图标（类似一个小窗户），点击 `+` 号添加一个新的变量：
>
>    - **Name**: `PATH`
>
>    - **Value**: `C:\Qt\6.11.0\msvc2022_64\bin;%PATH%`
>
>      *(注意：请确保路径与你实际安装的路径完全一致，分号 `;` 和 `%PATH%` 是必须的，它表示将 Qt 路径追加到系统原有环境变量之前)*
>
> 4. **应用并运行**：点击 OK 保存。再次点击 Debug 或 Run，程序就能瞬间启动了。
>
> ------
>
> ### 方案二：使用 Qt 官方的自动化部署工具（适合想要独立运行 exe 时）
>
> 如果你希望不依赖 CLion，直接去 `cmake-build-debug` 文件夹里双击运行 `sword_cutlass.exe`，你需要将依赖拷贝过去。
>
> 不要像你之前那样手动写 `copy_if_different` 脚本。Qt 官方提供了一个专用的工具叫 **`windeployqt.exe`**，它会自动分析你的 exe 文件，精准提取出所有需要的 DLL 和底层平台插件。
>
> 你可以将以下代码追加到你的 `CMakeLists.txt` 的**最末尾**：
>
> ```CMake
> # --- 自动化部署 (windeployqt) ---
> # 仅在 Windows 平台下执行
> if (WIN32)
>     # 寻找 windeployqt 工具
>     find_program(WINDEPLOYQT_EXECUTABLE windeployqt HINTS "${CMAKE_PREFIX_PATH}/bin")
>     
>     if (WINDEPLOYQT_EXECUTABLE)
>         # 添加在构建完成后的自定义命令
>         add_custom_command(TARGET sword_cutlass POST_BUILD
>             # 执行 windeployqt 命令
>             COMMAND "${WINDEPLOYQT_EXECUTABLE}" 
>             # --no-compiler-runtime 参数表示不拷贝 C++ 基础运行时库（因为你装了 MSVC 已经有了）
>             ARGS --no-compiler-runtime "$<TARGET_FILE:sword_cutlass>"
>             COMMENT "Running windeployqt to copy required Qt DLLs..."
>         )
>     endif()
> endif()
> ```
>
> **对比说明：**
>
> - **方案一**：适合高频的代码编写与 Debug 阶段，零额外构建时间。
> - **方案二**：适合在开发告一段落，你想把 exe 发给别人，或者脱离 IDE 独立测试时使用。每次编译后会自动运行拷贝，会稍微增加几秒钟的构建耗时。







