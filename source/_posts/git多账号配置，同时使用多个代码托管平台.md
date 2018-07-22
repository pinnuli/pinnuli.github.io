---
title: git多账号配置，同时使用多个代码托管平台
date: 2018-04-09 21:01:52
categories: "git" 
tags:	
        - git
---

我们在使用git管理代码的时候，经常需要放到不同的托管网站，如github，osc等，那么不同的网站账号不一样，就需要生成不同密钥，配置对应的不同网站，接下来我们写写如何处理。

ps:这里是在centos7.2下操作，不过其他操作系统依然适用，这里举的例子，一个是github，一个是osc。

### 1 生成密钥
这里可以设置密钥文件名和路径，/root/.ssh 是路径（一般路径选择默认），id_rsa_github是密钥文件名, 文件命名后按两次回车，即密码为空

``` bash
	ssh-keygen -T rsa -C "example@qq.com" 
```
生成github的密钥![git_multi_account_ssh_github](/images/git_multiaccount_ssh_github.jpg)

生成osc的密钥![git_multi_account_ssh_osc](/images/git_multiaccount_ssh_osc.jpg)



查看一下.ssh文件夹，发现有id_rsa_github, id_rsa_github.pub（放到github）,id_rsa_osc, id_rsa_osc.pub（放到osc)

``` bash
	ls -a /root/.ssh 
	
```

![git_multi_account_ssh_file](/images/git_multiaccount_ssh_file.png)

### 2 接下来配置多账号

在.ssh文件夹下面新建一个命名为config的文件，编辑如下内容

``` bash

	#github
        Host github.com    
        HostName github.com
        IdentityFile ~/.ssh/id_rsa_github
        User pinnuli

	#osc
        Host gitee.com
        HostName gitee.com
        IdentityFile ~/.ssh/id_rsa_osc
        User pinnuli

```

![git_multiaccount_config](/images/git_multiaccount_config.png)


### 3 把对应的公钥放到github和osc上面

![git_multiaccount_pub_github](/images/git_multiaccount_pub_github.png)

![git_multiaccount_pub_osc](/images/git_multiaccount_pub_osc.png)

### 4 测试是否成功


``` bash
	ssh -T git@github.com

```
![git_multiaccount_connect_github](/images/git_multiaccount_connect_github.png)

``` bash
	ssh -T git@gitee.com

	
```

![git_multiaccount_connect_osc](/images/git_multiaccount_connect_osc.png)


至此，git多账号配置完毕，需要更多账号也是一样的道理
