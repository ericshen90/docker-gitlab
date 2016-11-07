

# Docker-Gitlab部署

基于docker镜像sameersbn/docker-gitlab

### Quick Start

使用docker-compose快速启动Gitlab

```
$ wget https://github.com/tonywell/docker-gitlab/blob/master/docker-compose.yml
```

* GITLAB_SECRETS_OTP_KEY_BASE   如果从原来的gitlab进行数据迁移，就使用老的gitlab的db_key_base,位置应该在gitlab-8.9.4-0/apps/gitlab/htdocs/config/secrets.yml文件中



然后启动gitlab

```
$ docker-compose up -d
```



#### 注意事项

这里使用的是docker-compose，最好安装docker-compose的最新版

```
curl -L https://github.com/docker/compose/releases/download/1.8.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

compose模版中redis、postgresql和gitlab使用的是大牛已经做好的镜像，如果从docker仓库pull太慢的话，建议使用国内的加速器，我这里使用的是阿里的加速器

```
# 系统要求 CentOS 7 以上，Docker 1.9 以上。 
$ sudo cp -n /lib/systemd/system/docker.service /etc/systemd/system/docker.service
```

```
$ vim /etc/systemd/system/docker.service
```

修改ExecStart如下

```
ExecStart=/usr/bin/dockerd --registry-mirror=https://28b8dhu0.mirror.aliyuncs.com
```

重启

```
$ sudo systemctl daemon-reload 
$ sudo service docker restart
```





### gitlab数据迁移

如果要从老的gitlab迁移到docker-gitlab分为以下几步

#### 一、备份老数据库

在老的postgresql数据中执行：

```
$ pg_dump --inserts -U gitlab -a -f bak.sql gitlabhq_production
```

关键是--inserts这个选项，可以将数据导成insert脚本。

gitlab 为用户名

gitlabhq_production为数据库名称，-a参数是只导出数据，而不包含建表的sql。

#### 二、在新的数据库中导入数据

在docker postgresql数据库中导入数据

```
$ psql -d  gitlabhq_production  -f  /XXXX/dump.sql gitlab
```

#### 三、迁移repositories

将老的gitlab的repositories文件拷贝到宿主机的/xxx/data/下，/xxx/data/为compose中配置的路径

注意这里要删除掉项目目录下的hook*目录

然后登陆到gitlab容器

```
$ docker exec -it gitlab_gitlab_1 bash
```

运行下面命令：

```
$ sudo -u git -H bundle exec rake gitlab:import:repos RAILS_ENV=production
```

可以运行下面命令检查

```
$ sudo -u git -H bundle exec rake gitlab:check RAILS_ENV=production  
```

