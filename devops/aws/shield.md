# AWS Shield使用

<!-- more -->

1. AWS Shield是一种托管式分布式拒绝服务(DDoS)防护服务，用于保护AWS上运行的程序;
2. AWS Shield提供持续检测和自动内联缓解功能，尽可能缩短应用程序的停机时间和延迟;
3. AWS Shield分为Standard和Advanced两种服务，下表是两者的区别:

    | 功能                                                   | AWS shield standard                                                                    | AWS shield advanced                                                                    |
    | ------------------------------------------------------ | -------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
    | 网络流量监控                                           | 是                                                                                     | 是                                                                                     |
    | 自动始终开启检测                                       | 是                                                                                     | 是                                                                                     |
    | 自动化应用程序 (第 7 层) 流量监控                      | 否                                                                                     | 是                                                                                     |
    | 有助于防范常见的 DDoS 攻击，如 SYN 泛洪和 UDP 反射攻击 | 是                                                                                     | 是                                                                                     |
    | 获得其他 DDoS 缓解能力                                 | 否                                                                                     | 是                                                                                     |
    | 自定义应用程序层 (第 7 层) 缓解                        | 是，通过用户创建的 AWS WAF ACL。产生标准 AWS WAF 费用。                                | 是，通过用户创建或 DRT 创建的 AWS WAF ACL。作为 AWS Shield Advanced 订阅的一部分提供。 |
    | 即时规则更新                                           | 是，通过用户创建或 DRT 创建的 AWS WAF ACL。作为 AWS Shield Advanced 订阅的一部分提供。 | 是                                                                                     |
    | 用于应用程序漏洞保护的 AWS WAF                         | 是，通过用户创建的 AWS WAF ACL。产生标准 AWS WAF 费用。                                | 是                                                                                     |
    | 第 3/4 层攻击通知                                      | 否                                                                                     | 是                                                                                     |
    | 第 3/4 层攻击取证报告 (源 IP、攻击媒介等)              | 否                                                                                     | 是                                                                                     |
    | 第 7 层攻击通知                                        | 是，通过 AWS WAF。产生标准 AWS WAF 费用。                                              | 是                                                                                     |
    | 第 7 层攻击取证报告 (Top talker 报告、采样请求等)      | 是，通过 AWS WAF。产生标准 AWS WAF 费用。                                              | 是                                                                                     |
    | 第 3/4/7 层攻击历史报告                                | 否                                                                                     | 是                                                                                     |
    | 高严重性事件期间的事故管理                             | 否                                                                                     | 是                                                                                     |
    | 攻击期间的自定义缓解                                   | 否                                                                                     | 是                                                                                     |
    | 攻击后分析                                             | 否                                                                                     | 是                                                                                     |

4. Shield Advanced创建(启用和配置 AWS Shield Advanced)
    + 4-1. 打开[waf控制台](https://console.aws.amazon.com/waf/)，选择Protected resources; 2. 选择Activate AWS Shield Advanced;
    + 4-2. 选择要保护的资源类型和资源。
    + 4-3. 输入名称;
    + 4-4. 对于Web DDoS attack，选择Enable。关联Web ACL; 6. 选择Add DDoS protection。

5. 授权DDoS响应团队帮我们创建规则和Web ACL(授权 DDoS 响应团队代表您创建规则和 Web ACL)
    + 5-1. 使用[shield配置模版](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#cstack=sn%7eRateBasedBL%7cturl%7ehttps:%2f% 2fs3.amazonaws.com/aws-shield-drt-support/drt_support_iam_role.json)创建堆栈;
    + 5-2. 按需填写;