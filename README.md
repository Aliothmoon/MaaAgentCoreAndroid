# MaaAgentCoreAndroid

在 Android 上跑 MaaFramework Python agent 所需的预构建内核。

内核跟具体项目无关，只跟 `(CPython 版本, ABI, MaaFramework 版本)` 绑定。
构建它要 Android NDK；下载现成的则不用，接项目的人只需要 pip。

## 下载

见 [Releases](../../releases)：

| 资产 | ABI |
|---|---|
| `agent-core-3.13.15-arm64-v8a.tar.gz` | arm64-v8a |
| `agent-core-3.13.15-x86_64.tar.gz` | x86_64 |

## 内含

| | |
|---|---|
| CPython | 3.13.15（Chaquopy 的 Android 构建） |
| `bin/python3` | 单独编的可执行体——上游只发 `libpython3.13.so` |
| stdlib | 收进 `prefix/lib/python313.zip` 交 zipimport，`lib-dynload` 留磁盘 |
| MaaFramework | `maa` 5.12.3（只取 Python 源码，`maa/bin` 的桌面 native 栈已剔除） |
| 其它 | numpy 2.3.2、StrEnum 0.4.15 |
| `_multiprocessing.py` | 补丁：Android 的 bionic 没有 POSIX 命名信号量，CPython 不编译这个扩展 |

`site-packages` **不打包**，散装留着，等叠完项目自己的依赖再统一收进 `pure.zip`。

解开后的布局：

```
<abi>/bundle/
  bin/python3
  prefix/                 lib/libpython3.13.so、lib/python313.zip、lib/python3.13/lib-dynload
  site-packages/          maa/、numpy/、numpy.libs/
  agent-core.json         装了什么、按哪套 wheel tag 装的
```

## 拉起时的环境变量

`maa` 靠 ctypes 加载 MaaFramework，stdlib 的扩展模块没有 RUNPATH，两边都得靠
`LD_LIBRARY_PATH` 找：

```
PYTHONHOME      <bundle>/prefix
PYTHONPATH      <bundle>/site-packages/pure.zip:<bundle>/site-packages
LD_LIBRARY_PATH <bundle>/prefix/lib:<MaaFramework .so 所在目录>
MAAFW_BINARY_PATH  <MaaFramework .so 所在目录>
```

叠依赖时若装进了 Chaquopy 的 native 包（Pillow 这类），它们的 `.so` 落在
`site-packages/chaquopy/lib` 且**不带 RUNPATH**，这一段也要进 `LD_LIBRARY_PATH`。

命令行末位必须是 MaaFramework AgentClient 给的 identifier，入口脚本从 `sys.argv[-1]` 取。

## 已知边界

- 锁 CPython 3.13：numpy 的 Android wheel 只有 cp313
- wheel 只在 Chaquopy 的索引上，PyPI 本体基本没有 Android 轮子；
  带 native 的包（Pillow 等）在 `https://chaquo.com/pypi-13.1/`，版本落后 PyPI 不少
- `agent/` 的入口脚本不在内核里——那是项目自己的载荷

## 许可

本仓库只发布构建产物，内含组件各自的许可随包：

| 组件 | 许可 |
|---|---|
| CPython 3.13.15 | PSF License |
| MaaFramework（`maa` 包） | LGPL-3.0 |
| numpy | BSD-3-Clause |
| StrEnum | MIT |

`maa` 的 Python 源码原样收录（仅打了一处 Android 平台名的补丁，见
[PEP 738](https://peps.python.org/pep-0738/)），未做静态链接。
