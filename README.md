# MaaAgentCoreAndroid

在 Android 上跑 MaaFramework Python agent 所需的预构建内核

内核跟具体项目无关，只跟 `(CPython 版本, ABI, MaaFramework 版本)` 绑定

## 下载

见 [Releases](../../releases)：

| 资产 | ABI |
|---|---|
| `agent-core-3.13.15-arm64-v8a.tar.gz` | arm64-v8a |
| `agent-core-3.13.15-x86_64.tar.gz` | x86_64 |


## 许可

本仓库只发布构建产物，内含组件各自的许可随包：

| 组件 | 许可 |
|---|---|
| CPython 3.13.15 | PSF License |
| MaaFramework | LGPL-3.0 |
| numpy | BSD-3-Clause |
| StrEnum | MIT |

`maa` 的 Python 源码原样收录（仅打了一处 Android 平台名的补丁，见
[PEP 738](https://peps.python.org/pep-0738/)），未做静态链接。
