# Docker for Gogs(中文翻译)

可以在 [Docker Hub](https://hub.docker.com/r/gogs/) 下查看所有可用镜像和标签。
## 用法

为了实现数据持久化，可以先新建一个数据卷容器(`/var/gogs` -> `/data`)，当然你也可以根据实际情况修改数据卷路径。

```
# 从Docker Hub拉取gogs镜像。
$ docker pull gogs/gogs

# 为数据卷容器创建本地目录(宿主机目录)。
$ mkdir -p /var/gogs

# 首次可以使用 `docker run` 命令创建gogs容器。
$ docker run --name=gogs -p 10022:22 -p 10080:3000 -v /var/gogs:/data gogs/gogs

# 以后当容器停止时可以通过 `docker start` 命令启动。
$ docker start gogs
```

注意：
Note: It is important to map the Gogs ssh service from the container to the host and set the appropriate SSH Port and URI settings when setting up Gogs for the first time. To access and clone Gogs Git repositories with the above configuration you would use: `git clone ssh://git@hostname:10022/username/myrepo.git` for example.
>译者注：

如上配置，文件会被保存到（宿主机）本地路径 `/var/gogs`下。 

`/var/gogs` 路径下存放Git版本库以及Gogs数据：

    /var/gogs
    |-- git
    |   |-- gogs-repositories
    |-- ssh
    |   |-- # ssh public/private keys for Gogs
    |-- gogs
        |-- conf
        |-- data
        |-- log

### Volume with data container

If you're more comfortable with mounting data to a data container, the commands you execute at the first time will look like as follows:

```
# 创建数据容器
docker run --name=gogs-data --entrypoint /bin/true gogs/gogs

# 第一次需要使用 `docker run`运行容器。
docker run --name=gogs --volumes-from gogs-data -p 10022:22 -p 10080:3000 gogs/gogs
```

#### 使用Docker 1.9版本中新增的Volume命令

```
# 创建docker volume。
$ docker volume create --name gogs-data

# 第一次需要使用 `docker run`运行容器。
$ docker run --name=gogs -p 10022:22 -p 10080:3000 -v gogs-data:/data gogs/gogs
```

## 设置

### 应用

大多数的配置是非常明了、易于理解的，但是在Docker下运行Gogs仍有一些令人困惑的配置：

- **Repository Root Path**: 使用他的默认值`/home/git/gogs-repositories`就行因为在`start.sh`脚本中已经为你创建了一个软链接。
- **Run User**: 最好使用`git`因为在`start.sh`脚本中已经新建了一个名为`git`的用户。
- **Domain**: 一般情况填写Dokcer容器的ip地址(例如： `192.168.99.100`)。但是如果当你需要在其他的物理机访问你的Gogs服务，这里应该填写Docker宿主机的主机名或ip地址。
- **SSH Port**: Use the exposed port from Docker container. For example, your SSH server listens on `22` inside Docker, but you expose it by `10022:22`, then use `10022` for this value. **Builtin SSH server is not recommended inside Docker Container**
- **HTTP Port**: Use port you want Gogs to listen on inside Docker container. For example, your Gogs listens on `3000` inside Docker, and you expose it by `10080:3000`, but you still use `3000` for this value.
- **Application URL**: 通过**Domain** 和 **exposed HTTP Port** 的值组合起来 (例如：`http://192.168.99.100:10080/`).

需要查看完整的应用文档配置手册可以点击 [here](http://gogs.io/docs/advanced/configuration_cheat_sheet.html).

### 容器参数

This container have some options available via environment variables, these options are opt-in features that can help the administration of this container:

- **SOCAT_LINK**:
  - <u>Possible value:</u>
      `true`, `false`, `1`, `0`
  - <u>Default:</u>
      `true`
  - <u>Action:</u>
      Bind linked docker container to localhost socket using socat.
      Any exported port from a linked container will be binded to the matching port on localhost.
  - <u>Disclaimer:</u>
      As this option rely on the environment variable created by docker when a container is linked, this option should be deactivated in managed environment such as Rancher or Kubernetes (set to `0` or `false`)
- **RUN_CROND**:
  - <u>Possible value:</u>
      `true`, `false`, `1`, `0`
  - <u>Default:</u>
      `false`
  - <u>Action:</u>
      Request crond to be run inside the container. Its default configuration will periodically run all scripts from `/etc/periodic/${period}` but custom crontabs can be added to `/var/spool/cron/crontabs/`.

## 升级

:exclamation::exclamation::exclamation:<span style="color: red">**确保你已经将数据打卷备份到Docker容器以外的地方**</span>:exclamation::exclamation::exclamation:

在Docker下升级Gogs的步骤：

- `docker pull gogs/gogs`
- `docker stop gogs`
- `docker rm gogs`
- 最后, 像第一次一样创建容器，并且不要忘了使用以前的数据卷以及做端口映射。

## 已知的问题

- The docker container can not currently be build on Raspberry 1 (armv6l) as our base image `alpine` does not have a `go` package available for this platform.
