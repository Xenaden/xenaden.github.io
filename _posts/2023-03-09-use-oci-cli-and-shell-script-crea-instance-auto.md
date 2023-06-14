---
layout: post
title: 使用shell 脚本和crontab自动定时创建甲骨文云实例服务器
date: 2023-03-06 13:36
modified_date: 2023-06-14 15:14
categories: some-practice
tag: [Linux, Debian, oci-cli, shell script, crontab, Oracle Cloud instance]
---

下面直接来吧！展示过程和结果。  

### 执行`crontab -e`添加定时任务：
```
0,5,8,16,20,25,31,35,40,47,52,56 0-18 * * * cd /home/h**e/bin/oci-cli-scripts && ./instance_crea_auto.sh >>ica.log 2>&1 </dev/null &
```
在规定时间点进入指定目录执行shell 脚本，然后将脚本执行的结果输出到文件。

### 脚本如下：
```shell
/home/h**e/bin/oci compute instance launch --display-name a**e --availability-domain VNVa:PHX-AD-3--image-id ocid1.image.oc1. --subnet-id ocid1.subnet.oc1. --shape VM.Standard.A1.Flex --assign-public-ip true --metadata '{"ssh_authorized_keys": "your key"}' --compartment-id ocid1.tenancy.oc1. --shape-config '{"ocpus":4,"memory_in_gbs":24,"boot_volume_size_in_gbs":100}'

result="$(tail /home/h**e/bin/oci-cli-scripts/ica.log | grep -E 'Out of host capacity|The connection to endpoint timed out|Too many requests for the user')"
if  [ "${result}" =  "" ];then
        crontab -l | sed '1s/^/# /g' | crontab -
        $(echo Instance create successfully | mail -s Oci-cli yourmail@domain.com)
fi
```
首先是oci-cli 根据给定的条件创建实例的命令

接着`tail`命令获取oci-cli 执行命令输出的结果，在结果中找寻指定的字段，然后赋值给变量"result"。根据多次实验，未成功创建实例出错的状态有如下几种（如果你还有遇到其他的就自己加进去吧）：Out of host capacity、The connection to endpoint timed out、Too many requests for the user；  

然后就是判断。如果变量"result"为空，那应该就是成功了，停止crontab 定时任务。我这里用的是`sed`命令去注释了crontab 记录定时任务的文件；  

最后当然是开开心心地发送邮件到自己的邮箱啦。

<br><br>

---

<br><br>

### 下面是折磨过程
我爱白嫖，但我又是懒狗，这点东西搞了几天。

#### **安装oci-cli**
用oci-cli申请创建实例,这里略过，可以参考↗️ [^oci-cli_crea_instan]。
```
/home/h**e/bin/oci compute instance launch --display-name a**e --availability-domain VNVa:PHX-AD-3--image-id ocid1.image.oc1. --subnet-id ocid1.subnet.oc1. --shape VM.Standard.A1.Flex --assign-public-ip true --metadata '{"ssh_authorized_keys": "your key"}' --compartment-id ocid1.tenancy.oc1. --shape-config '{"ocpus":4,"memory_in_gbs":24,"boot_volume_size_in_gbs":100}'
```

<br>

#### **shell脚本**
##### 测试判断
执行了oci-cli创建实例的命令，然后对输出进行判断，若为空则执行
```
if  [ "${result}" =  "" ];then
        crontab -l | sed '1s/^/# /g' | crontab -
        $(echo Instance create successfully | mail -s Oci-cli yourmail@domain.com)
fi
```  
搞这脚本有个坑啊。crontab执行定时任务是用shell运行脚本，而用户登录进来默认使用的是bash，就导致shell对某些符号判断不同，在一直报错。我这里是`[]`的问题，在cron的文件/etc/crontab 有相关的配置。你说为何不直接修改shell为bash，我只能说搞了两天我已经搞懵，害怕任何的一个小修改又扯出什么幺蛾子，就让它们保持默认了当时。  
关于`[` [^open_bracket]，解决：  
> You should use the `=` operator for string comparison:
> ```
> Sourcesystem="ABC"
> 
> if [ "$Sourcesystem" = "XYZ" ]; then 
>     echo "Sourcesystem Matched" 
> else
>     echo "Sourcesystem is NOT Matched $Sourcesystem"  
> fi;
> ```

##### 重写入crontab [^rewrite_crontab]
判断是否有相关字符出现，停止crontab自动任务
```
crontab -l | sed '1s/^/# /g' | crontab -
```

##### 脚本执行结果提醒
上网翻来覆去，有现成的，如用Telegram Bot或者python脚本之类，但我都感觉差点意思，我想能不能脚本运行完成后直接发邮件到我的邮箱。

######  发邮件
安装mailuties, postfix，发送邮件直接进垃圾箱了，虽然是有发出来，但是没提醒，些许不爽，搜一下如何解决 [^Linux_mail]。  
根据方法， 伪装发件邮箱
> Open your main.cf config file using a text editor such as vim command/nano command:  
> `vi /etc/postfix/main.cf`
>
> Append following parameter  
> `smtp_generic_maps = hash:/etc/postfix/generic`
> 
> Save and close the file when using vim. Open /etc/postfix/generic file:  
> `vi /etc/postfix/generic`
> 
> Make sure tom-01@server01.hosting.com change to tom@domain.com as follows:  
> `tom-01@server01.hosting.com tom@domain.com`
> 
> Save and close the file. Create or update generic postfix table using the postmap command:  
> `postmap /etc/postfix/generic`
> 
> Finally restart or reload postfix service:  
> `/etc/init.d/postfix restart`  
> OR  
> `systemctl restart postfix.service`

###### 触发邮件 [^trigger_email]  

```
$(echo Instance create successfully | mail -s Oci-cli yourmail@domain.com)
```

<br>

##### **创建crontab任务**
解决了shell脚本的问题，开始执行定时任务测试。  
发现怎么没有输出，原来crontab 默认是发输出结果给当前用户了，更改一下重定向输出 [^redirect_crontab_output]
> crontab执行过程中，如果脚本输出没有重定向，那么会默认给系统用户发邮件，邮件内容一般存储在 
> /var/mail/$user 中，如果不清理就会打满服务器根分区，最终导致机器无法登陆。因此推荐的
> crontab命令写法如下：
> 
> ``` 
> 01 09 * * * cd /opdir/test/ && ./test.sh >>test.log 2>&1 </dev/null &
> ```


<br>

#### 题外话
由于我的机子，debian，默认系统时间是12小时制，但crontab 是按24小时制来执行任务的 [^24-hour_time_format]。
> It seems the reason for this change is that in buster the en_US locale is >updated. Since it is meant to represent US customs, which is to specify time in >the 12 hour format, time is 
> now shown as such, even though it may come as a >surprise to everyone who has relied on the old behavior.
> 
> A solution, as already mentioned by @JosefZ in the comments, is to use a locale >that has the time settings that you want, for example C.UTF-8. To make this 
> hange permanent on Debian you'd run:
> 
> ```
> update-locale LC_TIME=C.UTF-8
> ```
> 
> This sets the definition for LC_TIME in the file /etc/default/locale.
> 
> You can find more information about the change itself in [Debian bug #877900](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=877900)

然后重启了系统，时间还是12小时制。突然灵光一闪，是不是要打logout命令，因为刚刚也看了几个其他论坛的的回答说要log out，我在putty是exit了。输入logout，重新登录，date一下，变24小时制了。
再tail 一下日志，现在已经过了设定时间，不在进行定时任务了

---

[^oci-cli_crea_instan]: <https://note.lazzman.com:18084/project-2/doc-28/>,  
        <https://ioufev.com/2019/12/23/oracle-api-jiao-ben-chuang-jian-shi-li/>

[^Linux_mail]: <https://www.cyberciti.biz/tips/howto-postfix-masquerade-change-email-mail-address.html>,  
        <https://serverfault.com/questions/840834/how-to-change-recipient-on-postfix-relay-smtp-generic-maps-not-working>

[^trigger_email]: <https://yxli8023.github.io/2021/05/19/Linux-Mail.html#%E5%AE%9E%E4%BE%8B  >,
        <https://www.dbs724.com/130211.html>

[^open_bracket]: <https://stackoverflow.com/questions/10849297/compare-a-string-using-sh-shell>,  
        <https://stackoverflow.com/questions/3427872/whats-the-difference-between-and-in-bash>,  
        <https://stackoverflow.com/questions/5725296/difference-between-sh-and-bash>,  
        <https://www.runoob.com/linux/linux-shell-process-control.html>,  
        <https://www.runoob.com/w3cnote/shell-summary-brackets.html>,  
        <https://zhuanlan.zhihu.com/p/136591210>

[^rewrite_crontab]: <https://stackoverflow.com/questions/1251784/apply-sed-to-crontab-lines>

[^redirect_crontab_output]: <https://mengalong.github.io/2018/10/31/crontab-redirect/#%E5%95%B0%E5%97%A6%E4%B8%80%E5%8F%A5>

[^24-hour_time_format]: https://serverfault.com/questions/977598/date-in-terminal-now-in-12hr-format-on-debian-buster/979301#979301?newreg=7e192254ec6648bab07fa4df6161f2b1
