# 菜谱 3：发送 HTML 形式的邮件

我们平时需要使用 Python 发送各类邮件，这个需求怎么来实现？答案其实很简单，[smtplib](https://docs.python.org/2/library/smtplib.html) 和 [email](https://docs.python.org/2/library/email.html)库可以帮忙实现这个需求。[smtplib](https://docs.python.org/2/library/smtplib.html) 和 [email](https://docs.python.org/2/library/email.html) 的组合可以用来发送各类邮件：普通文本，HTML 形式，带附件，群发邮件，带图片的邮件等等。我们这里将会分几节把发送邮件功能解释完成。

[smtplib](https://docs.python.org/2/library/smtplib.html) 是 Python 用来发送邮件的模块，[email](https://docs.python.org/2/library/email.html) 是用来处理邮件消息。

发送 HTML 形式的邮件，需要 email.mime.text 中的 MIMEText 的 _subtype 设置为 html，并且 _text 的内容应该为 HTML 形式。其它的就和 [*菜谱 2：发送普通文本邮件*](http://www.pythondoc.com/python-cookbook/cookbook_2.html#cookbook-2) 一样:

```py
import smtplib
from email.mime.text import MIMEText

sender = '***'
receiver = '***'
subject = 'python email test'
smtpserver = 'smtp.163.com'
username = '***'
password = '***'

msg = MIMEText(u'''
你好
''','html','utf-8')

msg['Subject'] = subject

smtp = smtplib.SMTP()
smtp.connect(smtpserver)
smtp.login(username, password)
smtp.sendmail(sender, receiver, msg.as_string())
smtp.quit() 
```

注意：这里的代码并没有把异常处理加入，需要读者自己处理异常。