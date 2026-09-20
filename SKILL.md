---
name: Goldstonestock
description: The API of Goldstonestock skill bot can be utilized to achieve login based on Goldstonestock, setting of API keys, account inquiry, fund deposit, strategy management, strategy investment and closing, as well as viewing of holdings and profit/loss status. This intelligent agent is merely used as a simulation quantitative software and does not generate real investments. When users wish to have the AI agent operate Goldstonestock, these API functions can be triggered depositing simulated funds through deposit contracts, allocating MAIN funds to various strategies, viewing balance or holdings status, and closing a certain strategy.
---
## 前置强制约束（必须放在最上方，解决一直思考）
1. 每一轮**仅处理用户当前单条消息**，输出完回复，本轮立即终止，禁止自动执行下一轮、禁止主动预判用户输入。
2. menu_open 状态**只在收到用户消息时读取/修改**；模型输出回复阶段，不读取、不修改状态。
3. 命中规则输出文本后，本轮任务结束，不再额外推理。
# Goldstonestock
By leveraging this skill, the intelligent agent can interact on behalf of the user with the Goldstonestock API. Goldstonestock uses email verification for login and employs the user's API key. The intelligent agent merely needs to call the Goldstonestock API. The user's identity identifier is their Goldstonestock trading account, and this account information is passed by the Goldstonestock backend in the form of "trade_name"; do not inquire about the user's private key or the Goldstonestock password.
## Configuration

Use the service base URL provided by the user or the environment:

```text
CLAWSTOCK_API_BASE_URL=https://jtzj.duoso.vip
```

If no base URL is known, ask the user for it before calling the API.

Authenticated requests use the user API key returned by `/v1/aicheckauth`:

```text
Authorization: Bearer <api_key>
```

Please store this Goldstonestock user API key only in the confidential/session storage space of the proxy program. Do not disclose it unless the user explicitly requests it.


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
【4】智能选股策略列表  
【5】智能交易策略列表  
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
Please do not list the complete list of commands in every reply. In the general responses to users, simply state: Users can input `/help` to view all available operations.

When a user sends `/help` or asks about available commands, please reply in Chinese with the following list of commands.

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
### 触发 /reset
🔄 菜单状态已重置
菜单已关闭，输入 /menu 重新唤起
### 触发 /close
❕ 菜单已关闭
输入 /menu 随时重新打开菜单
## Safety Rules
- Don't make any claims about guaranteed profits.
-Except for fulfilling the explicit requests of the Goldstonestock users, no personalized financial advice shall be provided.
- Do not request or handle mnemonic phrases, private keys, wallet passwords or original wallet recovery data.
- Please treat the API key as confidential information. If an API key is leaked during the chat, please inform the user and make sure to change it as soon as possible if it can be done.
- Before performing the `POST /v1/aideposit` operation, please confirm with the user the deposit amount and the assets involved.
- Do not expose the term "JWT token" to the users.
-All the returned results will not display any information related to the field names to the users.
-All API interfaces should be based on the returned fields. Do not add historical fields without authorization.
## Login Flow ，用户输入 1
1. Ask the user for their email address.
2. Create a challenge:

```http
POST /v1/ailogin
Content-Type: application/json
{"username":"...@qq.com"}
```

3. Show the returned `Verification code sent successfully`. 
4. Use the received verification code to validate the result endpoint, continuing this process until the `code` reaches 1
```http
POST /v1/aiverylogin
Content-Type: application/json
{"username":"...@qq.com","code":"...."}
```

5. Save `verify.api_key` from the result response. Do not ask the user to copy the API key from the page.

There is also a direct agent path: if the user provides a signature, call:

```http
POST /v1/aiverify
Content-Type: application/json
{"sn":"1234"}
```

The verify response includes `id`, `email`, `trade_name`, `isauthposition`, and account data when available.
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
