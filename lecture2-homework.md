# 第 2 课 课后作业

## 第 1 题: 转换为 bit 位 Num2Bits
- 参数：nBits
- 输入信号：in
- 输出信号：b[nBits]

输出信号应该是长度为nBits的位数组，相当于in的二进制表示。 b[0] 是最低有效位。


我的答案：
```
pragma circom 2.1.4;

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
- 此电路将十进制数转换为二进制数，即表达方式的改变。
- nBits 为数组的元素个数，决定此数组所能代表的最大数。例如当 nBits 为 5 时，max = 11111(binary) = 2 ** 5 - 1= 31(decimal)。
- b[i] * (1 - b[i])：对二进制数的任意一位施加约束，为 0 或 1 。
- acc === in：保证 acc 与 in 两者的数值是一致的（仅是表达方式不同而已）。

## 第 2 题: 判零 IsZero
- 参数：无
- 输入信号：in
- 输出信号：out

要求：如果in为零，out应为1。 如果in不为零，out应为0。 这个有点棘手！


我的答案：
```
pragma circom 2.1.4;

template IsZero () {
    signal input in;
    signal output out;

    signal inv;

    inv <-- in!=0 ? 1/in : 0;

    out <== -in*inv +1;
    in*out === 0;
}

component main  = IsZero ();

/* INPUT = {
    "in":"0"
    } */
```
- 此电路使用中间变量 inv 为 0 或 1/in。
    * 当 in = 0, inv = 0, out = 0 * 0 + 1 = 1
    * 当 in ≠ 0, inv = 1/in, out = -in * 1/in + 1 = -1 + 1 = 0
- 施加约束，使得 in 和 out 至少其中之一为 0 。

## 第 3 题: 相等 IsEqual
- 参数：无
- 输入信号：in[2]
- 输出信号：out

要求：如果 in[0] 等于 in[1]，则 out 应为 1。 否则，out 应该是 0。
```
pragma circom 2.1.4;

template IsZero () {
    signal input in;
    signal output out;

    signal inv;

    inv <-- in!=0 ? 1/in : 0;

    out <== -in*inv +1;
    in*out === 0;
}

template IsEqual() {
    signal input in[2];
    signal output out;
    component isZero = IsZero();
    isZero.in <== in[0] - in[1];
    isZero.out ==> out;
}

component main  = IsEqual ();

/* INPUT = {
    "in": ["3", "3"]
} */
```

## 第 4 题: 选择器 Selector
- 参数：nChoices
- 输入信号：in[nChoices], index
- 输出信号：out

要求：输出out应该等于in[index]。 如果 index 越界（不在 [0, nChoices) 中），out 应该是 0。
```
pragma circom 2.1.4;

include "circomlib/poseidon.circom";

template IsZero () {
    signal input in;
    signal output out;

    signal inv;

    inv <-- in!=0 ? 1/in : 0;

    out <== -in*inv +1;
    in*out === 0;
}

template IsEqual() {
    signal input in[2];
    signal output out;
    component isZero = IsZero();
    isZero.in <== in[0] - in[1];
    isZero.out ==> out;
}

template Selector(nChoices) {
    signal input in[nChoices];
    signal input index;
    signal output out;
    signal sumUp[nChoices + 1];
    sumUp[0] <== 0;
    component isEqual[nChoices];
 for (var i = 0; i < nChoices; i++) {
    isEqual[i] = IsEqual();
    isEqual[i].in[0] <== i;
    isEqual[i].in[1] <== index;
    sumUp[i] + isEqual[i].out * in[i] ==> sumUp[i + 1];
}
    out <== sumUp[nChoices];
}

component main  = Selector(5);


/* INPUT = {
    "in": ["1", "2", "3", "4", "5"],
    "index": 4
} */
```

## 第 5 题: 判负 IsNegative


## 第 6 题: 少于 LessThan


## 第 7 题: 整数除法 IntegerDivide


## 第 8 题【可选】: 排序 Sort 
