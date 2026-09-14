# CRC 循环冗余校验
- Cyclic Redundancy Check，是一种用于检测数据在传输或存储过程中是否发生错误的校验方法，主要负责检错不负责纠错，是基于模 2 多项式除法（即无进位的加减法）的高效检错机制
- 将一串二进制数据看成一个很长的二进制多项式，然后使用一个约定好的“生成多项式”去做模 2 除法，得到的余数就是 CRC 校验值（CRC 值是为了校验而额外附加的比特，所以是冗余）
- 通常余数为 0 就说明未检测到错误

##  CRC 算法参数
```python
width   ：CRC 位宽，例如 8、16、32
poly    ：生成多项式
init    ：寄存器初始值
refin   ：输入字节是否按位反射
refout  ：输出 CRC 是否按位反射
xorout  ：最终结果异或值

# 标准 CRC-32
width  = 32
poly   = 0x04C11DB7
init   = 0xFFFFFFFF
refin  = true
refout = true
xorout = 0xFFFFFFFF



# 基础/裸 CRC-3
width = 3
poly = 0x3        # 完整二进制多项式为 1011，即 x^3 + x + 1
init = 0x0        # 你的代码没有预置初始寄存器，等价于全 0
refin = false     # 数据从左到右处理，没有按位反射
refout = false    # 最终余数不反射
xorout = 0x0      # 最终余数没有额外异或
```



## 步骤
- 1.1 选择生成多项式 G(x)，比如 1011（ G(x) = x^3 + x + 1 ）
- 1.2 根据 G(x) 的的最高次数确定 CRC 位数 r，这里 r = 3
- 1.3 在原始数据后补 r 个 0，（其实就是为了给最终计算出的 r 位 CRC 留出位置）
- 2.1 对补零后的数据使用模 2 除法（异或除法）
- 2.2 得到 r 位余数 R，就是 CRC 校验值
接收端用发送帧除以 G，余数为 0 则通过，否则报错


```python
# 基础/裸 CRC-3 模 2 除法实现
def crc_bits(data: str, generator: str = "1011") -> str:
    """
    使用模 2 除法计算 CRC 余数。

    参数：
        data: 原始二进制数据字符串，例如 "101110"
        generator: 生成多项式的二进制表示。
                   默认 "1011"，表示 x^3 + x + 1，
                   因此这是一个 3 位 CRC（CRC-3）。

    返回：
        CRC 余数字符串，例如 "110"
    """

    # 生成多项式的阶数：
    # "1011" 长度为 4，最高次为 x^3，所以 CRC 余数长度为 3 位
    degree = len(generator) - 1

    # 在原始数据尾部补上 degree 个 0。
    # CRC 模 2 除法中，先补 3 个 0，为余数预留位置。
    # 例如：101110 -> 101110000
    dividend = list(data + "0" * degree)

    # 从原始数据的每一位开始进行模 2 长除法
    for i in range(len(data)):

        # 只有当前位是 1 时，才需要与生成多项式进行异或。
        # 若当前位为 0，则相当于除法时该位置商为 0，直接跳过。
        if dividend[i] == "1":

            # 将当前窗口与生成多项式逐位异或。
            # 模 2 除法中：
            #   1 XOR 1 = 0
            #   1 XOR 0 = 1
            #   0 XOR 1 = 1
            #   0 XOR 0 = 0
            for j in range(len(generator)):
                dividend[i + j] = str(
                    int(dividend[i + j]) ^ int(generator[j])
                )

    # 最后的 degree 位就是 CRC 余数。
    # 对于生成多项式 1011，degree = 3，因此取最后 3 位。
    return "".join(dividend[-degree:])


# 原始待发送的二进制数据
data = "101110"

# 生成多项式：
# 1011 = x^3 + x + 1
# 去掉最高项 x^3 后，CRC 参数中通常写作 poly = 0x3
generator = "1011"

# 计算 CRC-3 余数
crc = crc_bits(data, generator)

# 输出计算结果
print("数据       :", data)
print("生成多项式 :", generator)
print("CRC-3      :", crc)

# 将 CRC 余数附加到原始数据末尾，形成最终发送码字
print("发送码字   :", data + crc)
```



```python
# 基础/裸 CRC-3 模 2 除法实现（详细过程）
def crc3_step_by_step(data_bits: str, poly_bits: str):
    """
    使用模 2 除法（异或）逐步计算 CRC-3，并打印每一步过程。

    参数：
        data_bits: 原始二进制数据，例如 "101110"
        poly_bits: 生成多项式的二进制表示，例如 "1011"

    说明：
        poly_bits = "1011" 表示生成多项式：
            x^3 + x + 1

        因此多项式次数为 3，最终 CRC 余数长度为 3 位。
    """

    # 计算生成多项式的次数。
    # 例如 poly_bits = "1011"，长度为 4，
    # 则最高项为 x^3，因此 poly_deg = 3。
    poly_deg = len(poly_bits) - 1

    # -------------------------------------------------
    # 1. 在原始数据末尾补 poly_deg 个 0
    # -------------------------------------------------
    # CRC 计算需要先在数据尾部补 0，
    # 补零数量等于生成多项式的阶数。
    #
    # 例如：
    # 原始数据：101110
    # 补 3 个 0：101110000
    padded_bits = list(data_bits + "0" * poly_deg)

    # 将生成多项式中的字符 '0'、'1' 转换为整数列表，
    # 方便后续执行按位异或运算。
    #
    # 例如：
    # "1011" -> [1, 0, 1, 1]
    poly = [int(b) for b in poly_bits]

    # 工作数组：保存模 2 除法过程中的当前被除数。
    # 同样转换为整数，方便直接使用 ^= 异或赋值。
    #
    # 例如：
    # ['1', '0', '1', '1', '1', '0', '0', '0', '0']
    # -> [1, 0, 1, 1, 1, 0, 0, 0, 0]
    work = [int(b) for b in padded_bits]

    # 输出初始信息
    print(f"原始数据: {data_bits}")
    print(f"生成多项式: {poly_bits}")
    print(f"补零后的数据: {''.join(padded_bits)}\n")
    print("--- 模 2 除法过程 ---")

    # -------------------------------------------------
    # 2. 执行模 2 长除法
    # -------------------------------------------------
    # 只需从原始数据的每一位开始处理。
    # 补在末尾的 0 是为余数留出的空间，不作为新的起始除法位置。
    for i in range(len(data_bits)):

        # 如果当前位置是 1，则与生成多项式进行异或。
        # 这相当于普通长除法中“被除数当前最高位为 1，可以相减”。
        #
        # 在模 2 运算中：
        # 减法与加法相同，均使用 XOR（异或）。
        if work[i] == 1:
            print(f"步骤 {i + 1}: 当前最高位为 1，与多项式异或")

            # 显示当前要参与异或的窗口。
            # 窗口长度等于生成多项式长度，例如 4 位。
            print(
                f"  当前: "
                f"{''.join(map(str, work[i:i + poly_deg + 1]))}"
            )

            # 显示生成多项式
            print(f"  多项: {poly_bits}")

            # 当前窗口逐位与生成多项式异或。
            #
            # ^= 表示：
            # work[i + j] = work[i + j] ^ poly[j]
            #
            # 例如：
            # 1011
            # XOR
            # 1011
            # ----
            # 0000
            for j in range(len(poly)):
                work[i + j] ^= poly[j]

            # 输出本轮异或后的当前窗口结果
            print(
                f"  结果: "
                f"{''.join(map(str, work[i:i + poly_deg + 1]))}\n"
            )

        # 如果当前位置为 0，则不需要异或生成多项式。
        # 相当于普通除法中这一位商为 0，直接继续下一位。
        else:
            print(f"步骤 {i + 1}: 当前最高位为 0，跳过异或\n")

    # -------------------------------------------------
    # 3. 取最后 poly_deg 位作为 CRC 余数
    # -------------------------------------------------
    # 生成多项式次数为 3，所以最终余数长度为 3 位。
    #
    # 例如：
    # work 最后 3 位为 [1, 1, 0]
    # 则 remainder = "110"
    remainder = "".join(map(str, work[-poly_deg:]))

    # 将 CRC 余数拼接到原始数据后面，
    # 得到最终发送的完整码字。
    final_frame = data_bits + remainder

    # 输出最终结果
    print("--- 最终计算结果 ---")
    print(f"CRC-3 校验码 (余数): {remainder}")
    print(f"含校验码的完整数据帧: {final_frame}")


# 运行示例：
# 原始数据：101110
# 生成多项式：1011，即 x^3 + x + 1
crc3_step_by_step("101110", "1011")
```