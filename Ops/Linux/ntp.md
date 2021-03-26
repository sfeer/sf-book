# 对时服务

在多服务协同使用时，保持一致的时间是很重要的环节。

```
yum install -y ntp
systemctl enable ntpd
systemctl start ntpd

timedatectl
      Local time: Fri 2020-09-18 16:16:01 CST
  Universal time: Fri 2020-09-18 08:16:01 UTC
        RTC time: Fri 2020-09-18 08:16:01
       Time zone: Asia/Shanghai (CST, +0800)
     NTP enabled: yes
NTP synchronized: no
 RTC in local TZ: no
      DST active: n/a

```
