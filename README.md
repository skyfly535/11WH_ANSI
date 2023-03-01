# Автоматизация администрирования. Ansible.

## Развертывание стенда для демонстрации работы Ansible.

Стэнд состоит из хостовой машины под управлением ОС Ubuntu 20.04 на которой развернут Ansible и двух виртуальных машин CentOS/7, Ubuntu 22.04 (для проигрывания на них сценариев).

### Установка Ansible

Скачиваем скрипт закачки пакетного менеджера `pip`

```
curl -LO https://bootstrap.pypa.io/get-pip.py
```

Вызываем `python3` скрипт и устанавливаем для текущего юзера

```
python3 get-pip.py --user
```

Добавить путь из предупркеждения при установке в переменные окружения `~/.bashrc` или `~/.zshrc`

Производим установку Ansible

```
python3 -m pip install --user ansible
```
### Развертывание виртуальных машин

Используем приложенный `Vagrantfile`

Делаем 

```
vagrant up
```

Проверяем состояние виртуальных машин

```
oman@root-ubuntu:~/ansible$ sudo vagrant status
Current machine states:

NginxCnt                  running (virtualbox)
NginxUbt                  running (virtualbox)

roman@root-ubuntu:~/ansible$ sudo vagrant ssh-config NginxCnt
Host NginxCnt
  HostName 127.0.0.1
  User vagrant
  Port 2222
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /home/roman/ansible/.vagrant/machines/NginxCnt/virtualbox/private_key
  IdentitiesOnly yes
  LogLevel FATAL

roman@root-ubuntu:~/ansible$ sudo vagrant ssh-config NginxUbt
Host NginxUbt
  HostName 127.0.0.1
  User vagrant
  Port 2203
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /home/roman/ansible/.vagrant/machines/NginxUbt/virtualbox/private_key
  IdentitiesOnly yes
  LogLevel FATAL

```
### SSH ключи

Для работы Ansible необходимо на хостовой машине сгенерировать SSH ключи

```
ssh-keygen
```
Публичную часть ключа из каталога `.ssh/id_rsa.pub` хостовой машины поместить в каталог `.ssh/authorized_keys`

Теперь все готово для работы

## Формирование и выполнение playbook.

С помощью модулей yum/apt производим обновление ОС CentOS и Ubuntu соответственно и установку на них NGINX: `task Update CentOS` , `task Update Ubuntu`, `task instal epel-release`, `tasks instal nginx`

С помощью шаблона jinja2 формируем нужный файл настроек NGINX (ставим порт прослушивания 8080): `task template`, файл `nginx.conf.j2`.

С помощью `notify` производим запуск NGINX `handler start nginx` и делаем reloload при изменении настроек `handler reloload nginx`.

### Ход выполнения playbook
```
roman@root-ubuntu:~/ansible$ ansible-playbook -i hosts  nginx.yml 

PLAY [test playbook] *****************************************************************************************

TASK [Gathering Facts] *****************************************************************************************
ok: [192.168.56.27]
ok: [192.168.56.28]

TASK [print os_family] *****************************************************************************************
ok: [192.168.56.27] => {
    "ansible_facts['os_family']": "RedHat",
    "changed": false
}
ok: [192.168.56.28] => {
    "ansible_facts['os_family']": "Debian",
    "changed": false
}

TASK [Update CentOS] *****************************************************************************************
skipping: [192.168.56.28]
ok: [192.168.56.27]

TASK [instal epel-release] *****************************************************************************************
skipping: [192.168.56.28]
changed: [192.168.56.27]

TASK [instal nginx] *****************************************************************************************
skipping: [192.168.56.28]
changed: [192.168.56.27]

TASK [Update Ubuntu] *****************************************************************************************
skipping: [192.168.56.27]
ok: [192.168.56.28]

TASK [instal nginx] *****************************************************************************************
skipping: [192.168.56.27]
changed: [192.168.56.28]

TASK [template] *****************************************************************************************
changed: [192.168.56.27]
changed: [192.168.56.28]

RUNNING HANDLER [start nginx] *****************************************************************************************
ok: [192.168.56.28]
changed: [192.168.56.27]

RUNNING HANDLER [reloload nginx] *****************************************************************************************
changed: [192.168.56.27]
changed: [192.168.56.28]

PLAY RECAP *****************************************************************************************
192.168.56.27              : ok=8    changed=5    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
192.168.56.28              : ok=7    changed=3    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0   
```
## Результат выполнения

### NGINX на CentOS
```
roman@root-ubuntu:~$ curl 192.168.56.27:8080
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
<head>
  <title>Welcome to CentOS</title>
  <style rel="stylesheet" type="text/css"> 

	html {
	background-image:url(img/html-background.png);
	background-color: white;
	font-family: "DejaVu Sans", "Liberation Sans", sans-serif;
	font-size: 0.85em;
	line-height: 1.25em;
	margin: 0 4% 0 4%;
	}

	body {
	border: 10px solid #fff;
	margin:0;
	padding:0;
	background: #fff;
	}

	/* Links */

	a:link { border-bottom: 1px dotted #ccc; text-decoration: none; color: #204d92; }
	a:hover { border-bottom:1px dotted #ccc; text-decoration: underline; color: green; }
	a:active {  border-bottom:1px dotted #ccc; text-decoration: underline; color: #204d92; }
	a:visited { border-bottom:1px dotted #ccc; text-decoration: none; color: #204d92; }
	a:visited:hover { border-bottom:1px dotted #ccc; text-decoration: underline; color: green; }
 
	.logo a:link,
	.logo a:hover,
	.logo a:visited { border-bottom: none; }

	.mainlinks a:link { border-bottom: 1px dotted #ddd; text-decoration: none; color: #eee; }
	.mainlinks a:hover { border-bottom:1px dotted #ddd; text-decoration: underline; color: white; }
	.mainlinks a:active { border-bottom:1px dotted #ddd; text-decoration: underline; color: white; }
	.mainlinks a:visited { border-bottom:1px dotted #ddd; text-decoration: none; color: white; }
	.mainlinks a:visited:hover { border-bottom:1px dotted #ddd; text-decoration: underline; color: white; }

	/* User interface styles */

	#header {
	margin:0;
	padding: 0.5em;
	background: #204D8C url(img/header-background.png);
	text-align: left;
	}

	.logo {
	padding: 0;
	/* For text only logo */
	font-size: 1.4em;
	line-height: 1em;
	font-weight: bold;
	}

	.logo img {
	vertical-align: middle;
	padding-right: 1em;
	}

	.logo a {
	color: #fff;
	text-decoration: none;
	}

	p {
	line-height:1.5em;
	}

	h1 { 
		margin-bottom: 0;
		line-height: 1.9em; }
	h2 { 
		margin-top: 0;
		line-height: 1.7em; }

	#content {
	clear:both;
	padding-left: 30px;
	padding-right: 30px;
	padding-bottom: 30px;
	border-bottom: 5px solid #eee;
	}

    .mainlinks {
        float: right;
        margin-top: 0.5em;
        text-align: right;
    }

    ul.mainlinks > li {
    border-right: 1px dotted #ddd;
    padding-right: 10px;
    padding-left: 10px;
    display: inline;
    list-style: none;
    }

    ul.mainlinks > li.last,
    ul.mainlinks > li.first {
    border-right: none;
    }

  </style>

</head>

<body>

<div id="header">

    <ul class="mainlinks">
        <li> <a href="http://www.centos.org/">Home</a> </li>
        <li> <a href="http://wiki.centos.org/">Wiki</a> </li>
        <li> <a href="http://wiki.centos.org/GettingHelp/ListInfo">Mailing Lists</a></li>
        <li> <a href="http://www.centos.org/download/mirrors/">Mirror List</a></li>
        <li> <a href="http://wiki.centos.org/irc">IRC</a></li>
        <li> <a href="https://www.centos.org/forums/">Forums</a></li>
        <li> <a href="http://bugs.centos.org/">Bugs</a> </li>
        <li class="last"> <a href="http://wiki.centos.org/Donate">Donate</a></li>
    </ul>

	<div class="logo">
		<a href="http://www.centos.org/"><img src="img/centos-logo.png" border="0"></a>
	</div>

</div>

<div id="content">

	<h1>Welcome to CentOS</h1>

	<h2>The Community ENTerprise Operating System</h2>

	<p><a href="http://www.centos.org/">CentOS</a> is an Enterprise-class Linux Distribution derived from sources freely provided
to the public by Red Hat, Inc. for Red Hat Enterprise Linux.  CentOS conforms fully with the upstream vendors
redistribution policy and aims to be functionally compatible. (CentOS mainly changes packages to remove upstream vendor
branding and artwork.)</p>

	<p>CentOS is developed by a small but growing team of core
developers.&nbsp; In turn the core developers are supported by an active user community
including system administrators, network administrators, enterprise users, managers, core Linux contributors and Linux enthusiasts from around the world.</p>

	<p>CentOS has numerous advantages including: an active and growing user community, quickly rebuilt, tested, and QA'ed errata packages, an extensive <a href="http://www.centos.org/download/mirrors/">mirror network</a>, developers who are contactable and responsive, Special Interest Groups (<a href="http://wiki.centos.org/SpecialInterestGroup/">SIGs</a>) to add functionality to the core CentOS distribution, and multiple community support avenues including a <a href="http://wiki.centos.org/">wiki</a>, <a
href="http://wiki.centos.org/irc">IRC Chat</a>, <a href="http://wiki.centos.org/GettingHelp/ListInfo">Email Lists</a>, <a href="https://www.centos.org/forums/">Forums</a>, <a href="http://bugs.centos.org/">Bugs Database</a>, and an <a
href="http://wiki.centos.org/FAQ/">FAQ</a>.</p>

	</div>

</div>


</body>
</html>

```
### NGINX на Ubuntu
```
roman@root-ubuntu:~$ curl 192.168.56.28:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

```