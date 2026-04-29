# just_for_record
# 量化笔试题 - 从零基础到面试级别完整解析 (深度版)

---

> **本文档定位**: 面向零量化基础的Python开发者，目标是让你在面对面试官的任何追问时都能从容回答。每个概念从"是什么"讲到"为什么"，每个设计决策从"做了什么"讲到"为什么不选其他方案"。文档按"先理解再应答"的结构组织——先讲清楚知识，再预判面试官会怎么问。

---

# 目录

1. [量化零基础入门](#1-量化零基础入门)
2. [Python量化编程深度](#2-python量化编程深度)
3. [笔试题逐行代码精讲](#3-笔试题逐行代码精讲)
4. [面试官100问](#4-面试官100问)
5. [面试模拟对话](#5-面试模拟对话)

---

# 1. 量化零基础入门

## 1.1 量化的本质: 用代码做投资

### 从传统投资到量化投资

传统投资经理早上到办公室，打开同花顺看行情，读研报，跟上市公司董秘打电话，凭经验判断"这个股票被低估了，买"。这叫**主观投资**。

量化投资不靠"感觉"。它靠的是:

```
数据 → 模型 → 信号 → 执行
```

举个例子:
```
① 数据: 获取全市场5000只股票的PE、PB、ROE、动量等指标
② 模型: 多因子模型 = 买低PE + 买高ROE + 买高动量
③ 信号: 模型输出"今天该买这50只，卖那30只"
④ 执行: 程序自动下单 (或生成下单指令给交易员)
```

**这道笔试题在量化流程中的位置:**

笔试题让你做的其实是量化系统的"绩效归因"模块:
- 题目2: 你的组合现在值多少钱? (持仓市值)
- 题目3: 你的组合中有多少是某个指数的成份股? (暴露分析)
- 题目4: 你的组合今天赚了多少? 跑赢市场了吗? (绩效归因)

这是量化交易最基础的日常操作——每天收盘后计算当天的PnL和风险暴露。

### 量化的三个层次 (面试加分概念)

| 层次 | 做什么 | 技能要求 | 本题对应 |
|------|--------|---------|---------|
| **P quant** | 定价、风险、套利 | 随机微积分、偏微分方程 | - |
| **Q quant** | 统计套利、因子模型 | 统计学、计量经济学 | 题目4(超额收益) |
| **执行量化** | 算法交易、TCA | 计算机科学、低延迟 | 题目2/3(数据处理) |

这道笔试题偏重"执行量化"和"Q quant的基础操作"。

---

## 1.2 股票账户的"净资产"怎么算

### 场景: 你的股票账户

假设今天是2025年12月12日收盘后，你的账户持仓如下:

```
股票代码    持有数量    今日收盘价
000001      200股       11.35元
600000      100股       10.50元
688981      200股       52.00元
```

你账户的市值 = 200×11.35 + 100×10.50 + 200×52.00 = 2,270 + 1,050 + 10,400 = 13,720元

数学公式:

$$TotalMarketValue = \sum_{i=1}^{n} hold\\_vol_i \times close\\_price_i$$

这就是题目2要做的事情——只不过持仓有618只股票，不是3只。

### 关键概念辨析: 市值 vs 持仓成本 vs 盈亏

- **市值 (Market Value)**: 按当前市价计算的值多少钱。这是"如果现在全部卖掉能拿回多少"。
- **持仓成本 (Cost Basis)**: 你当初买入时花了多少钱。比如你10元买的，现在市价12元，成本是2000元，市值是2400元。
- **浮动盈亏 (Unrealized PnL)**: 市值 − 成本 = 2400 − 2000 = 400元。你没卖，所以叫"浮动"。

本题只算**市值**，不涉及成本。收益计算(题目4)算的是**单日浮动盈亏**。

### 为什么题目指定"收盘价"而不是"最新价"或"均价"?

1. **收盘价是法定定价基准**: 《企业会计准则》规定金融资产按公允价值计量，收盘价是最公认的公允价值。
2. **基金净值用收盘价**: 你买的基金每天公布的净值，底层资产都按收盘价估值。
3. **一致性**: 所有交易参与者在同一个时间点(15:00)用同一个价格，才能公平比较。
4. **可审计**: 收盘价由交易所公开发布，不可篡改，审计师可以复验。

如果面试官问"为什么不用盘中实时价"，以上四点就是标准答案。

---

## 1.3 收益率: 你今天赚了多少?

### 金额 vs 比率

你今天赚了多少有两种表达:

**方式一: 赚了多少元 (PnL金额)**
```
PnL = 今日市值 − 昨日市值
    = Σ(持仓量 × 今日收盘价) − Σ(持仓量 × 昨日收盘价)
    = Σ[持仓量 × (今日收盘价 − 昨日收盘价)]
```

**方式二: 赚了百分之多少 (收益率)**
```
收益率 = (今日市值 − 昨日市值) / 昨日市值 × 100%
       = PnL / 昨日市值 × 100%
```

为什么需要收益率? 因为金额没有可比性:
- 1000万的账户赚了5万 → 收益率0.5%
- 10万的账户赚了5万 → 收益率50%
- 虽然金额一样，但后者赚得"狠"得多

### 为什么收益率分母用"昨日市值"而不是"今日市值"?

这是面试高频追问。深层原因:

$$Return = \frac{V_t - V_{t-1}}{V_{t-1}}$$

这个公式衡量的是: "我昨天持有的资产，到今天涨了多少"。

- 分子: 今天比昨天多出来的钱 (今天新增的)
- 分母: 昨天我有多少钱 (投入的基础)

如果用今日市值做分母: `(V_t - V_{t-1}) / V_t`，数学上这叫"折价率"而非"收益率"——它衡量的是"今天的资产中有多少是今天赚的"，逻辑上不是回报率的标准定义。

**金融行业的统一标准**: CFA、基金业协会、全球投资业绩标准(GIPS)全部使用 `(V_t - V_{t-1}) / V_{t-1}` 作为收益率定义。

### 简单收益率 vs 对数收益率 (面试加分概念)

```python
# 简单收益率 (本题使用, 业界最常用)
simple_return = (close - preclose) / preclose

# 对数收益率 (学术界常用, 有时间可加性)
log_return = log(close / preclose)
```

区别:
- 简单收益率: 直观，可加总(金额加权)
- 对数收益率: 有时间可加性(两天的log_return相加=两天总log_return)，近似正态分布

本题用简单收益率即可，这是行业惯例。

---

## 1.4 超额收益 (Alpha): 你比市场强多少?

### 为什么要跟市场比?

假设2025年12月12日:
- 你的组合涨了 **+1.2%**
- 沪深300指数涨了 **+2.5%**
- 你的超额收益 = 1.2% − 2.5% = **−1.3%**

你的1.2%看起来是正的，但如果当天是大牛市(央行降准之类的利好)，全市场普涨，你只涨1.2%其实说明你的选股反而拖了后腿。

反之:
- 你的组合跌了 **−0.5%**
- 沪深300跌了 **−3.0%**
- 你的超额收益 = −0.5% − (−3.0%) = **+2.5%**

虽然当天你亏钱了，但大盘暴跌时你只亏0.5%，说明你的风控做得很好。这种"熊市抗跌"恰是Alpha的来源。

### Alpha的金融学定义

在CAPM(资本资产定价模型)框架下:

$$R_{portfolio} = \alpha + \beta \times R_{market} + \epsilon$$

- **α (Alpha)**: 与市场无关的超额收益——你的选股/择时能力
- **β (Beta)**: 与市场相关的系统性收益——你承担了多少市场风险
- **ε**: 随机扰动

简单版(本题用的):

$$\alpha_{simple} = R_{portfolio} - R_{benchmark}$$

这是"简单超额收益"，假设β=1(你的组合跟市场同涨同跌)。精确的Alpha需要通过回归估算β，但这超出了笔试题范围，面试时提及这个区别会是加分项。

---

## 1.5 A股代码体系完全指南

### 为什么代码是6位数字?

中国证券代码采用6位定长编码，由交易所分配，终身不变(除非退市后重新上市)。

### 完整代码范围表

| 代码段 | 交易所 | 板块 | 成立时间 | 典型股票 | 涨跌幅限制 |
|--------|--------|------|---------|---------|-----------|
| 600000-609999 | 上交所 | 主板 | 1990 | 600000浦发银行 | ±10% |
| 688000-688999 | 上交所 | 科创板 | 2019 | 688981中芯国际 | ±20% |
| 000001-003999 | 深交所 | 主板 | 1990 | 000001平安银行 | ±10% |
| 002000-002999 | 深交所 | 中小板(已并入主板) | 2004 | 002230科大讯飞 | ±10% |
| 300000-300999 | 深交所 | 创业板 | 2009 | 300750宁德时代 | ±20% |
| 301000-301999 | 深交所 | 创业板(注册制) | 2020 | 301565中仑新材 | ±20% |
| 001000-001999 | 深交所 | 主板(合并后) | 2021 | 001289龙源电力 | ±10% |

### 代码判断逻辑 (面试可能让你手写)

```python
def judge_exchange(code: str) -> str:
    """判断股票属于哪个交易所"""
    code_int = int(code)

    if 600000 <= code_int <= 609999:
        return "上交所主板"
    elif 688000 <= code_int <= 688999:
        return "上交所科创板"
    elif 1 <= code_int <= 3999:  # 000001-003999, 001000-001999, 002000-002999
        return "深交所主板"
    elif 300000 <= code_int <= 301999:
        return "深交所创业板"
    else:
        return "未知"
```

**注意:** `int(code)` 会自动处理前导零，所以 `int("000001")` = 1。

### baostock格式映射

```python
def to_bs_format(code: str) -> str:
    """将6位代码转为baostock查询格式"""
    code_int = int(code)
    if code_int >= 600000:  # 6开头 → 上海交易所
        return f"sh.{code}"
    else:                    # 0/3开头 → 深圳交易所
        return f"sz.{code}"

# 示例:
# "600000" → "sh.600000"
# "000001" → "sz.000001"
# "300750" → "sz.300750"
```

**为什么临界值是600000?** 因为A股代码体系里，600000及以上全是上海交易所的票(600xxx主板+688xxx科创板)，600000以下的都是深圳交易所的票(000xxx+001xxx+002xxx+300xxx+301xxx)。

---

## 1.6 中国指数体系详解

### 上证系列指数 (上海证券交易所发布)

```
上证综指 (000001) - 所有上交所股票, 最古老的A股指数
  │
  ├── 上证180 (000010) - 沪市最大最活跃的180只 (大盘蓝筹)
  │     └── 上证50 (000016) - 沪市超大盘50只
  │
  ├── 上证380 (000009) - 剔除上证180后, 选380只新兴蓝筹
  │     = 沪市"中盘成长"代表
  │
  └── 上证科创50 (000688) - 科创板50只
```

### 沪深300 (中证指数公司发布, 跨两市)

```
沪深300 (000300 / sh.000300)
  = 沪市最大的约200只 + 深市最大的约100只
  = A股最通用的业绩基准
```

### 上证380的详细选股规则 (面试高分细节)

1. **样本空间**: 上证180成份股**之外**的所有上交所股票
2. **剔除条件**: ST股、*ST股、暂停上市股、财务有问题的
3. **排名因子**: 营业收入增长率、净资产收益率(ROE)、成交金额、总市值
4. **综合打分**: 四个因子加权排名, 选前380名
5. **调整频率**: 每半年调整一次 (6月和12月第二个周五后的周一)

**这意味着什么?** 上证380是沪市的"二线蓝筹"——不是最大的(被上证180挑走了)，但是成长性最好的。它代表的是"明日之星"而非"今日巨头"。

### 为什么笔试题选上证380和沪深300?

- **上证380**: 测试爬虫能力 + 指数概念理解 + 不是最常用的(面试官想看你没准备过的东西)
- **沪深300**: 测试你是否知道"基准"的概念 + 超额收益计算

---

## 1.7 除权除息: 为什么股价会突然"暴跌"

### 场景还原

```python
# 某股票昨天收盘 = 10.00元
# 今天每股分红 = 0.50元 (现金红利)
# 今天开盘参考价 = 10.00 - 0.50 = 9.50元

# 如果你持有1000股:
# 昨天市值 = 1000 × 10.00 = 10,000元
# 今天市值 = 1000 × 9.50 = 9,500元 (+ 500元现金红利在路上)
# 总资产 = 9,500 + 500 = 10,000元 → 没亏没赚
```

**如果你只看K线图**: 昨天收盘10块，今天"跌"到9.5，你会以为跌了5%。但实际上你没亏钱——那5%变成了现金红利。

**除权**: 送股导致的价格调整 (10送10 → 股价砍半)
**除息**: 分红导致的价格调整 (每股分0.5 → 股价减0.5)

### 不复权 vs 后复权 vs 前复权

这是baostock `adjustflag` 参数的三个选项，面试官几乎必问:

| adjustflag | 名称 | 价格含义 | 典型用途 |
|-----------|------|---------|---------|
| '1' | 后复权 | 以最新股本往前乘系数, 历史价格被放大 | 计算长期收益率 |
| '2' | 前复权 | 以最初股本往后除系数, 近期价格被缩小 | 技术分析画图 |
| '3' | 不复权 | 交易日实际成交价, 不调整 | **计算当日市值** |

### 本题为什么用 `adjustflag='3'` (不复权)?

**核心原因**: 市值 = 股数 × **实际成交价**。

如果你用了复权价格:
- 复权价格不是交易所实际成交的价格
- 算出的市值不是账户真实的权益金额
- 跟券商对账单对不上

**那收益率怎么办?** 单日收益率用 `close` 和 `preclose`，这两个价格都来自同一个交易日前后，且 `preclose` (昨收价) 是交易所发布的、已经考虑了除权除息的参考价。所以单日收益率直接用不复权价格的 `(close - preclose) / preclose`，在绝大多数情况下是正确的。

**什么时候复权收益率会出问题?** 只当除权除息日正好是你计算的那一天。此时 `preclose` 是除权前的收盘价，`close` 是除权后的交易价，两者之间包含了除权缺口。但题目给的是2025年12月12日，不是集中除权除息季(通常是5-7月)，所以影响微乎其微。面试时你能说出这个细节，属于高分表现。

---

## 1.8 交易日历概念

### 什么是交易日?

A股交易时间: 周一至周五 (法定节假日除外), 9:30-11:30, 13:00-15:00。

**为什么这个概念重要?**
- 你查12月12日的价格，如果那天是周六，baostock不会返回数据
- 你算"昨收"，实际上是"上一个交易日的收盘价"，不是"日历昨天"
- baostock的 `preclose` 字段已经帮你处理好了——它自动取上一个交易日

2025年12月12日是周五，正常交易日。面试官选这个日期是精心设计的。

---

# 2. Python量化编程深度

## 2.1 向量化: 量化Python的第一铁律

### Python为什么慢?

Python是解释型语言，每个操作都有巨大的overhead:

```python
# 这段代码在Python解释器里发生了什么?
a = df['hold_vol'][0]    # ① 索引查找 (hash lookup)
b = df['close'][0]       # ② 索引查找
c = a * b                # ③ Python int * Python float → 类型检查 → 装箱 → 乘法 → 拆箱
result += c              # ④ 加法 + 赋值
```

每行代码背后有数十条C指令在执行Python的对象模型(引用计数、类型检查、方法解析等)。跑618次(每只股票) × 这些overhead = 慢。

### NumPy怎么解决?

```python
# NumPy的C源码简化示意:
# for (i = 0; i < n; i++) {
#     output[i] = input1[i] * input2[i];
# }
result = (df['hold_vol'].values * df['close'].values).sum()
```

关键差异:
- `.values` 返回的是 `numpy.ndarray` —— 一块连续内存
- 乘法操作直接调用C的循环，没有Python对象参与
- CPU可以SIMD(单指令多数据)一条指令同时算4-8个元素
- 没有GIL(全局解释器锁)的竞争

### 性能数据 (真实测试)

```python
import numpy as np
import pandas as pd
import time

n = 618  # 本题持仓数量
df = pd.DataFrame({
    'hold_vol': np.random.randint(100, 5000, n),
    'close': np.random.uniform(1, 100, n)
})

# for循环方式
t0 = time.time()
total = 0
for i in range(len(df)):
    total += df['hold_vol'].iloc[i] * df['close'].iloc[i]
t_for = time.time() - t0

# 向量化方式
t0 = time.time()
total = (df['hold_vol'] * df['close']).sum()
t_vec = time.time() - t0

# 结果: t_for ≈ 0.001-0.003s, t_vec ≈ 0.00003-0.0001s
# 向量化快约 30-100倍 (对于618行数据)
# 如果是60万行数据, 差距会扩大到 200-500倍
```

### for循环在本题的唯一出现: API查询

```python
for bs_code in bs_codes:
    rs = bs.query_history_k_data_plus(bs_code, ...)
```

这里用for循环是因为:
1. **这是I/O操作，不是数值计算**: 每次循环在等网络响应(ms级延迟)，CPU完全在闲置
2. **baostock API不支持批量查询**: 一次只能查一只股票
3. **题目要求"秒级"**: 600次网络请求约60秒，这是网络I/O时间，不是计算时间

**改进方案(面试加分):** 使用 `concurrent.futures.ThreadPoolExecutor` 多线程并发查询。但baostock的login/logout不是线程安全的，需要每个线程独立login。实际优化幅度约3-5倍(受限于baostock服务端限流)。

```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import baostock as bs

def fetch_one(bs_code, trade_date):
    """每个线程独立login/查询/logout"""
    bs.login()
    rs = bs.query_history_k_data_plus(bs_code, "date,code,close,preclose",
                                       start_date=trade_date, end_date=trade_date,
                                       frequency="d", adjustflag="3")
    row = rs.get_row_data() if rs.error_code == '0' and rs.next() else None
    bs.logout()
    return row

with ThreadPoolExecutor(max_workers=10) as executor:
    futures = {executor.submit(fetch_one, code, trade_date): code for code in bs_codes}
    for future in as_completed(futures):
        row = future.result()
        if row:
            all_rows.append(row)
```

**面试时怎么说**: "我的代码中唯一的for循环是用在baostock API查询环节，因为API本身不支持批量查询，这是I/O密集操作不是计算密集操作。如果面试官要求消除这个循环，可以改为多线程并发查询，但需要每个线程独立维护baostock会话。"

---

## 2.2 pd.merge 内部原理

面试官问"pd.merge里有循环吗"，你需要理解Hash Join算法。

### Hash Join 三步走

```python
# pd.merge(holdings, prices, on='code', how='left')
# 内部执行流程:
```

**Step 1: Build Phase (构建哈希表)**
```python
# 对右表(prices)的'code'列建哈希表
hash_table = {}
for code in prices['code']:
    hash_table[hash(code)] = code  # O(m), m = prices行数
```

**Step 2: Probe Phase (探测匹配)**
```python
# 遍历左表(holdings), 在哈希表中查找匹配
for code in holdings['code']:
    matched_row = hash_table.get(hash(code))  # O(1) 哈希查找, 不是O(n)线性搜索!
    # 合并两行数据
```

**Step 3: Handle Unmatched (处理未匹配, left join特有)**
```python
# 左表有但右表没有的 → 右表字段填NaN
```

**总复杂度: O(n+m)** 而非嵌套循环的 O(n×m)。

### 为什么不是嵌套循环?

```python
# 如果是嵌套循环 (merge绝对不用这种方式):
for h_row in holdings.iterrows():     # O(n)
    for p_row in prices.iterrows():   # O(m)
        if h_row['code'] == p_row['code']:  # 总共 O(n×m) 次比较
            ...
# 618 × 618 = 381,924 次字符串比较 → 慢
```

### merge的join类型选择

| how参数 | SQL等价 | 含义 | 本题使用场景 |
|---------|---------|------|------------|
| 'left' | LEFT JOIN | 保留左表所有行, 右表无匹配填NaN | **题目2**: 持仓全保留, 没价格的标出来 |
| 'inner' | INNER JOIN | 只保留两表都有的行 | **题目3**: 只在成份股中的持仓 |
| 'right' | RIGHT JOIN | 保留右表所有行 | 一般不用 |
| 'outer' | FULL OUTER JOIN | 两表都保留 | 数据完整性检查 |

**面试追问: "题目2为什么用left join而不是inner join?"**

答: "如果某只持仓股票在baostock查不到价格(停牌、退市、数据缺失), 用inner join会静默丢弃它, 用户不知道有数据缺失。用left join则保留该行, close列为NaN, 我可以统计有多少只股票未匹配并报告给用户。在量化交易中, 数据缺失不能悄悄忽略, 必须显式报告。"

---

## 2.3 数据清洗的每一步为什么这么做

```python
# 1. 删除空值行
df = df.dropna(subset=["code", "hold_vol"])
# 为什么: CSV可能有空行(Excel编辑后保存常产生), 空代码无法查询价格, 空持仓无法计算
# 为什么用subset: 只能删code或hold_vol为空的, 其他列有空无所谓

# 2. 去重
df = df.drop_duplicates(subset="code", keep="first")
# 为什么keep='first': 同一股票出现两次大概率是录入重复, 保留第一条是最保守策略
# 为什么不用keep=False(全删): 万一只是一条有效记录, 删除可能导致数据丢失

# 3. 过滤0持仓
df = df[df["hold_vol"] > 0]
# 为什么: hold_vol=0表示不持仓, 不影响市值, 但留着会浪费API查询次数
# 注意: 如果某股票昨天有持仓今天清仓了, hold_vol=0是正常的, 不是数据错误

# 4. 代码标准化
df["code"] = df["code"].str.strip().str.zfill(6)
# 为什么strip: Excel里数字可能前后有空格; " 000001" → "000001"
# 为什么zfill(6): 代码可能是纯数字存储的: 1 → "000001"; "0001" → "000001"
#   这是最常见的CSV数据问题: 前导零被Excel吃掉

# 5. 正则过滤
df = df[df["code"].str.fullmatch(r"\d{6}")]
# 为什么: 过滤掉"ABC123"这种非法代码; fullmatch保证整串是6位数字
# 为什么不用str.len()==6: 因为"AB1234"也是6位, 但不是纯数字
```

### 面试官可能追问的边界情况

**Q: "如果CSV文件编码不是UTF-8怎么办?"**

A: pandas的 `read_csv` 默认用系统编码。Windows中文系统默认是GBK。如果CSV是UTF-8 with BOM, pandas通常能自动检测。如果乱码，`encoding` 参数可以指定。但题目给的文件是标准ASCII/UTF-8，不涉及编码问题。

**Q: "如果持仓文件有100万行怎么办?"**

A: `pd.read_csv` 对百万行仍然很快(<1秒)。数据清洗的向量化操作也都是一次性的。真正的瓶颈在baostock API查询——如果真有100万只不同的股票，API查询会非常慢。但实际A股总共也就5000多只，持仓文件不可能超过这个数。

**Q: "为什么zfill(6)而不是补到其他位数?"**

A: 中国A股代码标准是6位。基金代码也是6位。没有一个交易所的代码需要补到其他位数。

---

## 2.4 staticmethod vs classmethod: 从Python解释器角度

### 描述符协议 (Descriptor Protocol)

`@staticmethod` 和 `@classmethod` 在Python底层是通过**描述符协议**实现的:

```python
# @staticmethod 的C语言简化实现:
class staticmethod:
    def __init__(self, func):
        self.__func__ = func

    def __get__(self, obj, cls=None):
        # 关键: 不管怎么调用, 都返回原始函数, 不绑定任何东西
        return self.__func__

# @classmethod 的C语言简化实现:
class classmethod:
    def __init__(self, func):
        self.__func__ = func

    def __get__(self, obj, cls=None):
        # 关键: 把类(cls)绑定为第一个参数
        return self.__func__.__get__(cls, type(cls))
```

**这意味着什么?** 当你写 `MyClass.method()` 时:
- 如果是staticmethod: Python直接调用原始函数, 不传任何额外参数
- 如果是classmethod: Python自动把 `MyClass` 作为第一个参数(cls)传入
- 如果是普通方法: Python自动把实例 `self` 作为第一个参数传入

### 性能差异 (微乎其微, 但面试可能问)

```python
import timeit

# staticmethod比classmethod略微快一点(纳秒级差距, 可忽略)
# 因为classmethod多了一层闭包调用
# 但在实际应用中, 这种差异完全可以忽略
```

### 何时用static, 何时用class

**用 @staticmethod 的场景:**
- 函数逻辑与类相关但不需要访问类的任何数据
- 纯工具/辅助函数
- 例: `is_valid_code()`, `format_price()`, `clean_text()`

**用 @classmethod 的场景:**
- 工厂方法(替代构造函数重载): `from_csv()`, `from_api()`
- 需要访问/修改类属性: `set_config()`, `get_registry()`
- 需要支持继承多态: 子类调用时cls自动指向子类
- 单例模式: `get_instance()`

**可以互相替换的场景 (但你得说出为什么选这个):**
- 函数完全独立于类和实例 → staticmethod (更纯粹)
- 函数未来可能需要根据子类变化 → classmethod (预留扩展点)

---

## 2.5 异常处理: 为什么量化代码不能"将就"

### 量化代码中数据错误 = 交易错误 = 真金白银损失

```python
# 错误的数据 → 错误的信号 → 错误的交易 → 亏钱

# 场景: 某股票价格数据源返回了0 (API bug)
# 如果代码不报错:
market_value = 1000 * 0  # = 0元 (实际上是1000×50=50000元!)
# 持仓市值少了5万, 基于此计算的收益率也错了
# 交易员看到错误的组合数据 → 做出了错误的调仓决策

# 正确的做法:
if price <= 0:
    raise PriceFetchError(f"股票{code}价格异常: {price}")  # 直接报错, 不要静默
```

### 异常处理的分层设计

```python
# 第一层: 自定义异常 (语义化)
class DataValidationError(Exception):
    """数据问题 → 需要人工检查CSV文件"""
    pass

class PriceFetchError(Exception):
    """价格获取失败 → 可能是网络问题或API挂了"""
    pass

# 第二层: 底层函数抛出精确异常
def load_holdings(filepath):
    if not filepath.exists():
        raise FileNotFoundError(f"持仓文件不存在: {filepath}")  # Python内置异常
    df = pd.read_csv(filepath)
    if df.empty:
        raise DataValidationError("持仓文件为空")  # 自定义异常
    return df

# 第三层: 上层代码统一捕获和处理
try:
    total = calculator.calculate()
except DataValidationError as e:
    print(f"数据问题, 请检查CSV: {e}")
    sys.exit(1)  # 数据不对, 不应该继续计算
except PriceFetchError as e:
    print(f"价格获取失败, 请检查网络或切换数据源: {e}")
    sys.exit(2)
except Exception as e:
    print(f"未知错误: {e}")
    import traceback
    traceback.print_exc()  # 打印完整堆栈, 方便排查
    sys.exit(99)
```

### 面试官会追问的异常场景

**"baostock服务器宕机了怎么办?"**

代码行为: baostock SDK会抛连接异常 → 被 `_fetch_from_api` 捕获 → 转为 `PriceFetchError` → 上层报告用户 → 程序退出。

更好的设计: 如果有多个数据源(baostock + tushare), 可以在 `PriceFetcher` 里实现自动切换:
```python
def fetch_with_fallback(self, codes):
    for source in [self.fetch_via_baostock, self.fetch_via_tushare]:
        try:
            return source(codes)
        except Exception as e:
            print(f"数据源 {source.__name__} 失败: {e}")
    raise PriceFetchError("所有数据源均不可用")
```

**"如果某只股票退市了, 查不到价格怎么办?"**

持仓文件中可能有已退市的股票(历史遗留数据)。baostock不会返回该股票的数据。代码用 `left join`, 该股票的 `close` 为NaN:
- `market_value = hold_vol * NaN = NaN`
- `.sum()` 时NaN默认被当作0 (pandas的sum默认skipna=True)

注意: 退市股票的市值应该是0(无法交易), pandas的默认行为恰好符合预期。但需要报告给用户。

---

## 2.6 爬虫技术: 从上交所获取数据

### HTTP请求头为什么重要

```python
HEADERS = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 ...",
    "Referer": "https://www.sse.com.cn/",
}
```

**User-Agent (UA):**
- 告诉服务器"我是什么浏览器"
- 没有UA → 服务器知道你是脚本 → 可能拒绝服务
- 上交所API对UA检查相对宽松, 但加上保险

**Referer:**
- 告诉服务器"我从哪个页面来的"
- 上交所的API要求Referer必须是 `https://www.sse.com.cn/`, 否则返回空数据
- 这是最简单的反爬措施——防止其他网站直接调用API

**面试追问: "如果上交所加了更复杂的反爬怎么办?"**

答: "可以通过浏览器DevTools分析上交所页面的实际请求, 复制完整的请求头(包括Cookie)。如果使用了验证码, 可以考虑Selenium模拟浏览器, 或者换用第三方数据源如akshare(本质也是爬虫但社区维护更新及时)。"

### JSONP到底是什么

上交所API返回的不是纯JSON:

```javascript
// 返回的内容:
jsonpCallback12345({"result": [{"stockCode": "600000", ...}]})
```

这其实是一段JavaScript代码——调用一个叫 `jsonpCallback12345` 的函数, 传入JSON数据。

**为什么用JSONP?** 这是浏览器跨域请求的老方案。不是给你程序调用的, 是给网页JavaScript调用的。所以要解析必须先把函数调用剥掉:

```python
import re
import json

def parse_jsonp(text):
    # 正则提取 jsonpCallback(...) 括号内的JSON
    match = re.search(r"jsonpCallback\((.*)\)", text, re.DOTALL)
    if match:
        return json.loads(match.group(1))
    # 如果不是JSONP, 尝试直接解析
    return json.loads(text)
```

### akshare 的本质

akshare 不是独立的数据库, 它是对各大数据网站(新浪财经、东方财富、上交所等)的**爬虫聚合器**:

```python
import akshare as ak
# ak.index_stock_cons("000009") 内部做了什么:
# ① 构造URL: https://vip.stock.finance.sina.com.cn/...
# ② 发送HTTP请求
# ③ 解析返回的JSON/HTML
# ④ 转成 pandas DataFrame 返回
```

**优点**: 一行代码搞定, 不用写爬虫
**缺点**: 底层网站改版时akshare可能暂时不可用, 需要等更新

**为什么我的代码把akshare作为优先方案?**
因为akshare背后是新浪财经, 比上交所官网更稳定——上交所网站经常改版。

---

## 2.7 缓存设计: 为什么历史数据可以永久缓存

### 缓存策略的核心假设

**金融历史数据是immutable(不可变的)**: 2025年12月12日的收盘价, 在12月12日收盘后就是定值, 永远不变。

这与天气预报不同(今天预测明天的会变), 与实时股价不同(盘中每秒在变)。**收盘价是已经发生的历史事实。**

因此:
- 缓存不需要TTL(过期时间)
- 缓存不需要失效策略(除非数据本身有误)
- 缓存键 = 交易日期(同一天的数据不变)

### 缓存实现细节

```python
CACHE_DIR = Path(__file__).parent / ".cache"

def _get_cache_path(self) -> Path:
    # 缓存文件名: prices_20251212.csv
    self.CACHE_DIR.mkdir(exist_ok=True)  # 目录不存在就创建
    return self.CACHE_DIR / f"prices_{self.trade_date.replace('-', '')}.csv"

def _load_cache(self) -> pd.DataFrame | None:
    cache_path = self._get_cache_path()
    if cache_path.exists():
        df = pd.read_csv(cache_path, dtype={"code": str})
        if not df.empty:  # 文件存在但可能是空的(之前缓存失败)
            return df
    return None  # 返回None表示缓存未命中

def _save_cache(self, df: pd.DataFrame):
    cache_path = self._get_cache_path()
    df.to_csv(cache_path, index=False)  # index=False避免多出一列序号
```

### 面试追问: "如果缓存里的数据是错的怎么办?"

**操作层面**: 删除 `.cache/prices_20251212.csv` 重新运行即可。

**代码层面可以改进**: 加一个 `--no-cache` 命令行参数:
```python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('--no-cache', action='store_true', help='忽略缓存重新获取')
args = parser.parse_args()

if args.no_cache:
    # 删除缓存文件
    os.remove(cache_path)
```

---

# 3. 笔试题逐行代码精讲

## 3.1 题目1: staticmethod vs classmethod (概念题)

### 你应该背下来的标准回答 (2-3分钟版本)

"staticmethod和classmethod的核心区别在于**第一个参数**。

staticmethod没有隐式参数——它就是一个放在类命名空间里的普通函数, 调用时不会自动传入self或cls。它不能访问类属性, 也不能访问实例属性。典型用途是跟类业务相关的工具函数。

classmethod的第一个参数是cls, 代表调用它的类。关键特性是**在继承时cls指向子类**, 这使它成为实现工厂方法和多态构造的标准方式。

举个量化场景的例子: 写一个PriceFetcher类, `@staticmethod is_valid_code(code)` 判断股票代码格式是否正确(纯工具), `@classmethod from_config(cls, config)` 根据配置文件创建Fetcher实例(工厂方法, 子类可复用)。

性能上staticmethod略快(纳秒级), 但实际选哪个主要看语义——需不需要感知类。不需要就用staticmethod, 更纯粹。"

### 面试官可能的追问

**Q: "你刚才说staticmethod不能访问类属性, 那直接写ClassName.attr不就行了吗?"**

**你应该回答**: 技术上确实可以:
```python
class MyClass:
    value = 100

    @staticmethod
    def get_value():
        return MyClass.value  # 硬编码类名, 可以访问
```
但这样写的问题是**硬编码了类名**。如果子类继承了`MyClass`并覆盖了`value`, `get_value`返回的仍然是`MyClass.value`而不是子类的值。这就是静态方法不感知继承的体现。如果需求是"子类要能覆盖这个行为", 就该用classmethod。

**Q: "那classmethod是不是完胜staticmethod? 感觉功能更强?"**

**你应该回答**: 不是。classmethod多了一个参数, 调用时会多一层闭包调用。如果函数逻辑完全独立(不依赖类属性、不需要感知继承关系), 用staticmethod更干净——调用者不需要关心cls参数的存在。Python之禅: "Simple is better than complex."

---

## 3.2 题目2: 持仓市值计算 (核心题, 预计会被问最久)

### 3.2.1 整体架构解说 (面试画架构图用)

```
main.py (入口)
  │
  ├── question_1() → 纯文本输出, 无代码依赖
  │
  ├── question_2() → PortfolioCalculator
  │     ├── HoldingLoader.load()      → 加载CSV, 数据清洗
  │     ├── BaostockPriceFetcher.fetch() → 获取收盘价 (含缓存)
  │     └── PortfolioCalculator.calculate()
  │           ├── holdings.merge(prices, on='code', how='left')
  │           ├── market_value = hold_vol * close
  │           └── sum()
  │
  ├── question_3(calculator) → SSE380Crawler + IndexFilter
  │     └── 复用 calculator.portfolio (题目2的结果)
  │
  ├── question_4(calculator) → ReturnCalculator + CSI300Fetcher
  │     └── 复用 calculator.get_matched_portfolio()
  │
  └── question_5() → 纯文本输出
```

**为什么题目2的calculator要传给题目3和题目4?**
因为数据获取(I/O)已经完成, 缓存已建立, 不需要再重复查询API。这是对"计算结果复用"的设计考量。

### 3.2.2 HoldingLoader 逐行详解

```python
class HoldingLoader:
    """持仓数据加载器"""

    def __init__(self, filepath: str):
        self.filepath = Path(filepath)  # pathlib.Path: 跨平台路径处理
        # 为什么用Path而不是str: Path兼容Windows和Linux的路径分隔符

    def load(self) -> pd.DataFrame:
        # --- 步骤1: 文件存在性检查 ---
        if not self.filepath.exists():
            raise FileNotFoundError(f"持仓文件不存在: {self.filepath}")
        # 为什么先检查: 比pandas报错信息更友好; 且可以给出具体路径

        # --- 步骤2: 读取CSV ---
        df = pd.read_csv(
            self.filepath,
            dtype={"code": str, "hold_vol": int}
            # dtype的作用:
            # - code指定为str: 避免pandas把"000001"读成整数1
            # - hold_vol指定为int: 比默认float省内存(8字节→4或8字节, 但语义更准)
            # 不指定dtype的话, pandas会推断:
            #   "000001" → 整数1 (前导零丢失!), 因为pandas看到纯数字默认转int
        )

        # --- 步骤3: 数据清洗流水线 ---
        # 每步都是向量化操作, 没有for循环

        # 3.1 删除code或hold_vol为空的记录
        df = df.dropna(subset=["code", "hold_vol"])
        # subset参数: 只看这两列, 其他列有空也不影响
        # 为什么不是df.dropna(): 如果未来加了其他列有空值, 会误删有效行

        # 3.2 去重
        df = df.drop_duplicates(subset="code", keep="first")
        # keep='first': 保留第一次出现的, 删除后面的重复
        # 场景: CSV可能某只股票被录入了两次(人为错误)
        # 为什么不汇总: 因为不知道哪个hold_vol是正确的, 选择保守策略保留第一条

        # 3.3 过滤0持仓
        df = df[df["hold_vol"] > 0]
        # 布尔索引: 返回hold_vol>0的所有行
        # 为什么不是>=0: hold_vol=0意味着不持仓, 对市值贡献为0, 提前过滤省API调用

        # 3.4 代码标准化
        df["code"] = (
            df["code"]
            .str.strip()     # 去空格: " 000001" → "000001"
            .str.zfill(6)    # 补零至6位: "1" → "000001"; "000001" → "000001"(不变)
        )
        # 这是一个链式调用, pandas的.str访问器返回StringMethods对象
        # 注意: zfill(6)对已有6位的字符串不会改变, 是幂等操作

        # 3.5 正则过滤
        df = df[df["code"].str.fullmatch(r"\d{6}")]
        # fullmatch: 整串完全匹配, 不是部分匹配
        # \d{6}: 恰好6位数字
        # 为什么不用str.len()==6: 因为"ABC123".len()=6但不是纯数字
        # 过滤掉的可能: "N/A", "合计", "总计" 等CSV里的汇总行

        # --- 步骤4: 空DataFrame检查 ---
        if df.empty:
            raise DataValidationError("清洗后无有效持仓数据")
        # 如果文件只有标题行或全是无效数据, 不让程序继续(继续也会在上游报错)

        return df.reset_index(drop=True)
        # reset_index: 因为df可能被drop_duplicates等操作打乱了序号
        # drop=True: 不保留旧序号为新列
```

### 3.2.3 BaostockPriceFetcher 逐行详解

```python
class BaostockPriceFetcher(PriceFetcher):
    """通过baostock获取收盘价和昨收价"""

    CACHE_DIR = Path(__file__).parent / ".cache"
    # 类属性: 所有实例共享同一个缓存目录
    # 为什么不放在实例属性: 缓存目录是全局配置, 不应随实例变化

    @staticmethod
    def _to_bs_code(code: str) -> str:
        """将6位股票代码转为baostock查询格式"""
        code_int = int(code)
        if code_int >= 600000:
            return f"sh.{code}"
        return f"sz.{code}"
    # 为什么是static: 纯工具函数, 输入→输出, 不依赖任何类/实例状态
    # 为什么临界值是600000: 上海交易所代码从600000开始(主板)和688000(科创板)
    #   深圳的都在600000以下: 000xxx, 002xxx, 300xxx

    def _get_cache_path(self) -> Path:
        """获取缓存文件路径"""
        self.CACHE_DIR.mkdir(exist_ok=True)  # 目录不存在就创建
        return self.CACHE_DIR / f"prices_{self.trade_date.replace('-', '')}.csv"
        # 文件名示例: prices_20251212.csv

    def _load_cache(self) -> pd.DataFrame | None:
        """尝试加载缓存, 返回None表示缓存未命中"""
        cache_path = self._get_cache_path()
        if cache_path.exists():
            df = pd.read_csv(cache_path, dtype={"code": str})
            if not df.empty:  # 文件存在但可能为空
                return df
        return None
        # 返回类型: pd.DataFrame | None (Python 3.10+联合类型)

    def _save_cache(self, df: pd.DataFrame):
        """保存缓存"""
        cache_path = self._get_cache_path()
        df.to_csv(cache_path, index=False)  # index=False: 不保存行号列

    def _fetch_from_api(self, bs_codes: list) -> pd.DataFrame:
        """从baostock API获取数据"""
        import baostock as bs
        # 为什么延迟导入: baostock是第三方库, 可能未安装
        #   延迟导入让其他模块可以导入portfolio.py而不触发baostock依赖错误

        # --- 登录 ---
        lg = bs.login()
        if lg.error_code != "0":
            raise PriceFetchError(f"baostock登录失败: {lg.error_msg}")
        # baostock的login是匿名登录, 无需用户名密码
        # 错误码"0"表示成功; 非"0"可能原因: 网络不通、baostock服务器维护

        all_rows = []
        failed_count = 0

        try:
            for bs_code in bs_codes:
                # 这是唯一的for循环! baostock不支持批量查询
                try:
                    rs = bs.query_history_k_data_plus(
                        bs_code,
                        "date,code,close,preclose",  # 需要的字段
                        start_date=self.trade_date,    # 起始日期
                        end_date=self.trade_date,      # 结束日期(同一天=只查一天)
                        frequency="d",                 # 日线
                        adjustflag="3",                # 不复权
                    )
                    if rs.error_code == "0" and rs.next():
                        all_rows.append(rs.get_row_data())
                    else:
                        failed_count += 1
                        # 查不到数据的原因: 停牌、退市、新上市尚未有数据
                except Exception:
                    failed_count += 1
                    # 单只失败不影响整体, 继续查询下一只
        finally:
            bs.logout()
            # finally保证logout一定执行, 释放baostock连接资源

        # --- 异常检查 ---
        if failed_count:
            print(f"  {failed_count}只股票未查到数据(停牌/退市等)")

        if not all_rows:
            raise PriceFetchError(f"所有股票均未获取到 {self.trade_date} 的价格数据")
            # 如果一只都没查到: 很可能是日期错误(非交易日)或baostock全挂了

        # --- 构造DataFrame ---
        cols = ["date", "code", "close", "preclose"]
        df = pd.DataFrame(all_rows, columns=cols)

        # baostock返回的code格式: sh.600000, sz.000001
        df["code"] = df["code"].str.split(".").str[1]
        # 取"."后面的部分: "sh.600000" → "600000"

        # 价格转为数值
        df["close"] = pd.to_numeric(df["close"], errors="coerce")
        df["preclose"] = pd.to_numeric(df["preclose"], errors="coerce")
        # errors='coerce': 无法转换的值变成NaN(而不是报错)

        # 过滤无效价格
        df = df[(df["close"] > 0) & (df["preclose"] > 0)]
        # 价格为0或负数=数据异常; 价格为NaN的行也被过滤(pandas比较自动排除NaN)

        return df.reset_index(drop=True)

    def fetch(self, codes: pd.Series) -> pd.DataFrame:
        """获取收盘价, 优先使用缓存"""
        # --- Step 1: 尝试缓存 ---
        cached = self._load_cache()
        if cached is not None:
            print(f"  从缓存加载价格数据 ({len(cached)} 条), 耗时<1秒")
            return cached

        # --- Step 2: 缓存未命中, 调用API ---
        unique_codes = codes.drop_duplicates().values
        # 去重: 如果持仓文件有重复代码, 只查一次(虽然前面清洗已去重, 这里做二次保险)
        # .values: 返回numpy数组, 速度比Series快

        print(f"  首次获取, 正在通过baostock查询 {len(unique_codes)} 只股票...")
        bs_codes = [self._to_bs_code(c) for c in unique_codes]
        df = self._fetch_from_api(bs_codes)

        # --- Step 3: 保存缓存 ---
        self._save_cache(df)
        print(f"  已缓存价格数据到本地, 后续运行秒级加载")

        return df
```

### 3.2.4 PortfolioCalculator 逐行详解

```python
class PortfolioCalculator:
    """持仓市值计算器 - 核心类"""

    def __init__(
        self,
        holdings_file: str,
        trade_date: str,
        price_fetcher: PriceFetcher = None,
    ):
        self.holdings_file = holdings_file
        self.trade_date = trade_date
        self.loader = HoldingLoader(holdings_file)  # 组合关系: Calculator拥有Loader
        self.price_fetcher = price_fetcher or BaostockPriceFetcher(trade_date)
        # 依赖注入: 可以传入自定义的price_fetcher(如mock用于测试)
        # 如果不传, 默认用BaostockPriceFetcher
        self.holdings = None   # 清洗后的持仓
        self.prices = None     # 获取的价格
        self.portfolio = None  # 合并后的完整持仓(含价格和市值)

    def calculate(self) -> float:
        """
        计算持仓总市值
        返回: 总市值(元), 精确到分
        """
        t0 = time.time()  # 计时开始

        # --- Step 1: 加载持仓 ---
        self.holdings = self.loader.load()
        print(f"  已加载 {len(self.holdings)} 条持仓记录")

        # --- Step 2: 获取价格 ---
        self.prices = self.price_fetcher.fetch(self.holdings["code"])
        # 传入所有股票代码, fetcher内部会去重
        print(f"  已获取 {len(self.prices)} 条有效价格数据")

        # --- Step 3: 合并数据 (向量化) ---
        self.portfolio = self.holdings.merge(
            self.prices,
            on="code",       # 用code列作为合并键
            how="left"       # LEFT JOIN: 保留所有持仓
        )
        # merge是向量化的Hash Join, 不是嵌套循环

        # --- Step 4: 检查未匹配的股票 ---
        unmatched_mask = self.portfolio["close"].isna()
        # isna(): 判断close是否NaN → 返回布尔Series
        # 这里是向量化操作: 一次检查整列

        if unmatched_mask.any():
            print(f"  警告: {unmatched_mask.sum()}只股票未匹配到价格")
            # 如果面试官问为什么这些没价格:
            # - 停牌: 当天停牌的股票没有交易, 也就没有收盘价
            # - 退市: 已退市的股票baostock不再提供数据
            # - 数据源问题: baostock的数据可能不完整

        # --- Step 5: 向量化计算市值 ---
        # 核心计算: 就两行, 0个for循环
        self.portfolio["market_value"] = (
            self.portfolio["hold_vol"].fillna(0)   # 防止hold_vol为NaN
            * self.portfolio["close"].fillna(0)     # close为NaN → 市值=0
        )
        # fillna(0): NaN表示查不到价格, 市值应为0(无法交易)
        # 向量化乘法: pandas整列相乘 → 底层numpy的C实现

        total = self.portfolio["market_value"].sum()
        # sum(): 底层是numpy.sum(), C实现, 跳过NaN(但我们已经fillna(0)了)

        elapsed = time.time() - t0
        print(f"  总耗时: {elapsed:.2f} 秒")

        return round(total, 2)
        # round(, 2): 精确到分; 避免浮点精度问题
        # 例如: 2101495.099999999 → 2101495.10

    def get_detail(self) -> pd.DataFrame:
        """获取持仓明细"""
        if self.portfolio is None:
            self.calculate()  # 懒加载: 没算过就自动算
        cols = ["code", "hold_vol", "close", "preclose", "market_value"]
        return self.portfolio[cols].copy()
        # .copy(): 返回副本, 防止外部修改影响内部数据

    def get_matched_portfolio(self) -> pd.DataFrame:
        """获取有价格数据的持仓(排除未匹配的)"""
        if self.portfolio is None:
            self.calculate()
        return self.portfolio[self.portfolio["close"].notna()].copy()
        # notna(): close不为NaN → 只保留有价格的
        # 题目3和题目4用这个方法, 因为它们依赖价格数据
```

### 3.2.5 核验逻辑

```python
# main.py 中的核验代码:
target = 2101495.10
total_value = calculator.calculate()

diff = total_value - 2101495.10
if abs(diff) < 0.01:  # 允许1分钱误差
    print("PASS")
else:
    print(f"偏差: {diff:,.2f}")

# 为什么允许0.01误差:
# - 浮点精度: float累加可能产生微小误差(如0.00000000001)
# - 不同数据源: 同一天同一只股票的收盘价, baostock可能和其他源有微妙差异
# - 四舍五入: 中间计算可能有round差异
# 0.01(1分钱)是最合理的容差——大于这个说明有实质性差异, 需要排查
```

### 3.2.6 时间分析 (面试必须张口就来)

| 环节 | 首次运行耗时 | 缓存命中耗时 | 主要时间花在哪里 |
|------|------------|------------|---------------|
| CSV加载 | ~0.05s | ~0.05s | 磁盘I/O |
| 数据清洗 | ~0.01s | ~0.01s | 向量化, 秒完成 |
| 价格获取 | ~55s | ~0.3s | 600+次HTTP请求(每次~90ms) / 读本地CSV |
| 数据合并 | ~0.01s | ~0.01s | Hash Join O(n+m) |
| 市值计算 | ~0.005s | ~0.005s | 向量化乘法和sum |
| **总计** | **~60s** | **~0.4s** | |

**关键洞察**: 首次运行时99.9%的时间花在网络I/O上, 不是计算上。Python的向量化计算部分(<0.02s)完全满足"秒级"要求。第二次以后直接<1秒。

如果面试官说"首次60秒太慢", 你应该说三件事:
1. 这是baostock单线程查询的物理限制(每个请求~90ms×600只=54秒)
2. 可以用多线程优化到~10-15秒(10-20线程)
3. 但笔试题允许首次"获取数据"的时间, "秒级"指的是计算——计算部分<0.02s

---

## 3.3 题目3: 上证380成份股筛选

### 3.3.1 SSE380Crawler 逐行详解

```python
class SSE380Crawler:
    """上证380指数成份股获取器"""

    INDEX_CODE = "000009"  # 上证380的指数代码
    # 注意: 这是上交所指数代码, 不是股票代码
    # 000009在股票体系里也是存在的(中国宝安, 深市), 但在指数体系里是上证380

    HEADERS = {
        "User-Agent": (
            "Mozilla/5.0 (Windows NT 10.0; Win64; x64) "
            "AppleWebKit/537.36 (KHTML, like Gecko) "
            "Chrome/120.0.0.0 Safari/537.36"
        ),
        # 伪装成Chrome 120浏览器(2024年最新版)
        # 为什么要伪装: 部分网站拒绝非浏览器的HTTP请求
    }

    def __init__(self, index_code: str = None):
        self.index_code = index_code or self.INDEX_CODE
        # 允许传入其他指数代码(如"000010"上证180), 复用同一个类
        self.constituents = None  # 延迟获取

    def fetch(self) -> pd.DataFrame:
        """获取上证380成份股列表, 多方案回退"""
        # 方案A: akshare (新浪财经数据) - 最稳定
        try:
            return self._fetch_via_akshare()
        except Exception as e:
            print(f"  akshare获取失败 ({e}), 尝试上交所页面爬取...")

        # 方案B: 上交所API (JSONP接口)
        try:
            return self._fetch_via_sse_page()
        except Exception:
            pass  # 失败不抛, 等最后统一抛

        # 如果所有方案都失败了
        raise RuntimeError("无法获取上证380成份股数据, 请检查网络或安装akshare")

    def _fetch_via_akshare(self) -> pd.DataFrame:
        """通过akshare获取"""
        import akshare as ak

        df = ak.index_stock_cons(symbol=self.index_code)
        # akshare内部调用新浪财经的API:
        # http://vip.stock.finance.sina.com.cn/quotes_service/api/json_v2.php/...

        if df.empty:
            raise ValueError("akshare返回空数据")

        # 兼容不同akshare版本的列名
        code_col = (
            "品种代码" if "品种代码" in df.columns
            else "stock_code" if "stock_code" in df.columns
            else df.columns[0]  # 兜底: 取第一列
        )
        # 为什么兼容: akshare不同版本的列名可能不同

        codes = (
            df[code_col]
            .astype(str)       # 确保是字符串
            .str.strip()       # 去空格
            .str.zfill(6)      # 补零至6位
            .tolist()          # 转Python列表
        )

        self.constituents = codes
        print(f"  已获取上证380成份股 {len(self.constituents)} 只 (数据来源: akshare/新浪财经)")
        return pd.DataFrame({"code": self.constituents})

    def _fetch_via_sse_page(self) -> pd.DataFrame:
        """上交所官网爬虫(回退方案)"""
        # 方案B-1: 上交所JSONP API
        api_urls = [
            f"https://query.sse.com.cn/security/stock/indexStockList.do"
            f"?jsonCallBack=jsonpCallback&indexCode={self.index_code}&pageNo=1&pageSize=500",
            # 参数说明:
            # - jsonCallBack: JSONP回调函数名
            # - indexCode: 指数代码(000009=上证380)
            # - pageNo=1: 第一页
            # - pageSize=500: 每页500条(380只<500, 一页就够)
        ]

        for url in api_urls:
            try:
                headers = {
                    **self.HEADERS,
                    "Referer": "https://www.sse.com.cn/"
                    # 关键! 上交所API检查Referer, 不设会返回空
                }
                resp = requests.get(url, headers=headers, timeout=30)
                # timeout=30: 30秒超时, 防止永久等待

                if resp.status_code == 200:
                    codes = self._parse_jsonp_response(resp.text)
                    if codes:
                        self.constituents = codes
                        return pd.DataFrame({"code": codes})
            except Exception:
                continue  # 这个URL失败, 尝试下一个

        raise RuntimeError("所有API端点均无法访问")

    def _parse_jsonp_response(self, text: str) -> list:
        """解析JSONP响应"""
        # text 格式: jsonpCallback({"result": [...]});
        match = re.search(r"jsonpCallback\((.*)\)", text, re.DOTALL)
        # DOTALL: . 匹配换行符 (JSON可能跨多行)

        if match:
            data = json.loads(match.group(1))  # 提取括号内的JSON
        else:
            data = json.loads(text)  # 可能不是JSONP, 直接当JSON解析

        # 上交所返回格式: {"result": [{"stockCode": "600000", ...}, ...]}
        stocks = data.get("result", []) or data.get("data", []) or []

        codes = []
        for s in stocks:
            if isinstance(s, dict):
                code = (
                    s.get("stockCode")      # 尝试各种可能的键名
                    or s.get("SECURITY_CODE")
                    or s.get("code")
                    or ""
                )
                if code:
                    codes.append(str(code).strip().zfill(6))

        return codes
```

### 3.3.2 IndexFilter 详解

```python
class IndexFilter:
    """指数过滤器"""

    @staticmethod
    def filter_portfolio(
        portfolio: pd.DataFrame,    # 持仓数据(含价格和市值)
        index_codes: pd.DataFrame   # 指数成份股代码(只有code列)
    ) -> pd.DataFrame:
        """
        筛选在指数成份股中的持仓
        返回: 匹配的持仓行
        """
        return portfolio.merge(index_codes, on="code", how="inner")
        # inner join: 只保留两个DataFrame都有的code
        # 复杂度: O(n+m) Hash Join
        # 结果: 持仓中有、也在指数成份股中的股票

    @staticmethod
    def calculate_market_value(matched: pd.DataFrame) -> float:
        """计算匹配持仓的市值总和"""
        if "market_value" not in matched.columns:
            # 如果没有预先计算market_value, 现场算
            matched = matched.copy()
            matched["market_value"] = matched["hold_vol"] * matched["close"]
        return round(matched["market_value"].sum(), 2)

    # 为什么都是staticmethod:
    # 这两个方法不依赖任何类/实例状态, 是纯数据处理
    # 输入→输出, 无副作用, 放在类里只是为了组织代码
```

### 3.3.3 面试关键问题

**"上证380只有上海股票, 如果持仓里有深圳股票, inner join会自动过滤掉吗?"**

是的。比如持仓中有 `000001`(平安银行, 深圳), 上证380成份股列表里肯定没有它。inner join会自动把这个股票排除。这就是 inner join 的正确用法——只要在两个集合都存在的才保留。

**"怎么验证上证380成份股列表是正确的?"**

可以抽查几只: 比如 `600000`(浦发银行)应该在上证380, `000001`(平安银行)不应该在。你也可以写一个验证逻辑:

```python
# 验证上证380成份股都在上海交易所
assert all(code.startswith('6') for code in sse380_codes['code']), \
    "错误: 上证380成份股包含非上海交易所股票!"
```

---

## 3.4 题目4: 日收益与超额收益

### 3.4.1 ReturnCalculator 逐行详解

```python
class ReturnCalculator:
    """持仓收益计算器"""

    def __init__(self, portfolio: pd.DataFrame, csi300_return: float = None):
        """
        Args:
            portfolio: 含 code, hold_vol, close, preclose 的DataFrame
            csi300_return: 沪深300当日涨跌幅(%)
                       如 0.5 表示 +0.5%, -1.2 表示 -1.2%
                       如果为None, 则不计算超额收益
        """
        self.portfolio = portfolio.copy()
        # copy(): 防止外部修改原始数据影响计算结果
        self.csi300_return = csi300_return
        self.daily_pnl = None
        self.daily_return = None
        self.excess_return = None

    def calculate(self) -> dict:
        """执行收益计算"""

        # --- Step 1: 过滤有效数据 ---
        valid = self.portfolio[
            self.portfolio["close"].notna() & self.portfolio["preclose"].notna()
        ]
        # 需要close和preclose都不为NaN才能计算收益
        # 如果close或preclose任一缺失, 该股票不参与收益计算
        # 为什么: 收益需要 (close - preclose), 缺一个都算不了

        if valid.empty:
            raise ValueError("无有效价格数据, 无法计算收益")

        # --- Step 2: 向量化计算日收益金额 ---
        # PnL = Σ(hold_vol × (close - preclose))
        valid["pnl"] = valid["hold_vol"] * (valid["close"] - valid["preclose"])
        # 逐行PnL: 每只股票贡献的盈亏
        # 正数=赚钱, 负数=亏钱
        # 这是向量化操作: 整列运算, 一次完成

        self.daily_pnl = round(valid["pnl"].sum(), 2)
        # sum()求和, round精确到分

        # --- Step 3: 向量化计算日收益率 ---
        # Return = PnL / Σ(hold_vol × preclose) × 100%
        valid["prev_value"] = valid["hold_vol"] * valid["preclose"]
        # prev_value = 该股票昨天的市值

        prev_total = valid["prev_value"].sum()
        # 昨天总市值 = 分母

        if prev_total == 0:
            raise ValueError("昨收市值为0, 无法计算收益率")
            # 理论上不会出现(所有preclose都>0才是正常的)
            # 这是防御性编程: 万一出现异常数据, 报错而非返回inf

        self.daily_return = round(self.daily_pnl / prev_total * 100, 4)
        # ×100: 从小数转百分比
        # round(, 4): 保留4位小数 (如 1.2345%), 对应金额精确到分

        # --- Step 4: 计算超额收益 ---
        if self.csi300_return is not None:
            self.excess_return = round(self.daily_return - self.csi300_return, 4)
            # Alpha = 组合收益 - 基准收益
        else:
            self.excess_return = None  # 没有基准数据时无法计算

        return {
            "daily_pnl": self.daily_pnl,
            "daily_return_pct": self.daily_return,
            "csi300_return_pct": self.csi300_return,
            "excess_return_pct": self.excess_return,
        }
```

### 3.4.2 CSI300Fetcher 逐行详解

```python
class CSI300Fetcher:
    """获取沪深300指数涨跌幅"""

    @staticmethod
    def fetch_return(trade_date: str) -> float:
        """
        获取沪深300在指定交易日的涨跌幅(%)

        沪深300指数代码: sh.000300 (baostock格式)
        注意: 000300在baostock中是sh.000300, 不是sz.000300
              虽然000300不遵循"6开头=上海"的规则, 但沪深300是跨市场指数,
              中证指数公司将其归在上海市场数据中
        """
        import baostock as bs

        lg = bs.login()
        if lg.error_code != "0":
            raise RuntimeError(f"baostock登录失败: {lg.error_msg}")

        try:
            rs = bs.query_history_k_data_plus(
                "sh.000300",                          # 指数代码
                "date,close,preclose,pctChg",         # pctChg: 涨跌幅(%)
                start_date=trade_date,
                end_date=trade_date,
                frequency="d",
                adjustflag="1",                       # 指数用后复权(其实指数不存在除权问题)
            )

            data_list = []
            while (rs.error_code == "0") and rs.next():
                data_list.append(rs.get_row_data())

            if not data_list:
                raise ValueError(f"未获取到沪深300在{trade_date}的数据")

            row = data_list[0]
            pct_chg = float(row[3]) if len(row) > 3 else 0.0
            # row[3]是pctChg字段: baostock的涨跌幅列
            # 已经是百分比形式, 如 0.5 表示涨0.5%

            return pct_chg

        finally:
            bs.logout()
```

### 3.4.3 计算公式完整推导

面试官可能会让你在白板上推一遍。你应该这么说:

**第一步: 计算每只股票的单日盈亏**

```
单只股票PnL = 持有股数 × (今日收盘 − 昨日收盘)

例: 持有200股平安银行(000001)
    昨收=11.37, 今收=11.35
    PnL = 200 × (11.35 − 11.37) = 200 × (−0.02) = −4.00元
    (当天亏了4块钱)
```

**第二步: 加总所有股票的PnL**

```
总PnL = Σ 单只股票PnL
      = Σ[ hold_vol_i × (close_i − preclose_i) ]
```

**第三步: 计算昨收总市值(分母)**

```
昨收总市值 = Σ(持有股数 × 昨日收盘价)
          = Σ[ hold_vol_i × preclose_i ]
```

**第四步: 计算日收益率**

```
日收益率 = 总PnL / 昨收总市值 × 100%

如果总PnL = +5,234.56元
昨收总市值 = 2,096,260.54元
日收益率 = 5,234.56 / 2,096,260.54 × 100% = +0.2497%
```

**第五步: 计算超额收益**

```
超额收益 = 组合日收益率 − 沪深300日收益率

假设沪深300当天涨跌幅 = +0.1523%
超额收益 = 0.2497% − 0.1523% = +0.0974%
(组合跑赢大盘约0.1%)
```

### 3.4.4 面试官的刁钻追问

**"沪深300涨跌幅是+0.5%, 你的组合涨跌幅是+13%, 超额收益+12.5%。这个结果合理吗?"**

答: "这不太合理。一个持仓600多只股票的组合, 日收益率高达13%, 说明组合集中度极高或者加了杠杆。正常情况下600多只股票的分散组合, 日波动应该在±2%以内。如果真有13%的日收益, 应该检查:
1. preclose数据是否正确
2. 是否有股票当天涨幅异常(如新股上市首日不限涨跌幅)
3. 持仓量数据是否有误(比如hold_vol填的是金额而不是股数)"

**"如果沪深300当天停牌或者数据缺失, 你的代码怎么处理?"**

答: "CSI300Fetcher会抛出异常, 被上层question_4捕获, 设置csi300_return=None。然后ReturnCalculator检测到csi300_return为None, 就不计算超额收益, 只计算持仓自身的收益率。程序不会崩溃, 只是超额收益那一栏显示N/A。"

**"如果有股票前复权除权除息日正好是12月12日, 你的收益率计算会有什么问题?"**

答: "如果12月12日是除权除息日:
- preclose是除权前价格(比如10元)
- close是除权后价格(比如9.5元, 除息0.5元)
- (close - preclose) = -0.5, 看起来是跌了5%, 实际上投资者没亏钱(收到了0.5元分红)
- 这会导致收益率被低估

但要精确处理需要获取复权因子, 在单日收益计算中通常忽略(影响很小)。最严谨的做法是用后复权价格的涨跌幅来替代。"

---

## 3.5 题目5: 代码开发全生命流程

### 结合本题的完整SDLC讲述 (面试标准回答)

"我来以这道笔试题的开发过程为例, 完整讲述软件开发生命周期。"

**Phase 1: 需求分析 (1天)**

```
输入: 笔试题PDF
分析产出:
  - 功能点拆解:
    ① Q1: 纯输出(无计算)
    ② Q2: CSV→清洗→获取价格→计算市值→核验
    ③ Q3: 爬虫→筛选→计算子集市值
    ④ Q4: 收益金额→收益率→超额收益
    ⑤ Q5: 纯输出(无计算)
  - 约束条件:
    · 使用Python类
    · 向量化计算(禁用for循环)
    · 秒级响应
    · 核验目标值 2,101,495.10
  - 风险评估:
    · baostock可能数据不全
    · 上交所网站可能改版导致爬虫失效
    · CSV可能有脏数据
```

**Phase 2: 系统设计 (1-2天)**

```
架构决策:
  - 模块划分: portfolio / sse380_crawler / returns / main
    理由: 每个模块对应一个独立功能域, 可独立开发、测试、替换

  - 数据流:
    CSV → HoldingLoader(清洗) → pd.DataFrame
    → BaostockPriceFetcher(获取价格) → pd.DataFrame
    → pd.merge(left join) → 含价格的持仓表
    → 向量化计算 → 市值/收益

  - 缓存设计:
    .cache/prices_YYYYMMDD.csv
    理由: 历史收盘价不变, 永久缓存

  - 依赖管理:
    pandas, numpy, baostock, requests, akshare, beautifulsoup4
    → 写入 requirements.txt

  - 错误处理策略:
    分层异常: DataValidationError / PriceFetchError
    数据源回退: akshare → 上交所API → 页面解析
```

**Phase 3: 编码开发 (2-3天)**

```
开发顺序(按依赖关系):
  1. portfolio.py (HoldingLoader + BaostockPriceFetcher + PortfolioCalculator)
     → Q2依赖它
  2. sse380_crawler.py (SSE380Crawler + IndexFilter)
     → Q3依赖它 + Q2的结果
  3. returns.py (ReturnCalculator + CSI300Fetcher)
     → Q4依赖它 + Q2的结果
  4. main.py (整合+Q1/Q5文本输出)

编码规范:
  - PEP8: 4空格缩进, 每行≤120字符
  - 类型注解: pd.DataFrame, str, float, Path等
  - 注释: 每个公共方法有docstring, 关键参数有行内说明
  - 所有数值计算使用pandas向量化, 无Python层for循环
```

**Phase 4: 测试 (1天)**

```
测试策略:
  1. 单元测试(未写, 但可以说出怎么测):
     test_holding_loader.py:
       - test_load_valid_csv: 正常CSV加载
       - test_load_empty_csv: 空文件
       - test_load_missing_columns: 缺少列
       - test_duplicate_removal: 重复代码
       - test_zero_holding_filter: 0持仓过滤
       - test_code_normalization: 代码标准化

     test_price_fetcher.py:
       - test_cache_hit: 缓存命中
       - test_cache_miss: 缓存未命中, 调API
       - test_empty_api_response: API返回空

     test_portfolio_calculator.py:
       - test_market_value_calculation: 已知数据验证
       - test_unmatched_stocks: 有股票查不到价格

  2. 集成测试:
     python main.py → 全流程执行

  3. 核验测试(最关键):
     结果 == 2,101,495.10 → PASS/FAIL
     这是笔试题自带的"验收测试"
```

**Phase 5: 部署发布 (0.5天)**

```
部署步骤:
  1. 确保依赖安装: pip install -r requirements.txt
  2. 确保路径正确:
     所有路径使用 pathlib.Path(__file__).parent 相对路径
     支持解压后直接运行, 无需修改任何配置
  3. Windows兼容:
     sys.stdout.reconfigure(encoding='utf-8') 解决中文乱码
  4. 打包:
     包含 20251212.csv + 所有.py + requirements.txt
     → portfolio_project.zip
  5. 打包后验证: 解压到新目录 → pip install → python main.py → 验证结果
```

**Phase 6: 运维迭代 (长期)**

```
持续改进方向:
  - 数据源: 从单baostock → 多源(baostock + tushare + akshare)自动切换
  - 性能: API查询从单线程 → 多线程并发
  - 监控: 加入日志系统(loguru/structlog)替代print
  - 扩展: 支持任意日期(不只是12月12日) → 命令行参数
  - 测试: 补充单元测试和CI/CD
  - 安全: 敏感信息(API token)从环境变量读取而非硬编码
```

---

# 4. 面试官100问

## 4.1 量化基础类

**Q1: "什么是量化交易? 用一句话概括。"**
A: 用数据和模型替代主观判断来做投资决策, 覆盖「数据→信号→组合→执行→监控」全流程。

**Q2: "持仓市值怎么算?"**
A: 每只股票的持有股数 × 当日收盘价, 再求和。A股的收盘价是交易所公布的日终集合竞价价格。

**Q3: "收益率和超额收益的区别?"**
A: 收益率衡量绝对回报(自己赚了多少), 超额收益衡量相对回报(比市场多赚或少赚了多少)。专业投资者更关注后者。

**Q4: "为什么收益率的分母用昨收市值?"**
A: 收益率 = (今天市值 − 昨天市值) / 昨天市值。分母用昨天市值衡量的是"我昨天持有的资产到今天增值了多少"。如果用今天市值做分母数学上不对。

**Q5: "A股代码体系你怎么记?"**
A: 6开头=上海(600主板+688科创板), 0和3开头=深圳(000/001/002主板+300/301创业板)。baostock格式: 6开头加sh., 其余加sz.。

**Q6: "上证380和沪深300有什么区别?"**
A: 上证380是上交所380只新兴蓝筹(二线蓝筹), 沪深300是跨两市300只大盘蓝筹。沪深300是业绩基准, 上证380是中盘成长代表。

**Q7: "不复权和后复权什么时候分别用?"**
A: 计算当前市值用不复权(实际成交价), 计算跨期收益率用后复权(消除除权除息干扰)。本题单日收益率因除权影响极小, 所以用不复权也正确。

**Q8: "如果某只股票停牌了, 你的市值计算怎么处理?"**
A: 停牌股票没有当日收盘价, baostock不返回数据。merge后close为NaN, fillna(0)后市值贡献为0。程序会警告有多少只股票未匹配, 不静默丢弃。

**Q9: "什么是Alpha?"**
A: Alpha(α)是超出市场基准的收益, 衡量的是投资者"选股/择时"的真本事。α = 组合收益 − 基准收益(简化版)。理论上α应该独立于市场波动(β分离后)。

**Q10: "12月12日如果不是交易日怎么办?"**
A: 2025年12月12日是周五, 正常交易日。如果确实不是交易日, baostock不会返回数据, 程序会在获取价格阶段抛出PriceFetchError。

---

## 4.2 Python/编程类

**Q11: "你真的没有用任何for循环吗?"**
A: 数值计算部分完全没有Python层for循环——全部使用pandas向量化(整列运算)和pd.merge(Hash Join)。唯一的for循环在baostock API查询(I/O操作, 非计算), 因为API不支持批量查询。

**Q12: "pd.merge内部用了什么算法?"**
A: Hash Join。Build Phase对右表建哈希表O(m), Probe Phase遍历左表O(n)查找, 总复杂度O(n+m), 不是嵌套循环的O(n×m)。

**Q13: "向量化为啥比for循环快?"**
A: 三个原因: ①运算在NumPy的C层执行, 无Python解释器开销; ②数据连续内存布局, CPU缓存友好; ③可利用CPU的SIMD指令一次处理多个数据。

**Q14: "你用left join而不用inner join计算市值, 为什么?"**
A: Left join保留所有持仓, 没价格的就NaN。这让我能统计"有多少只股票未匹配到价格"并报告用户。Inner join会悄悄丢弃这些股票, 在量化中数据缺失不能被静默忽略。

**Q15: "dtype指定了code为str, 如果CSV里code本来就是数字会怎样?"**
A: pandas会把它转成字符串, 保留前导零的语义。如果不指定dtype, pandas会把"000001"读成整数1, 丢失前导零——这是P0级别的错误。

**Q16: "数据清洗为什么zfill(6)而不是zfill(7)?"**
A: 中国A股代码标准是6位数字, 没有7位的。基金代码也是6位。zfill(6)是语义正确的标准化操作。

**Q17: "异常处理为什么不用一个except Exception?"**
A: 不同异常需要不同处理: 文件不存在→提示用户; 网络超时→重试或回退; 数据格式错误→报告问题行。用自定义异常(DataValidationError/PriceFetchError)让错误语义明确, 方便上层针对性处理。

**Q18: "为什么缓存历史收盘价不设过期时间?"**
A: 历史收盘价是不可变数据——昨天的收盘价永远是那个值, 不会变。这与实时行情不同(需要频繁更新)。所以缓存可以永久保存, 除非发现数据本身有误。

**Q19: "你的代码里哪些是static方法, 为什么选static?"**
A: `_to_bs_code()`(代码格式转换), `IndexFilter.filter_portfolio()`(数据筛选)等都是静态方法。因为它们不依赖任何类或实例状态, 是纯函数(输入→输出)。staticmethod让这个语义更清晰——"我不需要self, 也不需要cls"。

**Q20: "你的代码里哪些是classmethod, 为什么选class?"**
A: 本题代码中没有classmethod的实际使用(因为不是工厂模式场景)。但示例中的`from_tushare()`演示了典型用途——工厂方法。如果未来需要从不同数据源创建Calculator, 就会用classmethod确保子类也正确工作。

**Q21: "requirements.txt里为什么指定>=而不是==?"**
A: `>=`允许小版本升级(如bug修复), 同时保持兼容性。`==`会锁死版本, 在新环境可能因为依赖冲突安装失败。对于笔试题, `>=`是更健壮的选择——除非已知某个特定版本有bug。

**Q22: "你为什么延迟导入baostock?"**
A: 放在函数内部导入而非文件顶部。好处: ①如果用户只导入某个不依赖baostock的模块, 不会触发导入错误; ②可以检测baostock是否已安装, 给出友好的错误提示。

**Q23: "pathlib.Path和os.path有什么区别?"**
A: Path是面向对象的路径处理, 支持 `/` 操作符拼接, 跨平台自动处理分隔符。os.path是面向过程的字符串操作。Path是现代Python的标准做法。

**Q24: "pd.to_numeric的errors='coerce'是什么意思?"**
A: 如果值无法转为数字(比如字符串'停牌'), 转换为NaN而不是抛异常。这保证了程序不会因为个别数据问题而崩溃, 同时NaN在后续计算中会被排除。

**Q25: "你的merge操作时间复杂度是多少?"**
A: O(n+m), 其中n是持仓数(618), m是价格数据条数。Hash Join的Build O(m) + Probe O(n)。如果是嵌套循环, 将是O(n×m) = 381,924次比较。

---

## 4.3 代码设计类

**Q26: "为什么拆成三个类(HoldingLoader, PriceFetcher, PortfolioCalculator)?"**
A: 单一职责原则。三个类分别对应三个独立变化的维度: 数据格式、数据来源、业务计算。如果换了数据源(如从baostock换tushare), 只需改/替换PriceFetcher, 其他两个类完全不用动。

**Q27: "PriceFetcher为什么设计为基类?"**
A: 面向接口编程。PortfolioCalculator依赖的是PriceFetcher抽象接口, 而不是具体的BaostockPriceFetcher。这样: ①可以替换数据源; ②可以mock用于测试; ③未来新增数据源只需写一个新的子类。

**Q28: "你的代码中有依赖注入吗? 在哪里?"**
A: PortfolioCalculator的`__init__`接受可选的`price_fetcher`参数。这是构造器注入——调用的代码可以传入自定义的fetcher, 如果不传则使用默认的BaostockPriceFetcher。这样灵活且可测试。

**Q29: "如果让你给这个代码写单元测试, 你会怎么设计?"**
A: 使用Mock对象替代baostock API, 返回固定价格数据; 构造包含各种边界情况的CSV(重复代码、0持仓、空值等); 对每个public方法分别测试正常路径和异常路径; 验证计算结果与预期一致。

**Q30: "你的代码有哪些可以改进的地方?"**
A: ①API查询加入多线程并发(减少首次运行时间); ②加入命令行参数(如--date, --no-cache); ③用logging替代print; ④加入数据源自动切换(baostock→tushare); ⑤补充单元测试; ⑥加入数据完整性检查(如验证持仓文件编码)。

**Q31: "为什么要写print报告未匹配的股票数量?"**
A: 在量化系统中, 静默的数据丢失是最危险的。如果5只股票没匹配到价格而被当0处理, 但用户不知道, 可能会导致错误的交易决策。显式报告让用户知道"有N只股票数据异常"。

**Q32: "两个文件main.py和portfolio_project/main.py内容一样吗?"**
A: `portfolio_project/main.py` 是打包后的入口, `main_20260429182504.py` 是打包前的开发版本。确认后内容一致。

**Q33: "如果持仓文件路径写死了, 怎么支持任意路径?"**
A: 应该使用 `Path(__file__).parent / "20251212.csv"` 相对路径(当前已实现)。也可以用argparse接受命令行参数。

**Q34: "你怎么保证代码在Windows和Linux上都能跑?"**
A: pathlib.Path自动处理路径分隔符; sys.stdout.reconfigure仅Windows执行; 所有文本操作使用UTF-8; requirements.txt不加平台限制。

---

## 4.4 数据源/爬虫类

**Q35: "为什么选baostock而不是tushare?"**
A: baostock免费且无需注册token, tushare免费版有频率限制且需要注册。但代码设计上PriceFetcher可以替换, 如果团队统一用tushare, 替换成本很低。

**Q36: "baostock的adjustflag参数你选了'3', 为什么?"**
A: adjustflag='3'是不复权, 返回实际成交价格。计算市值必须用实际价格, 因为市值=股数×实际成交价。复权价格不是交易所的真实成交价, 算出的市值会偏离账户实际资产。

**Q37: "akshare和直接爬虫有什么区别?"**
A: akshare是社区维护的爬虫封装库, 背后也是爬的新浪财经/东方财富等网站。优点是稳定(社区持续更新适配), 一行代码搞定。缺点是依赖第三方, 如果akshare出问题需要等更新。直接爬虫更可控但维护成本高。"

**Q38: "上交所反爬怎么处理?"**
A: 设置User-Agent伪装浏览器, 设置Referer满足防盗链要求, 设置timeout防止永久等待。如果遇到验证码, 建议回退到akshare而非硬闯(笔试题场景下不涉及验证码)。

**Q39: "什么是JSONP? 为什么上交所返回JSONP?"**
A: JSONP(JSON with Padding)是跨域Ajax的老方案——把JSON数据包在JavaScript函数调用里。上交所网站前端的JavaScript通过动态插入script标签来跨域获取数据, 绕开浏览器的同源策略。解析时需要正则提取括号内的JSON部分。

**Q40: "爬虫的三个回退方案分别是什么?"**
A: ①akshare(新浪财经API, 最稳定); ②上交所JSONP API(官方接口, 但可能改版); ③上交所网页HTML解析(最后兜底)。这种"优雅降级"确保只要有一个渠道可用, 程序就能运行。

---

## 4.5 金融计算类

**Q41: "市值计算中, 如果某股票没有收盘价怎么处理?"**
A: left join后close为NaN → fillna(0) → 市值贡献为0 → 程序警告用户"有N只股票未匹配"。不会让整个计算失败。

**Q42: "日收益率用了百分比还是小数?"**
A: 代码中存的是百分比(如1.5表示+1.5%), 输出时加%号。这是金融行业的习惯——说"收益率1.5%"比"收益率0.015"更直观。

**Q43: "如果昨收市值是0, 收益率怎么算?"**
A: 收益率=PnL/昨收市值, 除以0 → 无穷大。代码会先检查prev_total==0并抛出ValueError。这种情况理论上不应该发生(除非所有持仓都是第一天买入且preclose全为0)。

**Q44: "你用的是简单收益率还是对数收益率?"**
A: 简单收益率: (close-preclose)/preclose。因为单日波动小, 简单收益率和对数收益率近似相等。且简单收益率在业界沟通中更直观("今天涨了1.5%")。

**Q45: "超额收益假设β=1, 这个假设合理吗?"**
A: 这道题里的简单超额(α=组合收益−基准收益)假设β=1。对于600多只分散持仓的组合, β≈1是合理的近似。精确的α需要通过回归估计β, 然后用α=Rp−β×Rm。涉及更多统计学内容, 超出笔试题范围。

**Q46: "你的计算中有考虑交易费用吗?"**
A: 没有。题目问的是"持仓的日收益", 即持仓的浮动盈亏, 不涉及实际买卖。交易费用只在实现盈亏(卖出时)才产生。

**Q47: "什么是preclose? 跟close有什么区别?"**
A: close是当日收盘价, preclose是昨日收盘价。baostock提供的preclose是"上一交易日的收盘价", 不是"日历昨天"——如果中间有周末或节假日, preclose自动取最近交易日。这个细节由baostock处理好了。

**Q48: "沪深300涨跌幅你从哪里获取的?"**
A: baostock直接提供pctChg字段(涨跌幅百分比), 不需要自己用(preclose-close)/preclose计算。直接用官方计算的涨跌幅更权威。

---

## 4.6 SDLC/流程类 (题目5)

**Q49: "代码开发全生命流程有哪些阶段?"**
A: 需求分析→系统设计→编码开发→测试(单元+集成)→部署上线→运维迭代。6-7个阶段, 与SDLC标准一致。

**Q50: "你的开发流程中最重要的阶段是什么?"**
A: 需求分析。如果没有正确理解"向量化、秒级、2101495.10"这些约束, 后面全部推倒重来。需求分析阶段也决定了技术选型(baostock vs tushare)和架构设计(类拆分)。

**Q51: "你在本项目中的测试策略?"**
A: ①单元测试(各模块类的独立测试, mock掉外部依赖); ②集成测试(完整跑main.py); ③核验测试(自动对比2101495.10)。核验测试是最关键的——它是笔试题的"验收标准"。

**Q52: "部署时要注意什么?"**
A: ①依赖安装(pip install -r requirements.txt); ②路径可用(相对路径, 解压即用); ③Windows兼容(UTF-8编码); ④首次运行需要网络(获取价格数据)。

**Q53: "版本管理你怎么做?"**
A: Git管理代码, 每个功能独立分支开发(feature/xxx), 完成后合并到main, 打Tag标记版本。在团队中, 每个分支提交前进行Code Review。

**Q54: "你怎么保证代码质量?"**
A: ①PEP8代码风格; ②类型注解; ③异常处理分层; ④打印关键步骤日志; ⑤向量化优先; ⑥自定义异常让错误可追踪。额外的应该有lint工具(flake8/pylint)和单元测试。

---

## 4.7 刁钻追问/边界情况

**Q55: "如果持仓文件编码是GBK, 你的代码会怎样?"**
A: pandas read_csv默认用系统编码。在Windows中文系统上是GBK, 能正常读取。但如果文件是UTF-8且没有BOM, 在Windows上可能乱码。解决方案: `pd.read_csv(filepath, encoding='utf-8')` 或先检测编码(chardet库)。

**Q56: "如果持仓文件有100列, 但只有code和hold_vol有用, 你的代码受影响吗?"**
A: 不受影响。pd.read_csv会读所有列, 但后续代码只使用code和hold_vol列。除非那100列导致内存问题(但618行×100列, 内存完全可以忽略)。

**Q57: "你的代码中round(total, 2)会不会有舍入误差?"**
A: 可能有微小的浮点舍入差异(如2101495.099999999→2101495.10)。这也是为什么核验使用abs(diff)<0.01而不是==。这是IEEE 754浮点数的固有限制。

**Q58: "如果baostock返回的价格数据中有一只股票close=0, 你的代码怎么处理?"**
A: `_fetch_from_api`中有 `df[(df["close"] > 0) & (df["preclose"] > 0)]`, 价格为0的会被过滤掉。然后在merge时会表现为未匹配(left join的NaN), 被报告给用户。

**Q59: "停牌的股票有preclose吗?"**
A: 有。停牌只是当天没有交易, 但昨天的收盘价是存在的。所以preclose可能有但close没有。在收益计算中, 这样的股票被valid过滤(需要close和preclose都不为NaN), 不参与收益计算。

**Q60: "你的代码在大规模持仓(如5000只股票)时还能秒级吗?"**
A: 缓存命中后可以。计算部分(merge+向量化)对5000只股票仍然<0.1秒。首次运行会更慢(5000次API查询×90ms=450秒), 但计算本身不是瓶颈。

**Q61: "如果baostock的API改了返回格式, 你的代码会怎样?"**
A: 会抛出异常(IndexError或KeyError), 因为 `rs.get_row_data()` 返回的列表结构变了。补救: 需要在 `_fetch_from_api` 中做更灵活的数据解析, 或使用try/except包裹字段访问。

**Q62: "你的爬虫如果被上交所封IP怎么办?"**
A: 笔试题场景下单次请求不会触发封IP(封IP需要高频请求)。但如果发生, 可以: ①换用akshare(不直接请求上交所); ②加延迟(time.sleep); ③使用代理IP。

**Q63: "为什么pandas的sum()默认跳过NaN? 这正确吗?"**
A: sum(skipna=True, 默认值)会跳过NaN, 相当于NaN被当作0。在市值计算中, 如果某股票close=NaN(未匹配), 该股票的市值贡献确实是0。这个默认行为是正确的。但需要在日志中显式报告"有未匹配的股票"。

**Q64: "你把hold_vol定为int, 那如果有人持仓100.5股呢?"**
A: A股最小交易单位是1股(不是手), 但实际交易中确实可以出现非整数(如送股产生零股)。代码中dtype指定为int, pandas会让100.5变成100(截断)或报错。更健壮的做法是改为float。

**Q65: "你的代码在Python 3.8上能跑吗?"**
A: 大部分可以, 但 `pd.DataFrame | None` 类型注解需要Python 3.10+。如果要在3.8上跑, 需要把类型注解改为 `Optional[pd.DataFrame]`。

---

# 5. 面试模拟对话

## 5.1 开场: "简单介绍一下这道题的代码架构"

**你应该这样说(2-3分钟):**

"好的。我的代码采用模块化设计, 分为四个模块:

**第一个模块portfolio.py** 负责持仓市值计算。里面有HoldingLoader类负责CSV加载和数据清洗(5步清洗流水线), BaostockPriceFetcher类通过baostock API获取收盘价(含本地缓存), PortfolioCalculator是主控类, 用pd.merge做left join合并持仓和价格, 然后向量化计算市值。

**第二个模块sse380_crawler.py** 负责获取上证380成份股。SSE380Crawler有三层回退: 优先用akshare(新浪财经), 失败则爬上交所API, 再失败解析HTML。IndexFilter用inner join筛选在成份股中的持仓并计算市值。

**第三个模块returns.py** 负责收益计算。ReturnCalculator计算日PnL、日收益率和超额收益, 全部向量化。CSI300Fetcher从baostock获取沪深300涨跌幅。

**第四个模块main.py** 是入口, 依次执行5道笔试题。题目2算出的portfolio传给题目3和题目4复用, 避免重复查询API。

**设计理念**: 单一职责(每类一个功能), 依赖注入(PriceFetcher可替换), 向量化优先(无Python for循环计算), 优雅降级(爬虫多方案回退)。

**性能**: 首次运行约60秒(网络I/O), 缓存命中后<1秒。计算部分<0.02秒。"

## 5.2 追问: "你觉得你的代码最大的亮点是什么?"

**你应该这样说:**

"我最大的亮点有三个:

**第一, 缓存策略。** 历史收盘价是不可变数据, 我用交易日期做缓存键永久缓存。首次60秒, 之后<1秒。这是刻意设计的——牺牲首次运行时间换取后续极速体验。面试时我已经提前运行过, 缓存已建立。

**第二, 数据源的'优雅降级'。** 爬虫部分有三层回退, 从akshare到上交所API到HTML解析。量化系统的生命线是数据, 不能因为一个数据源挂了就报错退出。这个设计在真实生产环境中非常重要。

**第三, 核验机制。** 代码自动对比2,101,495.10, 输出PASS/FAIL。这比手动肉眼对比可靠得多。量化系统中所有关键数据都应该有校验——不信任任何数据源, 不信任任何计算结果。"

## 5.3 追问: "如果让你重构, 你会改什么?"

**你应该这样说:**

"我会改三个方面:

**短期(30分钟能做的):**
- 加命令行参数: `--date 20251212 --no-cache`
- 用logging替代print, 设置INFO/WARNING/ERROR级别
- 加数据完整性验证: 上证380成份股必须是6开头的

**中期(半天能做的):**
- API查询改为ThreadPoolExecutor多线程(首次从60秒降到15秒)
- 加入tushare作为备选数据源, 在PriceFetcher中自动切换
- 补充单元测试(至少覆盖数据清洗和市值计算)

**长期(架构级):**
- 用配置驱动(config.yaml)替代硬编码的参数(如数据源类型、缓存路径)
- 加入pandas性能监控(如用%%timeit对比不同操作)
- 考虑用DuckDB/Polars替代pandas(在某些大数据场景下更快)"

## 5.4 追问: "你的代码有没有不符合笔试题要求的地方?"

**你应该诚实回答:**

"有一处: `_fetch_from_api`中有for循环遍历每只股票查API。但笔试题要求的不使用for循环, 我理解为计算逻辑部分不能用for循环——实际上baostock API不支持批量查询, 这个for循环是I/O操作, 不是数值计算。如果面试官认为这也不符合要求, 我会改为多线程并发查询。但需要注意baostock的线程安全性——需要每个线程独立的login/logout。"

## 5.5 追问: "如果你来面试量化开发, 你觉得候选人应该具备什么?"

**你应该这样说:**

"我认为量化开发最核心的能力有三个:

**第一, 数据敏感性。** 股票代码'000001'被pandas读成整数1怎么办? 价格为0的股票如何处理? 停牌股的数据缺失怎么报告? 这些问题技术上简单, 但意识上关键——量化系统的问题是数据问题, 不是算法问题。

**第二, 工程化思维。** 类怎么拆分、异常怎么分层、缓存怎么设计、数据源怎么切换——这些不是算法问题, 是软件工程问题。量化代码最终要上生产环境, 不能是Jupyter Notebook的一次性脚本。

**第三, 金融直觉。** 为什么市值用不复权? 为什么收益率分母用昨收市值? 为什么超额收益要对标沪深300? 这些是金融逻辑, 不是编程技巧。技术是实现手段, 金融理解是方向。"

---

## 附录A: 关键数字速查

| 项目 | 数值 |
|------|------|
| 持仓股票数 | 618只 |
| 目标市值 | 2,101,495.10 元 |
| 上证380成份股数 | 约380只 |
| 沪深300成份股数 | 300只 |
| 上证380指数代码 | 000009 |
| 沪深300指数代码 | sh.000300 |
| 交易日期 | 2025-12-12 (周五) |
| baostock不复权参数 | adjustflag='3' |
| 首次运行耗时 | ~60秒 (网络I/O) |
| 缓存命中耗时 | <1秒 |
| 向量化计算耗时 | <0.02秒 |
| 核验容差 | 0.01元 (1分钱) |

## 附录B: 公式合集 (面试速记)

```
① 持仓总市值
   M = Σ( hold_vol_i × close_i )

② 日收益金额
   PnL = Σ[ hold_vol_i × (close_i − preclose_i) ]

③ 日收益率
   R = PnL / Σ( hold_vol_i × preclose_i ) × 100%

④ 超额收益 (Alpha)
   α = R_portfolio − R_CSI300

⑤ 向量化乘法 (pandas)
   df['market_value'] = df['hold_vol'] * df['close']

⑥ Hash Join (pd.merge)
   时间复杂度: O(n+m), 空间复杂度: O(n+m)
```

---

*本文档专为零基础入门量化面试准备而写。建议配合代码阅读, 每个概念都对着源码找到对应位置, 理解"代码为什么这么写"。面试时自信表达——你已经理解了方案背后的每一个"为什么"。*
