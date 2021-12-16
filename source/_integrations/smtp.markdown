---
title: SMTP
description: Instructions on how to add e-mail notifications to Home Assistant.
ha_category:
  - Notifications
ha_iot_class: Cloud Push
ha_release: pre 0.7
ha_codeowners:
  - '@fabaff'
ha_domain: smtp
ha_platforms:
  - notify
---

The SMTP platform allows you to deliver notifications from Home Assistant to an e-mail recipient.

To enable notification by e-mail in your installation, add the following to your `configuration.yaml` file:

```yaml
# Example configuration.yaml entry
notify:
  - name: "NOTIFIER_NAME"
    platform: smtp
    sender: "YOUR_SENDER"
    recipient: "YOUR_RECIPIENT"
```

Check your e-mail provider configuration or help pages to get the correct SMTP settings.

{% configuration %}
name:
  description: Setting the optional parameter `name` allows multiple notifiers to be created. The notifier will bind to the service `notify.NOTIFIER_NAME`.
  required: false
  type: string
  default: notify
sender:
  description: E-mail address of the sender.
  required: true
  type: string
recipient:
  description: Default E-mail address of the recipient of the notification. This can be a recipient address or a list of addresses for multiple recipients.<br>This is where you want to send your E-mail notifications by default (when not specifying `target` in the service call). Any E-mail address(es) specified in the service call's `target` field will override this recipient content.
  required: true
  type: [list, string]
server:
  description: SMTP server which is used to send the notifications.
  required: false
  type: string
  default: localhost  
port:
  description: The port that the SMTP server is using.  
  required: false
  type: integer
  default: 587
timeout:
  description: The timeout in seconds that the SMTP server is using.
  required: false
  type: integer
  default: 5
username:
  description: Username for the SMTP account.
  required: false
  type: string
password:
  description: Password for the SMTP server that belongs to the given username. Make sure to wrap it in double quotes; e.g., `"MY_PASSWORD"`.
  required: false
  type: string
encryption:
  description: Set mode for encryption, `starttls`, `tls` or `none`.
  required: false
  type: string
  default: starttls
sender_name:
  description: "Sets a custom 'sender name' in the emails headers (*From*: Custom name <example@mail.com>)."
  required: false
  type: string
debug:  
  description: Enables Debug, e.g., `true` or `false`.
  required: false
  type: boolean
  default: false
{% endconfiguration %}

### Usage

To use the SMTP notification, refer to it in an automation or script like in this example:

```yaml
burglar:
  alias: "Burglar Alarm"
  sequence:
    - service: shell_command.snapshot
    - delay:
          seconds: 1
    - service: notify.NOTIFIER_NAME
      data:
          title: "Intruder alert"
          message: "Intruder alert at apartment!!"
          target:
            - "my_intruder_alert@example.com"
          data:
              images:
                  - /home/pi/snapshot1.jpg
                  - /home/pi/snapshot2.jpg
```

The optional `target` field is used to specify recipient(s) for this specific service call. When `target` field is not used, this message will be sent to default recipient(s), in this example, james@example.com and bob@example.com.

The optional `images` field adds in-line image attachments to the email. This sends a text/HTML multi-part message instead of the plain text default.

The optional `html` field makes a custom text/HTML multi-part message, allowing total freedom for sending rich html emails. In them, if you need to attach images, you can pass both arguments (`html` and `images`), the attachments will be joined with the basename of the images, so they can be included in the html page with `src="cid:image_name.ext"`.

```yaml
burglar:
  alias: "Burglar Alarm"
  sequence:
    - service: shell_command.snapshot
    - delay:
          seconds: 1
    - service: notify.NOTIFIER_NAME
      data:
          message: "Intruder alert at apartment!!"
          data:
            images:
              - /home/pi/snapshot1.jpg
              - /home/pi/snapshot2.jpg
            html: >
              <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
              <html lang="en" xmlns="http://www.w3.org/1999/xhtml">
                  <head>
                      <meta charset="UTF-8">
                      <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
                      <meta name="viewport" content="width=device-width, initial-scale=1.0">
                      <title>Intruder alert</title>
                      <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-alpha.5/css/bootstrap.min.css">
                      <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/font-awesome/4.4.0/css/font-awesome.min.css">
                      <style type="text/css">
                          @font-face {
                            font-family: 'Open Sans';
                            font-style: normal;
                            font-weight: 300;
                            src: local('Open Sans Light'), local('OpenSans-Light'), url(http://fonts.gstatic.com/s/opensans/v13/DXI1ORHCpsQm3Vp6mXoaTZS3E-kSBmtLoNJPDtbj2Pk.ttf) format('truetype');
                          }
                          h1,h2,h3,h4,h5,h6 {
                              font-family:'Open Sans',Arial,sans-serif;
                              font-weight:400;
                              margin:10px 0
                          }
                      </style>
                  </head>
                  <body>
                    <div class="jumbotron jumbotron-fluid" style="background-color: #f00a2d; color: white;">
                        <div class="container py-0">
                            <h1>Intruder alert at apartment!!</h1>
                        </div>
                    </div>
                    <div class="container-fluid">
                      <div class="row">
                        <div class="col-xs-12 col-md-6 px-0">
                          <img class="rounded" style="width: 100%;"
                              alt="snapshot1" src="cid:snapshot1.jpg" />
                        </div>
                        <div class="col-xs-12 col-md-6 px-0">
                          <img class="rounded" style="width: 100%;"
                              alt="snapshot2" src="cid:snapshot2.jpg" />
                        </div>
                      </div>
                      <br>
                    </div>
                  </body>
                  <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.11.3/jquery.min.js"></script>
                  <script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-alpha.5/js/bootstrap.min.js"></script>
              </html>
```

To learn more about how to use notifications in your automations, please see the [getting started with automation page](/getting-started/automation/).

## Specific E-Mail Provider Configuration

Check below some configurations examples for specific e-mail providers. 
If you are in doubt about the SMTP settings required, check your e-mail provider configuration or help pages for more information about its specific SMTP configuration. 

### Google Mail

A sample configuration entry for Google Mail.

```yaml
# Example configuration.yaml entry for Google Mail.
notify:
  - name: "NOTIFIER_NAME"
    platform: smtp
    server: "smtp.gmail.com"
    port: 587
    timeout: 15
    sender: "YOUR_USERNAME@gmail.com"
    encryption: starttls
    username: "YOUR_USERNAME@gmail.com"
    password: "YOUR_PASSWORD"
    recipient:
      - "RECIPIENT_1@example.com"
      - "RECIPIENT_N@example.com"
    sender_name: "SENDER_NAME"
```

Keep in mind that Google has some extra layers of protection that need special attention. By default, the usage by external applications is limited so you will need to visit the [less secure apps](https://www.google.com/settings/security/lesssecureapps) page and enable it to be able to send e-mails. Be aware that Google will periodically turn it off if it is not used (no e-mail is sent).

To avoid having your e-mail notifications broken due to the less secure app's behavior, it is recommended that you enable 2-step verification on your Google account, and use [an application-specific password](https://support.google.com/mail/answer/185833?hl=en) in your notification configuration.
