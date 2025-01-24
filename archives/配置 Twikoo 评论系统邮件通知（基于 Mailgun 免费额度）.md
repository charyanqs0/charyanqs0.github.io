**前言：**

为了及时响应读者评论，给博客添加评论通知功能至关重要。本文将详细介绍如何利用 Mailgun 的免费邮件额度，为 Twikoo 评论系统配置完整的邮件通知功能。

**一、Mailgun 账号配置**

1. 注册 Mailgun 账号
- 访问 Mailgun 官网：[https://www.mailgun.com/](https://www.mailgun.com/)
- 每日免费额度：100封邮件
- 需要信用卡验证

> 如需虚拟信用卡，可使用：
> 👉 [野卡 | 一分钟注册，轻松订阅海外线上服务](https://bit.ly/bewildcard)

2. 域名绑定与验证
- 添加自定义域名
- 配置 DNS 记录
- 等待验证完成

**二、获取 SMTP 配置**

1. 访问路径：`Send` -> `Sending` -> `Domains`
2. 选择已验证域名
3. 查看 SMTP credentials
4. 重置 postmaster 密码

**三、验证 SMTP 配置**

python
import smtplib
from email.mime.text import MIMEText
from email.header import Header

# 邮件配置
msg = MIMEText('测试内容', 'plain', 'utf-8')
msg['Subject'] = Header('Twikoo 邮件测试', 'utf-8')
msg['From'] = 'postmaster@yourdomain.com'
msg['To'] = '收件人'

# SMTP 配置
smtp_server = 'smtp.mailgun.org'
smtp_port = 587
smtp_user = 'postmaster@yourdomain.com'
smtp_password = 'your-password'

# 发送测试
try:
    server = smtplib.SMTP(smtp_server, smtp_port)
    server.starttls()
    server.login(smtp_user, smtp_password)
    server.sendmail(smtp_user, ['recipient@email.com'], msg.as_string())
    print('发送成功')
except Exception as e:
    print('发送失败:', e)
finally:
    server.quit()


**四、配置 Twikoo 通知**

1. 基础配置
- 进入评论管理
- 选择配置管理
- 填写 SMTP 信息

2. 通知设置
- 配置管理员邮箱
- 设置发件人信息
- 配置邮件模板

**五、功能测试**

1. 管理员通知
- 发表测试评论
- 确认管理员收到通知

2. 用户通知
- 回复测试评论
- 验证用户收到通知

**六、安全设置**

1. 发信限制
- 进入 Mailgun 账户设置
- 配置 Custom Message Limit
- 建议设置：1000/月

2. 安全建议
- 定期更新密码
- 监控发信额度
- 及时处理异常

通过以上配置，您的 Twikoo 评论系统将具备完整的邮件通知功能，确保与读者的及时互动。记得定期检查配置是否正常，确保通知功能持续可用。

👉 [野卡 | 一分钟注册，轻松订阅海外线上服务](https://bit.ly/bewildcard)