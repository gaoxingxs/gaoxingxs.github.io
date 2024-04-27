## 配置

```python
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_USE_TLS = True
EMAIL_HOST = 'smtp.qq.com'
EMAIL_PORT = 587
EMAIL_HOST_USER = 'xxx@qq.com'
EMAIL_HOST_PASSWORD = 'xxx'
EMAIL_DEFAULT_FROM_EMAIL = 'xxx@qq.com'
```

## 发送简单邮件

```python
def send_email_captcha(request):
    email_account = request.GET.get('email')
    if not email_account:
        # 设置中文编码
        return JsonResponse({'code': 400, 'message': '请输入邮箱'}, json_dumps_params={'ensure_ascii': False})
    # 生成四位验证码
    captcha = ''.join(random.sample(string.digits, 4))
    # from_email不能为None
    send_mail('注册验证码', f'您的验证码是：{captcha}', settings.EMAIL_DEFAULT_FROM_EMAIL, [email_account])
    return JsonResponse({'code': 200, 'message': '发送成功'})
```