# Swift mailer Failed to authenticate on SMTP server with username
Recently we started using Amazon SES to send email out from a Symfony2 app. We struck an issue where mail wouldn’t send and in the dev log we had the following error
```
app.ERROR: Exception occurred while flushing email queue: Failed to authenticate on SMTP server with username &quot;USERNAME&quot; using 2 possible authenticators [] []
```
The issues is caused by the encryption and port missing from the swift mailer configuration. In your app/config/config.yml make sure your swift mailer configuration looks like the following. You need to add the encryption and port directives.
```
swiftmailer:
    transport: "%mailer_transport%"
    host:      "%mailer_host%"
    username:  "%mailer_user%"
    password:  "%mailer_password%"
    encryption: %mailer_encryption%
    port:       %mailer_port%
    spool:     { type: memory }
```