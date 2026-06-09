# rime-llm-translator

给Rime 输入法接入大模型进行拼音联想，支持TUI 图形化配置。平时正常输入，在遇到长难句或者生僻句的时候通过双击 v 键呼叫 AI 对拼音进行处理。输入的时候不太需要考虑输入的拼音是否正确，大模型强大的预测功能会自动把误拼甚至乱序处理成正确的句子。对于句子中的英文词汇也能正常处理，甚至还能补充正确的空格和标点符号。

>甚至还能用输入法跟 AI 聊天。

以下是无法正常输入，但AI 可以联想出来的例子：

包含英文词汇的输入

![](pictures/usevibecoding.gif)

歌词

![](pictures/cnag.gif)

古诗词

![](pictures/shici.gif)

超长句

![](pictures/steam.gif)

甚至可以在输入法给 AI 指令：

![](pictures/call.gif)

缺点是大模型出token 需要时间，导致联想有一点延迟，尤其在使用推理模型的时候延迟尤为明显。使用快速响应的模型或者本地部署小模型可以解决延迟问题，但是效果就一般了。我目前使用`DeepSeek V4 Pro 关闭思考`和`Gemini 3 Flash Preview 最少思考`，效果很棒。

>也许可以使用特调小模型。

## 使用方法

1. 安装 Rime 和雾凇拼音

    > 对其他基于 Rime 的输入法方案也是生效的，例如万象拼音，但是要自己手动配置。

    输入法安装方法可以看[ShorinArch_中文输入法](https://github.com/SHORiN-KiWATA/Shorin-ArchLinux-Guide/wiki/%E4%B8%AD%E6%96%87%E8%BE%93%E5%85%A5%E6%B3%95)

2. 安装 `rime-llm-translator-git`

    ```
    yay -S rime-llm-translator-git
    ```

    此时会安装三个文件：

    - `/usr/share/rime-llm-translator/default_state.json` 默认配置文件。用户空间的配置文件位于`~/.config/rime-llm-translator/state.json`。

    - `/usr/share/rime-data/lua/llm_translator.lua` 功能主体。

    - `/usr/bin/rime-llm-config` 管理工具，支持TUI编辑配置文件。

3. 运行 `rime-llm-config init` 初始化

    初始化完成后会有使用教程提示。

    这一步会自动在 `~/.local/fcitx5/rime` 编辑配置文件：

    > 配置文件在修改前会备份至`~/.cache/rime-llm-translator-backup`
    
    - 新建 `rime.lua`导入`llm_translator`；
    
      ```
      llm_translator = require("llm_translator")
      ```
      > 如果已经存在的话会备份后在文件末尾追加
    
    - 如果检测到安装了雾凇拼音的话会新建 `rime_ice.custom.yaml`写入patch，启用llm_translator并配置一些上屏规则；

      ```
      patch:
        # 1. 扩充允许输入的字符集：允许在拼音中直接输入指定的标点符号，阻止其直接上屏
        "speller/alphabet": "zyxwvutsrqponmlkjihgfedcba.,?'!:<>"
        # 2. 将 Lua AI 脚本 (llm_translator) 强行插入到处理列表的第 0 位之前
        "engine/translators/@before 0": lua_translator@llm_translator
        # 3. 定义正则捕获规则：把输入当成不可分割的整体喂给 AI 脚本处理
        "recognizer/patterns/llm_pinyin": "^[a-z][a-z.,?'!:]*$"
      ```

    - 如果 Fcitx5 正在运行的话，重启以重新部署。
    
4. 配置大模型

    运行`rime-llm-config`命令进行模型配置。自带了一个硅基流动的 GLM-4 试用，效果很差，算是体验一下hhh

## 编辑配置

`rime-llm-config` 是编辑配置的 TUI 工具。

> 如果你要手动编辑配置文件请编辑 `~/.config/rime-llm-translator/state.json` 后用 `rime-llm-config sync`命令同步至 `config.lua`


![](pictures/TUI/mainmenu.png)

- 激活配置

  此处可以设置具体使用哪一个配置

  ![](pictures/TUI/active.png)

- 供应商和模型

  ![](pictures/TUI/edit.png)

  最左侧一列是供应商（同时也是配置），回车可以配置供应商的显示名称、API 地址、API 密钥等内容，部分模型支持开关思考模式。

  > 显示名称指的是在输入法候选框里显示的名称

  ![](pictures/TUI/provider.png)

  配置可用之后右侧会出现`可用模型`列表，回车确定此配置使用的模型。

- 全局参数配置

  ![](pictures/TUI/prompt2.png)

  这里可以对系统提示词和模型参数进行配置。`历史上下文容量`指的是记录多少之前输入过的内容，用于提高 AI 的联想质量。

- 自定义词库

  ![](pictures/TUI/vocab.png)

  这里可以自定义词库。`常用英文词`是为了避免 AI 把句子中的英文视为拼音进行分词；`拼音缩写映射`可以提高首字母缩写、简拼的联想质量。

  ![](pictures/TUI/vocab3.png)
