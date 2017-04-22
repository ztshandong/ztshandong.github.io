# [docker gitbook](https://www.gitbook.com/book/yeasy/docker_practice/details)
# [aliyun docker](https://dev.aliyun.com/search.html)
# [hub docker](https://hub.docker.com/explore/)
# [163](https://c.163.com/hub#/m/home/)
# [搭建docker](http://morning.work/page/2016-01/deploying-your-own-private-docker-registry.html)
- 阿里云-容器服务-镜像-镜像仓库控制台    

# MySQL
```sh
docker pull mysql
docker run -p 3306:3306 --name mysql -v /data/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=19820108 -eMYSQL_DATABASE=testDB -e MYSQL_USER=testuser -e MYSQL_PASSWORD=12345678 --restart=always -d mysql
```






