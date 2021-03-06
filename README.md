# okex BTC/USDT交割合约逐仓模式爆仓价格计算器

计算okex的合约爆仓价格，因全仓模式的保证金率涉及到已实现盈亏(数据量比较复杂)，所以暂时先开发逐仓模式的计算。

## 参数
| 名称 | 参数 | 参数类型 | 示例 | 默认值 | 描述 | 
| ---- | ---- | ---- | ---- | ---- | ---- |
| 保证金类型 | --mgnType | `int` | 0 | 0 | `0` PA：币本位, `1` BS：美元本位 |
| 挂单手续费 | --makerFee | `float` | 0.02 | 0.02 | 百分比，[详情参考](https://www.okex.com/fees.html) |
| 吃单手续费 | --takerFee | `float` | 0.05 | 0.05 | 百分比，[详情参考](https://www.okex.com/fees.html) |
| 购入价格 | --buyPrice | `float` | 6666.66 | 100 | 合约成交价 |
| 购入张数 | --buyCount | `float` | 100 | 1 | 购入量 |
| 杠杆 | --lever | `int` | 10 | 1 | 杠杆 |
| 持仓方向 | --posSide | `string` | 'long' | 'long' | 持仓方向，`long`(做多)或者`short`(做空) |
| 最新标记价格 | --lastMarkPrice | `float` | 55000.0 | 0 | 标记价格 |

## 示例
币本位做多：
```
python main.py --mgnType 0 --buyPrice 10000 --buyCount 100 --lever 10 --posSide 'long' --lastMarkPrice 10000
```
结果：
```
=========================================
币本位爆仓价格计算
购入价格: 10000.000000, 购入张数: 100, 杠杆倍数: 10, 持仓方向: long, 最新标记价格: 10000.000000
当前档位: 1, 档位维持保证金率: 0.400000%, 最低初始保证金率: 0.800000%
=========================================
固定保证金为：0.100000BTC(1000.000000USDT)
未实现盈亏为：0.000000BTC(0.000000UDST)
保证金率为: 10.000000%
-----------------------------------------
当标记价格等于或小于: 9131.818182时
未实现盈亏为：-0.095072BTC(-868.181818USDT)
平仓手续费为：0.000548BTC(5.000000USDT)
档位最低维持保证金为：0.004380BTC(40.000000USDT)
原先固定保证金为: 0.100000BTC(913.181818USDT)
当前保证金(加上未实现盈亏和减去平仓手续费)为：0.004380BTC(40.000000USDT)
保证金率为: 0.450000%, 等于或小于当前档位维持保证金率+平仓手续费率0.450000%触发爆仓
-----------------------------------------
```

## 计算公式

### 币本位

**逐仓保证金率：**
```
面值: 100USD

固定保证金 = 面值 * 张数 / (购入价格 * 杠杆倍数)

保证金率 = (固定保证金 + 未实现盈亏) / (面值 * 张数 / 最新标记价格)

未实现盈亏 (买入开多) = (面值 * 张数 / 开仓均价 - 面值 * 张数 / 最新标记价格) * 持仓量

未实现盈亏 (卖出开空) = (面值 * 张数 / 最新标记价格 - 面值 * 张数 / 开仓均价) * 持仓量
```

**举例**
```
假设当前BTC价格为$10000，用户选择逐仓模式，10倍杠杆，开多1BTC

对应张数为100张，处于档位1，维持保证金率 = 1%，平仓手续费率 = 0.075%

保证金 = 面值 * 张数 / (BTC价格 * 杠杆倍数) = 100 * 100 / (10000 * 10) = 0.1BTC

此时，用户初始保证金率 = 1/10 = 10%

当BTC最新标记价格下跌至$9150

未实现盈亏 = 面值 * 张数 / 开仓均价 - 面值 * 张数 / 最新标记价格
          = 100 * 100 / 10000 - 100 * 100 / 9150
          = -0.0929

此时，保证金率 = (固定保证金 + 未实现盈亏) / 仓位价值
              = (0.1 - 0.0929) / (100 * 100 / 9150)
              = 0.0071 / 1.0929
              = 0.64% 
              < 维持保证金率 + 平仓手续费率 = 1.075%，此时，仓位触发爆仓
```

### USDT本位

**逐仓保证金率：**
```
面值: 0.01BTC

固定保证金 = 面值 * 张数 * 购入价格 / 杠杆倍数

保证金率 = (固定保证金 + 未实现盈亏) / (面值 * 张数 * 最新标记价格)

未实现盈亏 (买入开多) = (面值 * 张数 * 最新标记价格 - 面值 * 张数 * 结算基准价) * 持仓量

未实现盈亏 (卖出开空) = (面值 * 张数 * 结算基准价 - 面值 * 张数 * 最新标记价格) * 持仓量
```

**举例**

```
假设当前BTC价格为10000USDT/BTC，用户选择逐仓模式，10倍杠杆，开多1BTC
对应张数为10000张，处于档位3，维持保证金率 = 1.50%，平仓手续费率 = 0.05%

保证金 = 面值 * 张数 * BTC价格 / 杠杆倍数
       = 0.0001 * 10000 * 10000 / 10 = 1000USDT

此时，用户初始保证金率 = 1 / 10 = 10%

当BTC最新标记价格下跌至9010 USDT/BTC

未实现盈亏 = 面值 * 张数 * 最新标记价格 - 面值 * 张数 * 开仓均价
          = 0.0001 * 10000 * 9010 - 0.0001 * 100 * 10000
          = -990USDT

此时，保证金率 = (固定保证金 + 未实现盈亏) / 仓位价值
              = (1000 - 990) / (0.0001 * 10000 * 9010)
              = 10 / 9010
              = 0.11% < 维持保证金率+平仓手续费率 = 1.55%，此时，仓位触发强制平仓。
```

```
合约未实现盈亏：

买入：合约未实现盈亏 = (合约面值 * 最新标记价格 – 合约面值*结算基准价) * 持仓量

例如某用户以结算基准价500 USDT/BTC 买入开多600张BTC合约
现在最新标记价格为600 USDT/BTC
则合约未实现盈亏 = (0.0001 * 1 * 600 - 0.0001 * 1 * 500) * 600 = 6USDT

卖出：合约未实现盈亏 = (合约面值 * 结算基准价 - 合约面值 * 最新标记价格) * 持仓量

例如某用户以结算基准价1000 USDT/BTC 卖出开空1000张BTC合约
现在最新标记价格为500USDT/BTC
则合约已实现盈亏 = (0.0001 * 1 * 1000 — 0.0001 * 1 * 500) * 1000 = 50USDT
```