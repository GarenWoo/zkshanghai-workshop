# 第 2 课 课后作业

## 第 1 题: 转换为 bit 位 Num2Bits
- 参数：nBits
- 输入信号：in
- 输出信号：b[nBits]

输出信号应该是长度为nBits的位数组，相当于in的二进制表示。 b[0] 是最低有效位。


我的答案：
```
pragma circom 2.1.4;

include "circomlib/poseidon.circom";
// include "https://github.com/0xPARC/circom-secp256k1/blob/master/circuits/bigint.circom";

template Num2Bits (nBits) {
    signal input in;
    signal output b[nBits];
    var acc = 0;

    for (var i = 0; i < nBits; i++) {
        b[i] <-- (in \ 2 ** i) % 2;          // 二进制表达
        0 === b[i] * (1 - b[i]);              // 约束：b[i]为二进制数, 1 or 0
        acc += b[i] * (2 ** i);            // 累加
    }
    acc === in;           // 约束：二进制累与原数值加相等
}

component main  = Num2Bits(5);

/* INPUT = {
    "in": "11"
} */
```

## 第 2 题: 判零 IsZero


## 第 3 题: 相等 IsEqual


## 第 4 题: 选择器 Selector


## 第 5 题: 判负 IsNegative


## 第 6 题: 少于 LessThan


## 第 7 题: 整数除法 IntegerDivide


## 第 8 题【可选】: 排序 Sort 
