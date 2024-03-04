# 菜谱 2：发送普通文本邮件

我们平时需要使用 Python 发送各类邮件，这个需求怎么来实现？答案其实很简单，[smtplib](https://docs.python.org/2/library/smtplib.html) 和 [email](https://docs.python.org/2/library/email.html)库可以帮忙实现这个需求。[smtplib](https://docs.python.org/2/library/smtplib.html) 和 [email](https://docs.python.org/2/library/email.html) 的组合可以用来发送各类邮件：普通文本，HTML 形式，带附件，群发邮件，带图片的邮件等等。我们这里将会分几节把发送邮件功能解释完成。

[smtplib](https://docs.python.org/2/library/smtplib.html) 是 Python 用来发送邮件的模块，[email](https://docs.python.org/2/library/email.html) 是用来处理邮件消息。

发送普通文本的邮件，只需要 email.mime.text 中的 MIMEText 的 _subtype 设置为 plain。首先导入 smtplib 和 mimetext。创建 smtplib.smtp 实例，connect 邮件 smtp 服务器，login 后发送:

```py
import smtplib
from email.mime.text import MIMEText
from email.header import Header

sender = '***'
receiver = '***'
subject = 'python email test'
smtpserver = 'smtp.163.com'
username = '***'
password = '***'

msg = MIMEText(u'你好','plain','utf-8')#中文需参数‘utf-8'，单字节字符不需要
msg['Subject'] = Header(subject, 'utf-8')

smtp = smtplib.SMTP()
smtp.connect(smtpserver)
smtp.login(username, password)
smtp.sendmail(sender, receiver, msg.as_string())
smtp.quit()
```

注意：这里的代码并没有把异常处理加入，需要读者自己处理异常。