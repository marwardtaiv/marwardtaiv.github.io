## 背景

由于业务需求需要集成 PayPal 实现循环扣款功能，我在查阅了搜索引擎的相关资料后，发现没有找到合适的开发教程，只有官方文档提供了一些信息。因此，经过两天的努力，我成功地实现了集成。在此分享一下如何使用 PayPal 的支付接口。

## PayPal 接口概览

PayPal 目前有多种接口可供选择：

1. 通过 **Braintree** 实现 Express Checkout（后续将重点介绍 Braintree）；
2. 创建 App，通过 REST API 接口方式（当前主流方式）；
3. NVP/SOAP API（旧接口）。

### Braintree 接口

**Braintree** 是 PayPal 收购的一家公司，除了支持 PayPal 支付外，还提供了升级计划、信用卡和客户信息等一整套管理功能，使用体验更加友好。尽管 PayPal 的 REST 接口也提供大部分功能，但由于 PayPal 的仪表盘无法直接管理这些信息，而 Braintree 则可以，因而我更倾向于使用 Braintree。不过，当我完成了相关功能后却发现 **Braintree** 在国内不被支持，这让我感到无奈。

### REST API

REST API 是顺应时代发展的产物。如果你之前使用过 OAuth 2.0 和 REST API，那么在使用这些接口时应该不会感到困惑。

### 旧接口

除非 REST API 无法满足需求（如政策限制等），否则不推荐使用旧接口。全球范围内都在向 OAuth 2.0 认证方式和 REST API 使用方式迁移，选择旧接口并不明智。

## REST API 的介绍

官方的 API 参考文档 [PayPal API 文档](https://developer.paypal.com/webapps/developer/docs/api/) 提供了详细的接口介绍。然而，直接调用这些 API 可能会比较繁琐，我们期待尽快完成业务需求，而不是深入了解 API 的细节。

建议直接安装官方提供的 [PayPal-PHP-SDK](https://github.com/paypal/PayPal-PHP-SDK)，并利用其 Wiki 作为起点。

在完成第一个示例之前，请确保你有 **Sandbox** 账户并正确配置以下内容：

1. **Client ID**
2. **Client Secret**
3. **Webhook API**（必须是 HTTPS 开头且使用 443 端口，建议使用 ngrok 进行反向代理）
4. **Return URL**（同样要求为 HTTPS）

完成 Wiki 的第一个示例后，了解接口的分类将有助于满足你的业务需求，具体接口分类如下：

- **Payments**: 一次性支付接口，不支持循环捐款。
- **Payouts**: 忽略。
- **Authorization and Capture**: 支持用户通过 PayPal 账户登录你的网站并获取相应信息。
- **Sale**: 与商城相关，忽略。
- **Order**: 与商城相关，忽略。
- **Billing Plan & Agreements**: 这是实现循环扣款的核心功能，必须使用的接口。
- **Vault**: 存储信用卡信息。
- **Payment Experience**: 忽略。
- **Notifications**: 处理 Webhook 信息，重要但不在本文讨论范围内。
- **Invoice**: 票据处理。
- **Identity**: 认证处理，已在 PayPal-PHP-SDK 中涵盖，本文不讨论。

## 如何实现循环扣款

实现循环扣款的步骤如下：

1. 创建并激活升级计划；
2. 创建订阅（创建 Agreement），用户将被引导至 PayPal 网站进行同意；
3. 用户同意后，执行订阅；
4. 获取扣款账单。

### 1. 创建升级计划

升级计划对应 **Plan** 类，注意事项包括：

- 升级计划创建后默认为 **CREATED** 状态，需将状态修改为 **ACTIVE** 才能正常使用；
- **Plan** 对象必须包含 **PaymentDefinition** 和 **MerchantPreferences** 两个属性；
- 若要创建 **TRIAL** 类型的计划，必须有与之对应的 **REGULAR** 支付定义；
- `setSetupFee` 方法用于设置首次扣款费用，而 **Agreement** 对象的费用设置用于第二次扣款。

以下是创建一个 **Standard** 计划的示例参数：

php
$param = [
    "name" => "standard_monthly",
    "display_name" => "Standard Plan",
    "desc" => "standard Plan for one month",
    "type" => "REGULAR",
    "frequency" => "MONTH",
    "frequency_interval" => 1,
    "cycles" => 0,
    "amount" => 20,
    "currency" => "USD"
];


创建并激活计划的代码示例：

php
public function createPlan($param)
{
    $apiContext = $this->getApiContext();
    $plan = new Plan();

    // 填充基本信息
    $plan->setName($param->name)
        ->setDescription($param->desc)
        ->setType('INFINITE');

    // 定义支付内容
    $paymentDefinition = new PaymentDefinition();
    $paymentDefinition->setName($param->name)
        ->setType($param->type)
        ->setFrequency($param->frequency)
        ->setFrequencyInterval((string)$param->frequency_interval)
        ->setCycles((string)$param->cycles)
        ->setAmount(new Currency(array('value' => $param->amount, 'currency' => $param->currency)));

    // 设置商户偏好
    $merchantPreferences = new MerchantPreferences();
    $merchantPreferences->setReturnUrl("$returnUrl?success=true")
        ->setCancelUrl("$returnUrl?success=false")
        ->setAutoBillAmount("yes")
        ->setInitialFailAmountAction("CONTINUE")
        ->setMaxFailAttempts("0")
        ->setSetupFee(new Currency(array('value' => $param->amount, 'currency' => 'USD')));

    $plan->setPaymentDefinitions(array($paymentDefinition));
    $plan->setMerchantPreferences($merchantPreferences);

    // 尝试创建计划
    try {
        $output = $plan->create($apiContext);
    } catch (Exception $ex) {
        return false;
    }

    // 激活计划
    $patchRequest = new PatchRequest();
    $patch = new Patch();
    $patch->setOp('replace')
          ->setPath('/')
          ->setValue(new PayPalModel('{"state":"ACTIVE"}'));
          
    $patchRequest->addPatch($patch);
    $output->update($patchRequest, $apiContext);

    return $output;
}


### 2. 创建订阅（Agreement）

创建 **Agreement** 对象，注意事项包括：

- **setSetupFee** 方法设置首次扣款费用，而 **Agreement** 对象设置的是第二次费用；
- **setStartDate** 方法需正确设置为第二次扣款的时间，格式使用 `ISO8601`；
- 为了唯一标识用户订阅，可通过 **Agreement** 的 **getApprovalLink** 方法获取 URL，提取 **token** 作为标识。

以下是创建订阅的参数示例：

php
$param = [
    'id' => 'P-26T36113JT475352643KGIHY', // 上一步中创建的 Plan ID
    'name' => 'Standard', 
    'desc' => 'Standard Plan for one month'
];


代码示例：

php
public function createPayment($param)
{
    $apiContext = $this->getApiContext();
    $agreement = new Agreement();

    $agreement->setName($param['name'])
        ->setDescription($param['desc'])
        ->setStartDate(Carbon::now()->addMonths(1)->toIso8601String());

    // 添加 Plan ID
    $plan = new Plan();
    $plan->setId($param['id']);
    $agreement->setPlan($plan);

    // 添加付款方
    $payer = new Payer();
    $payer->setPaymentMethod('paypal');
    $agreement->setPayer($payer);

    // 尝试创建 Agreement
    try {
        $agreement = $agreement->create($apiContext);
        $approvalUrl = $agreement->getApprovalLink();
    } catch (Exception $ex) {
        return "创建支付失败，请重试或联系商家。";
    }
    return $approvalUrl; // 返回跳转 URL
}


返回的 `$approvalUrl` 需要通过 `redirect($approvalUrl)` 跳转至 PayPal 网页等待用户支付。

### 用户同意后执行订阅

用户同意后，需执行 **Agreement** 的 **execute** 方法完成真正的订阅，注意事项包括：

- 完成订阅并不代表立即扣款，可能会延迟；
- 如果 `setSetupFee` 设置为 0，则需等待循环扣款时间到达才能产生订单。

代码片段示例：

php
public function onPay($request)
{
    $apiContext = $this->getApiContext();
    if ($request->has('success') && $request->success == 'true') {
        $token = $request->token;
        $agreement = new \PayPal\Api\Agreement();
        try {
             $agreement->execute($token, $apiContext);
        } catch (\Exception $e) {
            return null;
        }
        return $agreement;
    }
    return null;
}


### 获取交易记录

在订阅后，交易记录可能不会立即生成，若为空则可稍后重试。注意事项包括：

- `start_date` 和 `end_date` 不能为空。
- 该函数返回的对象可能不总是返回有效的 JSON 对象，需根据 **AgreementTransactions** 的 API 说明手动输出相应参数。

代码示例：

php
public function transactions($id)
{
    $apiContext = $this->getApiContext();
    $params = ['start_date' => date('Y-m-d', strtotime('-15 years')), 'end_date' => date('Y-m-d', strtotime('+5 days'))];
    try {
        $result = Agreement::searchTransactions($id, $params, $apiContext);
    } catch (\Exception $e) {
        Log::error("获取交易记录失败：" . $e->getMessage());
        return null;
    }
    return $result->getAgreementTransactionList();
}


PayPal 官方也提供了相应的教程，尽管其调用原生接口与本文的流程有所不同，供有兴趣的开发者参考：[PayPal 教程](https://developer.paypal.com/docs/integration/direct/billing-plans-and-agreements/)。

## 需要注意的问题

功能虽然实现，但仍需考虑以下几点：

1. 国内使用 **Sandbox** 测试时连接速度较慢，经常超时，因此需考虑用户在此过程中可能关闭页面的情况；
2. 必须实现 **Webhook** ，否则用户在 PayPal 取消订阅时无法及时获得通知；
3. 一旦创建了 **Agreement**，除非主动取消，否则将一直生效。因此，若网站设计了多种升级计划，用户在切换计划时必须取消前一个计划；
4. 用户同意订阅的过程应视为一个原子操作，应该放入队列中执行，以提高用户体验。

👉 [野卡 | 一分钟注册，轻松订阅海外线上服务](https://bit.ly/bewildcard)