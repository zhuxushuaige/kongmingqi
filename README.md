# 孔明棋求解器

命令式计算原理课程设计：使用 C0 实现单人孔明棋求解，比较直接搜索、回溯搜索和使用哈希表记录棋盘状态的实现。

## 游戏规则

棋子沿水平或竖直方向跳过相邻棋子，落到其后的空位，被跳过的棋子随之移除。目标是使棋盘最终只剩一枚棋子。程序找到解后，会逐步输出移动坐标，并从初始棋盘重新验证每一步及最终棋子数。

棋盘使用 8×8 数组表示，文本文件中的符号含义如下：

- `*`：棋子
- `o`：空位
- `.`：不可使用的位置

各位置按行读取，空格和换行用于分隔。棋盘坐标从 0 开始，输出格式为 `起始行:起始列 -> 目标行:目标列`。

## 求解版本

| 文件 | 方法 | 说明 |
| --- | --- | --- |
| `peg1.c0` | 不回溯的搜索 | 每一步选择一个合法移动继续，遇到死路便停止；找不到解不代表棋盘无解。 |
| `peg2.c0` | 深度优先搜索与回溯 | 依次尝试合法移动，失败时恢复棋盘并尝试其他分支。 |
| `peg3.c0` | 回溯与哈希表 | 记录已搜索过的棋盘状态，减少重复搜索。 |
| `peg3_ordered.c0` | 调整移动顺序的哈希表版本 | 调整候选移动次序，并输出记录的不可解状态数量。 |

搜索耗时受棋盘、候选移动顺序和哈希函数影响较大。当前实现仍有可改进之处：移动生成的边界检查、哈希状态编码与碰撞分布值得进一步测试；不要将现有程序视为已验证适用于任意棋盘的完整求解器。

## 环境与运行

需要**C0 编译器 `cc0`** 及其标准库。以下命令沿用项目原有说明，适用于支持通配符的类 Unix shell；在 Windows 上可以使用配置好 C0 的 Linux / WSL 环境。先进入本目录，再分别编译需要的版本：

```bash
# 不回溯版本
cc0 -d -w -o peg1 peg-client.c0 lib/*.c0 peg1.c0 peg-main.c0
./peg1 german.txt

# 回溯版本
cc0 -d -w -o peg2 peg-client.c0 lib/*.c0 peg2.c0 peg-main.c0
./peg2 german.txt

# 哈希表版本
cc0 -d -w -o peg3 peg-client.c0 lib/*.c0 peg3.c0 peg-main.c0
./peg3 german.txt

# 调整移动顺序的版本
cc0 -d -w -o peg3_order peg-client.c0 lib/*.c0 peg3_ordered.c0 peg-main.c0
./peg3_order german.txt
```

一次编译只选择一个 `peg*.c0` 求解器文件，因为这些文件定义了相同的接口。将 `german.txt` 替换为其他棋盘文件即可尝试不同初始状态。也可以不传文件参数，使用程序内置的 English 棋盘，例如 `./peg3`。

成功找到并验证解时，程序输出 `Solution verified!`；如果验证失败则输出 `Solution check failed!`；未找到解时会输出相应提示。复杂棋盘的搜索可能需要较长时间和较多内存。

项目原说明还提供了性能优化编译选项，例如：

```bash
cc0 -w -o peg3 -r unsafe -c-O2 peg-client.c0 lib/*.c0 peg3.c0 peg-main.c0
./peg3 english.txt
```

`unsafe` 会改变运行时检查行为；调试时建议先使用上面的 `-d` 编译方式。不同环境对编译选项的支持以本地 `cc0` 为准。

## 示例棋盘

- `german.txt`：较简单的棋盘，适合初步运行。
- `english.txt`：标准 English 棋盘。
- `french1.txt`、`french2.txt`、`french3.txt`：较困难的棋盘。

也可以按照上述符号和 8×8 布局创建自己的棋盘文件，并将文件名作为程序参数。

## 项目结构

```text
src/
├── peg-main.c0              # 读取棋盘、调用求解器并验证解
├── peg-client.c0            # 栈元素及哈希表键、哈希函数
├── peg1.c0                  # 不回溯版本
├── peg2.c0                  # 回溯版本
├── peg3.c0                  # 回溯与哈希表版本
├── peg3_ordered.c0          # 调整搜索顺序的版本
├── pegmark.c0               # 哈希表性能测试框架
├── lib/
│   ├── peg-util.c0          # 棋盘读取、输出和基本操作
│   ├── stacks.c0            # 栈
│   └── ht.c0                # 哈希表
├── german.txt / english.txt / french*.txt  # 示例棋盘
├── performance-debugging.txt # 性能分析参考说明
└── README.txt               # 原课程资料说明
```

`pegmark.c0` 中的 `htelem_of_board` 尚未实现，需先根据求解器的棋盘编码补全，才能使用该基准测试。原 `README.txt` 提到的 `peg-test.c0` 不在当前目录中。


