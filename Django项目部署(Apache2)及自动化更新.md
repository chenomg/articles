> 本篇介绍Django项目部署(Apache2)，并使用paramiko自动化更新Django项目  

### 1. Django项目部署(Apache2)  

#### 1.1 安装相关应用  

##### 运行环境及对应版本
> 以下为本项目使用的环境, 若你的环境不同请参考

* OS: Ubuntu 16.04
* Apache: 2.4
* Python: 3.5
* Django: 1.11

##### 连接Apache2和Django
> 在Apache2下通过wsgi接口托管基于Python的Django应用

Python3:  
​	`$ sudo apt-get install libapache2-mod-wsgi-py3`  

Python2:  
​	`$ sudo apt-get install libapache2-mod-wsgi`

#### 1.2 设置Django项目wsgi.py  

*以下为wsgi.py配置供参考:*  
*FILE:  
​	`{Django_project_dir}/{Django_project_name}/wsgi.py`*  

```python
"""
WSGI config for jase_im project.

It exposes the WSGI callable as a module-level variable named ``application``.

For more information on this file, see
https://docs.djangoproject.com/en/1.11/howto/deployment/wsgi/
"""

import os
import sys
from os.path import join, dirname, abspath
from django.core.wsgi import get_wsgi_application

# 指定项目所在目录并将其加入系统路径,即manage.py所在目录
PROJECT_DIR = dirname(dirname(abspath(__file__)))
sys.path.insert(0, PROJECT_DIR)

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "jase_im.settings")

application = get_wsgi_application()
```
#### 1.3 设置Apache2  
##### 主要步骤  

1. 将Django项目文件放在服务器(可以放在`~/www/{yoursite}/`)
2. 修改Apache2的站点配置文件:  
  `$ sudo vi /etc/apache2/sites-available/{yoursite}.conf`
3. 使站点配置文件生效:  
  `$ sudo a2ensite {yoursite}.conf`  
  失效:  
  `$ sudo a2dissite {yoursite}.conf`
4. 重启Apache2服务:  
  `$ sudo service apache2 restart`

**Django项目变更之后记得restart一下Apache2**

*以下为站点配置供参考:*  

*FILE:  
​	`/etc/apache2/sites-available/{yoursite}.conf`*  
​	
```
<VirtualHost *:80>
    # 网站域名或IP
    ServerName {hostname}
    # 网站别名或IP
    ServerAlias {hostname_A}
    # 管理员邮箱
    ServerAdmin {email}

    # 媒体文件存放位置
    Alias /media/ {Django_project_dir}/media/

    # 静态文件的存放位置
    Alias /static/ {Django_project_dir}/collected_static/

    # 允许通过网络获取媒体文件
    <Directory {Django_project_dir}/media/>
        Require all granted
    </Directory>

    # 允许通过网络获取静态文件(CSS/JS/images)
    <Directory {Django_project_dir}/collected_static/>
        Require all granted
    </Directory>

    # 通过wsgi.py让Apache2识别Django项目, 注意前面的'/'
    WSGIScriptAlias / {Django_project_dir}/{Django_project_name}/wsgi.py

    # wsgi.py文件所在的目录
    <Directory {Django_project_dir}/{Django_project_name}/>
    <Files wsgi.py>
        Require all granted
    </Files>
    </Directory>
</VirtualHost>
```


#### 1.4 设置权限及其他  

-1. 一般目录设置为755, 文件权限644

-2. 设定上传文件夹及权限

*FILE: settings.py*

```python
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
STATIC_ROOT = os.path.join(BASE_DIR, 'collected_static')
STATIC_URL = '/static/'
MEDIA_URL = '/media/'
```

media文件夹可写入

-3. 如果使用sqlite3, 设置权限用户可写入

-4. 设定Apache2服务器运行用户,设置为当前用户

*FILE:*  
​	`/etc/apache2/envvars`  

```
# Since there is no sane way to get the parsed apache2 config in scripts, some
# settings are defined via environment variables and then used in apache2ctl,
# /etc/init.d/apache2, /etc/logrotate.d/apache2, etc.
 
export APACHE_RUN_USER={username}
export APACHE_RUN_GROUP={username}
```



#### 1.5 常见错误  

1. 网站打开后页面错乱  
  原因分析:  
  * 可能是静态文件及媒体文件路径设置不正确,检查settings.py  
  * 收集静态文件:  
    `$ sudo python3 manage.py collectstatic`  
2. 未关闭DEBUG, 在settings中设置DEBUG = False
3. 若还是无法判断错误原因,查看Apache错误日志:  
  `$ cat /var/log/apache2/error.log`

----

### 2. 项目自动更新  

项目部署到服务器后,每次更新服务器需要执行以下操作  

> 1. 连接远程服务器  
> 2. 进入Django项目根目录  
> 3. 拉取远程仓库:  
>   `$ git fetch && git pull`  
> 4. 如加入新的app, 则需要安装新的依赖:  
>   `$ pip3 install -r requirements.txt`  
> 5. 如静态文件变更则需要重新收集静态文件:  
>   `$ python3 mange.py collectstatic`  
> 6. 如数据库结构发生变化需要执行迁移:  
>   `$ python3 manage.py makemigrations && python3 manage.py migrate`  
> 7. 重启Apache2服务器  

#### 2.1 使用的工具  

**paramiko**: paramiko是用python语言写的一个模块，遵循SSH2协议，支持以加密和认证的方式，进行远程服务器的连接。
> paramiko源代码托管在GitHub, 文档详见[Paramiko](http://www.paramiko.org/ "Paramiko")

安装: `$ pip3 install paramiko`

#### 2.2 自动更新示例  

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
import os
import paramiko
import json

# 用于存放服务器登陆信息的文件, 格式如下
"""
(KEY_FILE: host.key) config sample
{
    "hostname": "hostname/ip",
    "port": port,
    "username": "username",
    "password": "password",
    "dir": "django_project_dir"
}
"""
KEY_FILE = 'host.key'


# 获取服务器数据
def get_config(key_file=KEY_FILE):
    with open(key_file, 'r') as k:
        _key = json.load(fp=k)
    return _key


config = get_config()
# 远程执行自动部署代码
cmd = 'cd ' + config['dir'] + ';\
    rm -r collected_static;\
    git fetch;\
    git reset --hard origin/master;\
    git pull;\
    cd ..;\
    pip3 install -r requirements.txt;\
    cd jase_im;\
    python3 manage.py makemigrations;\
    python3 manage.py migrate;\
    python3 manage.py collectstatic;\
    service apache2 restart'

client = paramiko.SSHClient()
client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
client.load_system_host_keys()
client.connect(config['hostname'], config['port'], config['username'], config['password'])
stdin, stdout, stderr = client.exec_command(cmd)
print(stdout.read().decode('utf-8'))
print('Process Done')
```