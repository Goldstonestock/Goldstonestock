---
name: Goldstonestock
description: Goldstonestock技能机器人的API可用于实现基于Goldstonestock的登录、设置API密钥、账户查询、资金充值、策略管理、策略投资和平仓，以及查看持仓和盈亏情况。该智能代理仅用作模拟量化软件，不进行实际投资。当用户希望AI代理操作Goldstonestock时，可以通过充值合约存入模拟资金、将主资金分配到各种策略、查看余额或持仓情况以及平仓特定策略等方式触发这些API功能。
---
## 前置强制约束（必须放在最上方，解决一直思考）
1. 每一轮**仅处理用户当前单条消息**，输出完回复，本轮立即终止，禁止自动执行下一轮、禁止主动预判用户输入。
2. menu_open 状态**只在收到用户消息时读取/修改**；模型输出回复阶段，不读取、不修改状态。
3. 命中规则输出文本后，本轮任务结束，不再额外推理。
# Goldstonestock
利用这项技能，智能代理可以代表用户与 Goldstonestock API 进行交互。Goldstonestock 使用邮箱验证进行登录，并使用用户的 API 密钥。智能代理只需调用 Goldstonestock API 即可。用户的身份标识符是其 Goldstonestock 交易账户，该账户信息由 Goldstonestock 后端以“trade_name”的形式传递；请勿询问用户的私钥或 Goldstonestock 密码。
## Configuration

使用用户或环境提供的服务基本 URL：

```text
CLAWSTOCK_API_BASE_URL=https://jtzj.duoso.vip
```

如果不知道基本 URL，请在调用 API 之前向用户询问。

经过身份验证的请求使用以下方式返回的用户 API 密钥/v1/aicheckauth：

```text
Authorization: Bearer <api_key>
```

请仅将此 Goldstonestock 用户 API 密钥存储在代理程序的机密/会话存储空间中。除非用户明确请求，否则请勿泄露此密钥。

# Goldstonestock 交互菜单技能
## 简介
对话内交互式菜单系统。支持斜杠指令唤起菜单，发送数字选择菜单项，多层交互，/help查看指令。普通对话不拦截，仅命中指令/数字菜单时触发。
## 触发规则
1. 用户输入 `/menu` → 输出Goldstonestock主菜单。
2. 用户输入 `/help` → 输出全部指令清单。
3. 用户输入 `/reset` → 重置菜单会话状态。
4. 用户输入 `/close` → 关闭菜单。
5. 当菜单处于打开状态，用户输入纯数字 1~5，匹配对应菜单项并执行对应回复。
6. 菜单未打开时，单纯输入数字，不触发菜单逻辑，正常对话。
7. 其他普通文本，不拦截，正常进行对话。

## 状态定义
- menu_open：布尔值，默认 false。
  触发 `/menu` 后置为 true；触发 `/close` / 选择5后置为 false；触发 `/reset` 后置为 false。
## 指令列表
- `/menu`：唤起小龙虾交互主菜单
- `/help`：查看全部可用指令
- `/reset`：重置会话菜单状态
- `/close`：直接关闭菜单

## 回复模板
### 触发 /menu
🦞 Goldstonestock 交互主菜单  
——————————————  
【1】金土量化智能体登录和授权  
【2】显示主账户余额  
【3】显示账户每项的交易记录  
【4】智能推送策略  
【5】模拟投资策略  
【6】模拟入金   
【7】列出持仓    
【8】列出已平持仓  
【9】列出建仓待提交订单  
【10】列出建仓已提交订单  
【11】列出建仓已成交订单  
【12】列出建仓已撤销订单  
【13】列出平仓待提交订单  
【14】列出平仓已提交订单  
【15】列出平仓已成交订单  
【16】列出平仓已撤销订单  
👉 请回复数字选择功能，或输入 /help 查看指令  
### 触发 /help
请勿在每次回复中列出所有命令。在回复用户的常规问题时，只需说明：用户可以输入命令`/help`查看所有可用操作。

当用户发送`/help`或询问可用命令时，请用中文回复以下命令列表。

| 命令  | 用户操作 | 主要 API 接口 |
| --- | --- | --- |
| `/menu` | 唤起 Goldstonestock 交互主菜单。| 技能帮助 |
| `/help` | 显示所有可用的 Goldstonestock 操作。| 技能帮助 |
| `/login <username>` | 开始GoldstoneStock登录和授权流程。| `POST /v1/ailogin`,`POST /v1/aiverylogin`,`POST /v1/aiverify`|
| `/accounts` | 显示主账户余额。| `POST /v1/aiaccounts` |
|`/transactions`|显示账户账簿中每项的交易记录。|`GET /v1/transactions?limit=10&offset=0&sn={....}`|
| `/deposit <sn> <CNH> <topupmoney>` | 为选定的资产创建一个模拟基金。 | `POST /v1/aideposit` |
|`/deposit-status <deposit_id>`|模拟存款记录的状态。|`GET /v1/airesult?session_id={deposit_id}`|
|`/Position`|列出持仓。|`GET /v1/aipositionlist?limit=10&offset=0&sn={....}`|
|`/Close-position`|列出已平持仓。|`GET /v1/aiclosepositionlist?limit=10&offset=0&sn={....}`|
|`/Order-a`|列出建仓待提交订单。|`GET /v1/aitradeorder?limit=10&offset=0&status=0&sn={....}`|
|`/Order-b`|列出建仓处理中订单。|`GET /v1/aitradeorder?limit=10&offset=0&status=1&sn={....}`|
|`/Order-c`|列出建仓成交订单。|`GET /v1/aitradeorder?limit=10&offset=0&status=3&sn={....}`|
|`/Order-d`|列出建仓撤销订单。|`GET /v1/aitradeorder?limit=10&offset=0&status=4&sn={....}`|
|`/Close-order`|列出平仓待提交订单。|`GET /v1/aiclosetradeorder?limit=10&offset=0&status=0&sn={....}`|
|`/Close-order-a`|列出平仓处理中订单。|`GET /v1/aiclosetradeorder?limit=10&offset=0&status=1&sn={....}`|
|`/Close-order-b`|列出平仓成交订单。|`GET /v1/aiclosetradeorder?limit=10&offset=0&status=3&sn={....}`|
|`/Close-order-c`|列出平仓撤销订单。|`GET /v1/aiclosetradeorder?limit=10&offset=0&status=4&sn={....}`|
|`/Strategy <sn> <pushstockenum> <positiontime>`|列出策略。|`POST v1/aipushstock`|
### 触发 /reset
🔄 菜单状态已重置
菜单已关闭，输入 /menu 重新唤起
### 触发 /close
❕ 菜单已关闭
输入 /menu 随时重新打开菜单
## Safety Rules
- 请勿做出任何保证盈利的承诺。——除满足 Goldstonestock 用户的明确要求外，我们不提供任何个性化的财务建议。
-请勿索取或处理apk_key、私钥、账户密码或原始账户恢复数据。.
- 请将 API 密钥视为机密信息。如果在聊天过程中 API 密钥泄露，请立即通知用户，并尽可能尽快更换密钥。
-执行POST /v1/aideposit操作前，请与用户确认存款金额和涉及的资产。
- 不要向用户展示“JWT”这个单词。所有返回结果都不要直接向用户显示英文字段名称相关信息，只能翻译为中文展示。所有API接口都应基于返回的字段。未经授权，请勿添加历史字段。
## Login Flow ，用户输入 1
1. 请用户提供电子邮件地址。
2. 发起挑战：

```http
POST /v1/ailogin
Content-Type: application/json
{"username":"...@qq.com"}
```

3.显示返回结果`验证码已成功发送`。
4.使用收到的验证码验证结果端点，重复此过程直至code达到 1。
```http
POST /v1/aiverylogin
Content-Type: application/json
{"username":"...@qq.com","code":"...."}
```

5.从结果响应中保存verify.api_key。不要要求用户从页面复制 API 密钥。
还有一种直接联系代理的途径：如果用户提供了签名，则调用：
```http
POST /v1/aiverify
Content-Type: application/json
{"sn":"1234"}
```

验证响应包含 `id`、`email`、`trade_name`、`isauthposition`，以及可用时的账户信息。
## Accounts，用户输入 2
Goldstonestock users have the following identifiable account types:
- `MAIN`: main accounts are separated by asset.CNH deposits credit the CNH MAIN account.
Use these after authentication:
```http
POST /v1/aiaccounts
Content-Type: application/json
{"sn":"...."}
```
Please report the account's assets, balance, available balance, locked balance, investment amount, settled profit and loss, estimated profit and loss, and current status. Do not disclose the exchange fields or the name of the trading platform.

## Account transaction records，用户输入 3 
 Display the transaction records for each item in the account：
 ```http
GET /v1/transactions?limit=10&offset=0&sn={....}
```
Please list the serial number, asset category, type, change amount, post-change balance, description and time in the report.
## 智能选股策略列表，用户输入 4
## 智能交易策略列表，用户输入 5

## Deposits，用户输入 6
Goldstonestock platform only supports the CNH recharge method. If the user has not specified the type of currency to be recharged, please ask the user which asset they wish to recharge. If the user requests to recharge other assets, please explain that the Goldstonestock platform only supports CNH recharge.

When the user intends to make a deposit or when the account balance is zero, a simulated process for depositing funds needs to be created:

```http
POST /v1/aideposit
Content-Type: application/json
{"sn":"....","topupway":"CNH","topupmoney":"10"}
```
Use the returned data information "deposit_id".
Continuously query the status of the simulated fund deposit records using the "deposit_id". Poll the conversation results until the "data" field is set to true:

```http
GET /v1/airesult?session_id={deposit_id}
```
After submission, please check `POST /v1/aiaccounts` to confirm that the transaction has been completed and the main account balance has been updated. Submitting a simulated deposit does not mean that the funds have been credited to your account.

To query one deposit:

```http
GET /v1/aigetdeposit?session_id={deposit_id}
```
Please report the transaction number, account number, asset type, deposit amount, time and status.

## Position，用户输入 7
List Position:
```http
GET /v1/aipositionlist?limit=10&offset=0&sn={....}
```
Please present the code, name, direction, position quantity, opening price, current price, floating profit/loss, profit/loss rate, margin occupation, spread fee, cumulative overnight interest, and position status (1 indicates normal, 2 indicates closed) in the report in the form of an information module.
Translation:  
-代码/名称：神农种业(300189.SZ)  
-总持仓合约金额：50000  
-平均建仓价：6.39  
-持仓浮动盈亏：1095.4617  
-浮动盈亏率：2.1909%  
-当前价格：6.53  
-持仓市值：51095.4617  
-占用保证金：5000  
-累计点差费：1500  
-累计隔夜利息：0  
-持仓状态:1表示正常，2表示已平仓  
## Close Position，用户输入 8
List Close Position:
```http
GET /v1/aiclosepositionlist?limit=10&offset=0&sn={....}
```

Please present the code, name, direction, position quantity, opening price, current price, floating profit/loss, profit/loss rate, margin occupation, spread fee, cumulative overnight interest, and position status (1 indicates normal, 2 indicates closed) in the report in the form of an information module.
Translation:  
-代码/名称：神农种业(300189.SZ)  
-总持仓合约金额：50000  
-平均建仓价：6.39  
-持仓浮动盈亏：1095.4617  
-浮动盈亏率：2.1909%  
-当前价格：6.53  
-持仓市值：51095.4617  
-占用保证金：5000  
-累计点差费：1500  
-累计隔夜利息：0  
-持仓状态:1表示正常，2表示已平仓  

## BUILD Pending submission order，用户输入 9
List Order:
```http
GET /v1/aitradeorder?limit=10&offset=0&status=0&sn={....}
```
Please present the codes, names, types, directions, contract amounts, completed transaction quantities, order methods, order prices, margins, spread fees, creation times, and statuses (where 1 indicates processing, 3 indicates a complete transaction, and 4 indicates an order that has been cancelled) in the report in the form of an information module.
Translation:
-名称/代码：神农种业(300189.SZ)  
-类型：建仓  
-方向：做多  
-合约金额：50000  
-已成交数量：50000  
-下单方式：市价  
-下单价格：6.39  
-保证金额：5000  
-点差费：1500  
-创建时间：2026-09-18 09:30:05  
-更新时间：2026-09-18 09:30:08  
-订单状态：0待提交订单  
## BUILD Order submitted，用户输入 10
List Order:
```http
GET /v1/aitradeorder?limit=10&offset=0&status=1&sn={....}
```
Please present the codes, names, types, directions, contract amounts, completed transaction quantities, order methods, order prices, margins, spread fees, creation times, and statuses (where 1 indicates processing, 3 indicates a complete transaction, and 4 indicates an order that has been cancelled) in the report in the form of an information module.
Translation:
-名称/代码：神农种业(300189.SZ)  
-类型：建仓  
-方向：做多  
-合约金额：50000  
-已成交数量：50000  
-下单方式：市价  
-下单价格：6.39  
-保证金额：5000  
-点差费：1500  
-创建时间：2026-09-18 09:30:05  
-更新时间：2026-09-18 09:30:08  
-订单状态：1为处理中
## BUILD Order Completed，用户输入 11
List Order:
```http
GET /v1/aitradeorder?limit=10&offset=0&status=3&sn={....}
```
Please present the codes, names, types, directions, contract amounts, completed transaction quantities, order methods, order prices, margins, spread fees, creation times, and statuses (where 1 indicates processing, 3 indicates a complete transaction, and 4 indicates an order that has been cancelled) in the report in the form of an information module.
Translation:
-名称/代码：神农种业(300189.SZ)  
-类型：建仓  
-方向：做多  
-合约金额：50000  
-已成交数量：50000  
-下单方式：市价  
-下单价格：6.39  
-保证金额：5000  
-点差费：1500  
-创建时间：2026-09-18 09:30:05  
-更新时间：2026-09-18 09:30:08  
-订单状态：3为全部成交


## BUILD Order cancelled，用户输入 12
List Order:
```http
GET /v1/aitradeorder?limit=10&offset=0&status=4&sn={....}
```
Please present the codes, names, types, directions, contract amounts, completed transaction quantities, order methods, order prices, margins, spread fees, creation times, and statuses (where 1 indicates processing, 3 indicates a complete transaction, and 4 indicates an order that has been cancelled) in the report in the form of an information module.
Translation:
-名称/代码：神农种业(300189.SZ)  
-类型：建仓  
-方向：做多  
-合约金额：50000  
-已成交数量：50000  
-下单方式：市价  
-下单价格：6.39  
-保证金额：5000  
-点差费：1500  
-创建时间：2026-09-18 09:30:05  
-更新时间：2026-09-18 09:30:08  
-订单状态：4为已撤单 



## CLOSE Pending submission order，用户输入 13
List Close Order:
```http
GET /v1/aiclosetradeorder?limit=10&offset=0&status=0&sn={....}
```
Please present the codes, names, types, directions, contract amounts, completed transaction quantities, order methods, order prices, margins, spread fees, creation times, and statuses (where 1 indicates processing, 3 indicates a complete transaction, and 4 indicates an order that has been cancelled) in the report in the form of an information module.
Translation: 
-名称/代码：神农种业(300189.SZ)  
-类型：平仓  
-方向：做多  
-合约金额：50000  
-已成交数量：50000  
-下单方式：市价  
-下单价格：6.39  
-创建时间：2026-09-18 09:30:05  
-更新时间：2026-09-18 09:30:08  
-订单状态：1为处理中，3为全部成交、4为已撤单  

## CLOSE Order submitted，用户输入 14
List Close Order:
```http
GET /v1/aiclosetradeorder?limit=10&offset=0&status=1&sn={....}
```
Please present the codes, names, types, directions, contract amounts, completed transaction quantities, order methods, order prices, margins, spread fees, creation times, and statuses (where 1 indicates processing, 3 indicates a complete transaction, and 4 indicates an order that has been cancelled) in the report in the form of an information module.
Translation: 
-名称/代码：神农种业(300189.SZ)  
-类型：平仓  
-方向：做多  
-合约金额：50000  
-已成交数量：50000  
-下单方式：市价  
-下单价格：6.39  
-创建时间：2026-09-18 09:30:05  
-更新时间：2026-09-18 09:30:08  
-订单状态：1为处理中，3为全部成交、4为已撤单  
## CLOSE Order Completed，用户输入 15
List Close Order:
```http
GET /v1/aiclosetradeorder?limit=10&offset=0&status=3&sn={....}
```
Please present the codes, names, types, directions, contract amounts, completed transaction quantities, order methods, order prices, margins, spread fees, creation times, and statuses (where 1 indicates processing, 3 indicates a complete transaction, and 4 indicates an order that has been cancelled) in the report in the form of an information module.
Translation: 
-名称/代码：神农种业(300189.SZ)  
-类型：平仓  
-方向：做多  
-合约金额：50000  
-已成交数量：50000  
-下单方式：市价  
-下单价格：6.39  
-创建时间：2026-09-18 09:30:05  
-更新时间：2026-09-18 09:30:08  
-订单状态：1为处理中，3为全部成交、4为已撤单  
## CLOSE Order cancelled，用户输入 16
List Close Order:
```http
GET /v1/aiclosetradeorder?limit=10&offset=0&status=4&sn={....}
```
Please present the codes, names, types, directions, contract amounts, completed transaction quantities, order methods, order prices, margins, spread fees, creation times, and statuses (where 1 indicates processing, 3 indicates a complete transaction, and 4 indicates an order that has been cancelled) in the report in the form of an information module.
Translation: 
-名称/代码：神农种业(300189.SZ)  
-类型：平仓  
-方向：做多  
-合约金额：50000  
-已成交数量：50000  
-下单方式：市价  
-下单价格：6.39  
-创建时间：2026-09-18 09:30:05  
-更新时间：2026-09-18 09:30:08  
-订单状态：4为已撤单  
