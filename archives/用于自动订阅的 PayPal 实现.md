本文将探讨 PayPal 自动续费的实现过程，包括支付结果处理和订阅管理的必要步骤。

## 自动续费的五个步骤

官方给出的自动续费流程如下：

1. 预先创建并激活订阅计划；
2. 用户创建订阅，跳转到 PayPal 网站等待用户确认；
3. 用户同意后，返回网站以执行订阅；
4. 获取用户账单，并接收每次扣款结果通知或主动查询支付结果；
5. 处理用户取消订阅等通知。

## 使用 PayPal SDK

您可以通过以下命令安装 PayPal SDK：

bash
composer require paypal/rest-api-sdk-php


有关完整的开发示例，可以访问 [PayPal SDK 示例页面](https://paypal.github.io/PayPal-PHP-SDK/sample/#billing)。

进行调试时，您可以使用 [PayPal Sandbox](https://www.sandbox.paypal.com/) 环境方便地测试。

## 创建和激活订阅计划

- 订阅计划（Billing Plan）相当于产品，每个商品应创建不同价格的计划；
- 在 Payment 中创建 `TRIAL` 类型支付时，必须同时存在 `REGULAR` 支付。`TRIAL` 并未特别识别新用户，首次优惠需由业务逻辑自行实现；
- 创建用户订阅协议时，**协议生效时间必须在当前时间24小时以后**，因此循环扣款无法立即执行，最早需等待24小时。通常情况下，首次扣款通过 `MerchantPreferences` 的 `setSetupFee` 设置；
- 若收到 `NotifyUrl` 值为 NULL 的错误，这是 PayPal 服务端的问题，详见相关 [GitHub 问题](https://github.com/paypal/PayPal-PHP-SDK/pull/1152/files)。

## 创建订阅

用户可以创建多个订阅协议（Billing Agreement）针对同一订阅计划，并跳转至 PayPal 网站以待用户同意。

因为协议开始时间 `start_date` 最早为当前时间后24小时，所以该值其实设定了第二次扣款时间。假设设置为每月付款，则 `start_date` 应设定为一个月后，并通过 `setSetupFee` 设置首次扣款费用。

创建订阅后尚未生成 `Agreement.id`，此时需从跳转链接中提取 `token`，以对应创建的订阅与用户同意后的协议信息。

php
$link = $agreement->getApprovalLink();
parse_str(parse_url($link, PHP_URL_QUERY), $params);
$token = $params['token'];


- 同一订阅计划可以被同一用户多次订阅，因此在执行新协议时，需要手动取消之前的协议；
- 实际扣款时间会有所延迟，每次循环扣款执行时间通常会比 `AgreementDetail.next` 显示的时间晚几个小时。因此，为确保交易连续性，可以设置提前一天扣款。

您也可以在 `My Apps -> REST API apps -> WEBHOOKS` 中设置 `webhook` 通知。每次循环扣款成功后，PayPal 会发送 `PAYMENT.SALE.COMPLETED` 的事件通知，其中的 `billing_agreement_id` 字段有助于匹配已创建的订阅。

每次 `AgreementDetail` 都会返回下次收款时间 `next` 参数。超过该时间后，可以通过 `Agreement::searchTransactions` 方法查询协议的所有交易。请注意，PayPal 实际的扣款时间一般会延迟，因此需要多次重试。

👉 [野卡 | 一分钟注册，轻松订阅海外线上服务](https://bit.ly/bewildcard)