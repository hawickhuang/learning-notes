# AWS WAF使用

<!-- more -->

1. 工作原理

![](http://pe94yf4jn.bkt.clouddn.com/waf-01.png)

2. 创建web acl
    + 打开网址:https://console.aws.amazon.com/waf/;
    + 点击Create Web ACL
    + 输入对应名称，按需选择CloudFront或区域，若选择CF，选择对应CloudFront资源;若选择区域，则选择对应ALB资源;
    + 创建条件:(在conditions标签下有多个选择，依据条件需要创建所需条件)
        + 跨站脚本(使用跨站点脚本匹配条件) 
        + GEO匹配
        + IP匹配(使用 IP 匹配条件) 
        + 大小匹配(使用大小约束条件) 
        + SQL注入(使用 SQL 注入匹配条件) 
        + 字符串匹配(使用字符串匹配条件)
    + 创建规则:输入名称和关联CloudWatch，选择规则类型(常规规则和基于速率的规则)。若创建了基于速率的规则，则需 要配置速率限制;
    + 添加条件到规则;(使用规则)
    + 添加规则到Web ACL中和配置符合规则的操作(允许、阻止或计数);
    + 选择Web ACL的默认操作;(确定 Web ACL 的默认操作)
3. 针对常见攻击使用CloudFormation快速设置WAF保护
    + 创建CloudFormation堆栈，具体可查看在美国东部(弗吉尼亚北部)创建堆栈;([选择模板](https://s3.amazonaws.com/clo udformation-examples/community/common-attacks.jsonS3S3)
    + 将Web ACL与CloudFront关联;
    + 启用S3事件通知;
    + (可选)额外添加条件或规则;
4. 针对恶意请求的IP地址快速设置WAF保护(阻止提交恶意请求的 IP 地址)
    + 工作原理:
        + CloudFront 代表Web 应用程序接收请求时，它会将访问日志发送到 Amazon S3 存储桶，其中包含有关请求的详细信息; 
        + 对于存储在 Amazon S3 存储桶中的每个新的访问日志，都会触发 Lambda 函数。Lambda 函数分析日志文件并查找导致错误代码400、403、404 和 405 的请求。然后，该函数对恶意请求计数，并将结果临时存储在访问日志的 Amazon S3 存储桶中的 current_outstanding_requesters.json 中。
        + Lambda 函数将更新 AWS WAF 规则，在指定的时间段内阻止 current_outstanding_requesters.json 中列出的 IP 地址。 
        + Lambda 函数会在 CloudWatch 中发布执行指标，如分析的请求数和被阻止的 IP 地址数。
    + 创建步骤:
        + 创建CloudFormation;[模板文件](https://s3.amazonaws.com/awswaf.us-east-1/block-bad-behaving-ips/block-bad-behaving-ips_template.json)
        + 配置S3存储桶，每分钟单IP最大请求数，超出阀值的阻止时间;
        + 将Web ACL与CloudFront关联
        + 启用S3事件通知
        + 测试阀值和IP规则
5. 测试Web ACL(测试Web ACL)
6. CloudFront增强WAF
    + 自定义错误页面
    + 做地理限制
    + 选择响应的HTTP方法