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
注意：信号是模 p（Babyjubjub 素数）的残基，并且没有负数模 p 的自然概念。 但是，很明显，当我们将p-1视为-1时，模运算类似于整数运算。 所以我们定义一个约定：取负 按照惯例认为是 (p/2, p-1] 中的余数，非负是 [0, p/2) 中的任意数

- 参数：无
- 输入信号：in
- 输出信号：out
要求：如果根据我们的约定，in 为负数，则 out 应为 1。 否则，out 应该是 0。 您可以自由使用CompConstant circuit，它有一个常量参数ct，如果in（二进制数组）在解释为整数时严格大于 ct 则输出 1 ，否则为 0。

```
pragma circom 2.1.4;

template Num2Bits(n) {
    signal input in;
    signal output out[n];
    var lc1=0;

    var e2=1;
    for (var i = 0; i<n; i++) {
        out[i] <-- (in >> i) & 1;
        out[i] * (out[i] -1 ) === 0;
        lc1 += out[i] * e2;
        e2 = e2+e2;
    }

    lc1 === in;
}


// Returns 1 if in (in binary) > ct
template CompConstant(ct) {
    signal input in[254];
    signal output out;

    signal parts[127];
    signal sout;

    var clsb;
    var cmsb;
    var slsb;
    var smsb;

    var sum=0;

    var b = (1 << 128) -1;
    var a = 1;
    var e = 1;
    var i;

    for (i=0;i<127; i++) {
        clsb = (ct >> (i*2)) & 1;
        cmsb = (ct >> (i*2+1)) & 1;
        slsb = in[i*2];
        smsb = in[i*2+1];

        if ((cmsb==0)&&(clsb==0)) {
            parts[i] <== -b*smsb*slsb + b*smsb + b*slsb;
        } else if ((cmsb==0)&&(clsb==1)) {
            parts[i] <== a*smsb*slsb - a*slsb + b*smsb - a*smsb + a;
        } else if ((cmsb==1)&&(clsb==0)) {
            parts[i] <== b*smsb*slsb - a*smsb + a;
        } else {
            parts[i] <== -a*smsb*slsb + a;
        }

        sum = sum + parts[i];

        b = b -e;
        a = a +e;
        e = e*2;
    }

    sout <== sum;

    component num2bits = Num2Bits(135);

    num2bits.in <== sout;

    out <== num2bits.out[127];
}


template IsNegative() {
    signal input in;
    signal output out;

    var p = 21888242871839275222246405745257275088548364400416034343698204186575808495617;
    component num2bits = Num2Bits(254);
    component compconstant = CompConstant((p - 1) \ 2);
    num2bits.in <== in;
    compconstant.in <== num2bits.out; 
    out <== compconstant.out;
}

component main = IsNegative(); 

/* INPUT = {
    "in": "21888242871839275222246405745257275088548364400416034343698204186575808495616"
} */
```

## 第 6 题: 少于 LessThan
- 参数：无
- 输入信号：`in[2]`。 假设提前知道这些最多 $2^{252} - 1$。
- 输出信号：`out`

要求：如果 `in[0]` 严格小于 `in[1]`，则 `out` 应为 `1`。 否则，`out` 应该是 `0`。

- **扩展 1**：如果您知道输入信号最多为 $2^k - 1 (k ≤ 252)$，您如何减少该电路所需的约束总数？ 编写一个在`k`中参数化的电路版本。
- **扩展 2**：编写 LessEqThan（测试 in[0] 是否 ≤ in[1]）、GreaterThan 和 GreaterEqThan
```
pragma circom 2.1.4;

template Num2Bits(n) {
    signal input in;
    signal output out[n];
    var lc1=0;

    var e2=1;
    for (var i = 0; i<n; i++) {
        out[i] <-- (in >> i) & 1;
        out[i] * (out[i] -1 ) === 0;
        lc1 += out[i] * e2;
        e2 = e2+e2;
    }

    lc1 === in;
}

template LessThan(n) {
    assert(n <= 252);
    signal input in[2];
    signal output out;

    component n2b = Num2Bits(n+1);

    n2b.in <== in[0]+ (1<<n) - in[1];

    out <== 1-n2b.out[n];
}

template LessThan (n) {
    assert(n <= 252);
    signal input in[2];
    signal output out;

    component n2b = Num2Bits(n+1);
    n2b.in <== in[1] - in[0] + (2**n); // n+1位二进制，最大数是 2**n
    out <== n2b.out[n];
}

template GreaterThan(n) {
    assert(n <= 252);
    signal input in[2];
    signal output out;

    component n2b = Num2Bits(n+1);
    n2b.in <== in[1] - in[0] + (2**n);
    out <== 1 - n2b.out[n];
}

template GreaterEqThan(n) {
    assert(n <= 252);
    signal input in[2];
    signal output out;

    component n2b = Num2Bits(n+1);
    n2b.in <== in[0] - in[1] + (2**n);
    out <== n2b.out[n];
}

component main = LessThan(3); 

/* INPUT = {
    "in": ["6","7"]
} */
```

## 第 7 题: 整数除法 IntegerDivide
注意：这个电路非常难！

- 参数：`nbits`。 使用 `assert` 断言这最多为 126！
- 输入信号：`dividend`, `divisor` （被除数，除数）
- 输出信号：`remainder`, `quotient` （余数，商）

要求：首先，检查`dividend`和`divisor`是否最多为`nbits`位长。 接下来，计算并约束`余数`和`商`。

- **扩展**：您将如何修改电路以处理负的被除数？

```
pragma circom 2.1.4;

// 将 in 转换为 n 位二进制表达
template Num2Bits(n) {
    signal input in;
    signal output out[n];

    for (var i = 0; i < n; i++) {
        out[i] <-- (in \ 2**i) % 2;
    }

    var accm = 0;
    for (var i = 0; i < n; i++) {
        accm += out[i] * 2**i;
    }
    accm === in;

    // 约束输出的每一位是 0 / 1
    for (var i = 0; i < n; i++) {
        out[i] * (1 - out[i]) === 0;
    }
}

// in[0] < in[1] 返回 1，否则返回0
template LessThan (n) {
    // 先确定输入信号最多为 2**252 - 1
    assert(n <= 252);
    signal input in[2];
    signal output out;

    component n2b = Num2Bits(n+1);
    n2b.in <== in[0] - in[1] + (2**n); // n+1位二进制，最大数是 2**n
    out <== 1 - n2b.out[n];
}


// 限制 in 最多为 nbits 位
template Range(nbits) {
    signal input in;
    signal bits[nbits];
    var bitsum = 0;
    for (var i = 0; i < nbits; i++) {
        bits[i] <-- (in >> i) & 1;
        bits[i] * (1 - bits[i]) === 0;
        bitsum = bitsum + 2 ** i * bits[i];
    }

    in === bitsum;
}

template IntegerDivide (nbits) {
    assert(nbits <= 126);
    signal input dividend;
    signal input divisor;

    signal output remainder;
    signal output quotient;
    
    // 限制被除数、除数最多为 nbits 位
    component range1 = Range(nbits);
    range1.in <== dividend;
    component range2 = Range(nbits);
    range2.in <== divisor;

    // 限制除数不能为0
    signal inv;
    inv <-- 1 / divisor;
    inv * divisor === 1;

    // 计算商和余数
    quotient <-- dividend \ divisor;
    remainder <== dividend - quotient * divisor;

    // 约束余数在范围 [0, divisor) 之间
    component lessthan = LessThan(nbits);
    lessthan.in[0] <== remainder;
    lessthan.in[1] <== divisor;
    lessthan.out === 1;

    // 约束是正确的计算
    dividend === quotient * divisor + remainder;
}

component main = IntegerDivide(9);

/* INPUT = {
    "dividend": "76",
    "divisor": "37"
} */
```

## 第 8 题【可选】: 排序 Sort 
- 参数：`N`
- 输入信号：`in[N]`
- 输出信号：`out[N]`

要求：将输入`in[N]`的`N`个数字按照从小到大进行排列，并输出到`out[N]`信号中。
