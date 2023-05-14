---
layout: post
title: 执行了pvremove 后如何重新加载PV
date: 2023-05-03 19:18
# modified_date: 2023-05-14 22:47
categories: some-practice
tag: [LVM, PV, VG, Linux]
---

关于LVM必须要先了解的一些概念知识  
https://grimoire.carcano.ch/blog/lvm-tutorial/#Creating_LV


<br><br>

想恢复vg元数据（metadata）的时候发现找不到uuid和PV  
[All LVM volume group logical volumes are missing with inconsistent metadata error]
(https://www.suse.com/support/kb/doc/?id=000020552)

如何解决找不的uuid和PV呢  
<https://www.learnitguide.net/2017/12/couldnt-find-device-with-uuid-recover.html>

Activate volume group 激活VG  
<https://tldp.org/HOWTO/LVM-HOWTO/activatevgs.html>
