# 第十一章：容器技术：Docker 和 Docker Compose

虚拟化技术自 IBM 大型机时代就已存在。大多数人没有机会使用过大型机，但我们相信本书的一些读者还记得他们曾经在惠普或戴尔等制造商那里设置或使用裸金属服务器的日子。这些制造商今天仍然存在，你仍然可以在像互联网泡沫时代那样的机房中使用裸金属服务器。

然而，当大多数人想到虚拟化时，他们不会自动想到大型机。相反，他们很可能想象的是在虚拟化管理程序（如 VMware ESX 或 Citrix/Xen）上运行的虚拟机（VM），并且运行了 Fedora 或 Ubuntu 等客户操作系统（OS）。虚拟机相对于普通裸金属服务器的一大优势是，通过使用虚拟机，你可以通过在几个虚拟机之间分割它们来优化服务器的资源（CPU、内存、磁盘）。你还可以在一个共享的裸金属服务器上运行几个操作系统，每个操作系统在自己的虚拟机中运行，而不是为每个目标操作系统购买专用服务器。像亚马逊 EC2 这样的云计算服务如果没有虚拟化管理程序和虚拟机是不可能的。这种类型的虚拟化可以称为内核级，因为每个虚拟机运行其自己的操作系统内核。

在对资源追求更大回报的不懈努力中，人们意识到虚拟机在资源利用上仍然是浪费的。下一个逻辑步骤是将单个应用程序隔离到自己的虚拟环境中。通过在同一个操作系统内核中运行容器来实现这一目标。在这种情况下，它们在文件系统级别被隔离。Linux 容器（LXC）和 Sun Solaris zones 是这种技术的早期示例。它们的缺点是使用起来很困难，并且与它们运行的操作系统紧密耦合。在容器使用方面的重大突破是当 Docker 开始提供一种简便的方法来管理和运行文件系统级容器。

# 什么是 Docker 容器？

一个 Docker 容器封装了一个应用程序以及它运行所需的其他软件包和库。人们有时会将 Docker 容器和 Docker 镜像的术语互换使用，但它们之间是有区别的。封装应用程序的文件系统级对象称为 Docker 镜像。当你运行该镜像时，它就成为了一个 Docker 容器。

你可以运行许多 Docker 容器，它们都使用相同的操作系统内核。唯一的要求是你必须在要运行容器的主机上安装一个称为 Docker 引擎或 Docker 守护程序的服务器端组件。通过这种方式，主机资源可以在容器之间以更精细的方式分割和利用，让你的投资得到更大的回报。

Docker 容器提供了比常规 Linux 进程更多的隔离和资源控制，但提供的功能不及完整的虚拟机。为了实现这些隔离和资源控制的特性，Docker 引擎利用了 Linux 内核功能，如命名空间、控制组（或 cgroups）和联合文件系统（UnionFS）。

Docker 容器的主要优势是可移植性。一旦创建了 Docker 镜像，您可以在任何安装有 Docker 服务器端守护程序的主机操作系统上作为 Docker 容器运行它。如今，所有主要操作系统都运行 Docker 守护程序：Linux、Windows 和 macOS。

所有这些可能听起来太理论化，所以现在是一些具体示例的时候了。

# 创建、构建、运行和删除 Docker 镜像和容器

由于这是一本关于 Python 和 DevOps 的书籍，我们将以经典的 Flask “Hello World” 作为在 Docker 容器中运行的应用程序的第一个示例。本节中显示的示例使用 Docker for Mac 包。后续章节将展示如何在 Linux 上安装 Docker。

这是 Flask 应用程序的主文件：

```py
$ cat app.py
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World! (from a Docker container)'

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0')
```

我们还需要一个需求文件，其中指定了要与 `pip` 安装的 Flask 包的版本：

```py
$ cat requirements.txt
Flask==1.0.2
```

在 macOS 笔记本电脑上直接使用 Python 运行 *app.py* 文件，而不先安装要求，会导致错误：

```py
$ python app.py
Traceback (most recent call last):
  File "app.py", line 1, in <module>
    from flask import Flask
ImportError: No module named flask
```

要解决这个问题的一个明显方法是在本地机器上使用 `pip` 安装需求。这将使一切都与您本地运行的操作系统具体相关。如果应用程序需要部署到运行不同操作系统的服务器上，该怎么办？众所周知的“在我的机器上能运行”的问题可能会出现，即在 macOS 笔记本电脑上一切运行良好，但由于操作系统特定版本的 Python 库，一切在运行其他操作系统（如 Ubuntu 或 Red Hat Linux）的暂存或生产服务器上却突然崩溃。

Docker 为这个难题提供了一个优雅的解决方案。我们仍然可以在本地进行开发，使用我们喜爱的编辑器和工具链，但是我们将应用程序的依赖项打包到一个可移植的 Docker 容器中。

这是描述将要构建的 Docker 镜像的 Dockerfile：

```py
$ cat Dockerfile
FROM python:3.7.3-alpine

ENV APP_HOME /app
WORKDIR $APP_HOME

COPY requirements.txt .

RUN pip install -r requirements.txt

ENTRYPOINT [ "python" ]
CMD [ "app.py" ]
```

关于这个 Dockerfile 的一些说明：

+   使用基于 Alpine 发行版的 Python 3.7.3 预构建 Docker 镜像，这样可以生成更轻量的 Docker 镜像；这个 Docker 镜像已经包含了诸如 `python` 和 `pip` 等可执行文件。

+   使用 `pip` 安装所需的包。

+   指定一个 ENTRYPOINT 和一个 CMD。两者的区别在于，当 Docker 容器运行从这个 Dockerfile 构建的镜像时，它运行的程序是 ENTRYPOINT，后面跟着在 CMD 中指定的任何参数；在本例中，它将运行 `python app.py`。

###### 注意

如果您没有在您的 Dockerfile 中指定 ENTRYPOINT，则将使用以下默认值：`/bin/sh -c`。

要为这个应用程序创建 Docker 镜像，运行 `docker build`：

```py
$ docker build -t hello-world-docker .
```

要验证 Docker 镜像是否已保存在本地，请运行`docker images`，然后输入镜像名称：

```py
$ docker images hello-world-docker
REPOSITORY               TAG       IMAGE ID            CREATED          SIZE
hello-world-docker       latest    dbd84c229002        2 minutes ago    97.7MB
```

要将 Docker 镜像作为 Docker 容器运行，请使用`docker run`命令：

```py
$ docker run --rm -d -v `pwd`:/app -p 5000:5000 hello-world-docker
c879295baa26d9dff1473460bab810cbf6071c53183890232971d1b473910602
```

关于`docker run`命令参数的几点说明：

+   `--rm`参数告诉 Docker 服务器在停止运行后删除此容器。这对于防止旧容器堵塞本地文件系统非常有用。

+   `-d`参数告诉 Docker 服务器在后台运行此容器。

+   `-v`参数指定当前目录（*pwd*）映射到 Docker 容器内的*/app*目录。这对于我们想要实现的本地开发工作流至关重要，因为它使我们能够在本地编辑应用程序文件，并通过运行在容器内部的 Flask 开发服务器进行自动重新加载。

+   `-p 5000:5000`参数将本地的第一个端口（5000）映射到容器内部的第二个端口（5000）。

要列出运行中的容器，请运行`docker ps`并注意容器 ID，因为它将在其他`docker`命令中使用：

```py
$ docker ps
CONTAINER ID  IMAGE                      COMMAND         CREATED
c879295baa26  hello-world-docker:latest  "python app.py" 4 seconds ago
STATUS          PORTS                    NAMES
Up 2 seconds    0.0.0.0:5000->5000/tcp   flamboyant_germain
```

要检查特定容器的日志，请运行`docker logs`并指定容器名称或 ID：

```py
$ docker logs c879295baa26
 * Serving Flask app "app" (lazy loading)
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 647-161-014
```

使用`curl`命中端点 URL 以验证应用程序是否正常工作。由于使用`-p`命令行标志将运行在 Docker 容器内部的应用程序的端口 5000 映射到本地机器的端口 5000，因此可以使用本地 IP 地址 127.0.0.1 作为应用程序的端点地址。

```py
$ curl http://127.0.0.1:5000
Hello, World! (from a Docker container)%
```

现在用你喜欢的编辑器修改*app.py*中的代码。将问候文本更改为*Hello, World! (from a Docker container with modified code)*。保存*app.py*并注意 Docker 容器日志中类似以下行：

```py
 * Detected change in '/app/app.py', reloading
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 647-161-014
```

这表明运行在容器内部的 Flask 开发服务器已检测到*app.py*中的更改，并重新加载了应用程序。

使用`curl`命中应用程序端点将显示修改后的问候语：

```py
$ curl http://127.0.0.1:5000
Hello, World! (from a Docker container with modified code)%
```

要停止运行中的容器，请运行`docker stop`或`docker kill`，并指定容器 ID 作为参数：

```py
$ docker stop c879295baa26
c879295baa26
```

要从本地磁盘中删除 Docker 镜像，请运行`docker rmi`：

```py
$ docker rmi hello-world-docker
Untagged: hello-world-docker:latest
Deleted:sha256:dbd84c229002950550334224b4b42aba948ce450320a4d8388fa253348126402
Deleted:sha256:6a8f3db7658520a1654cc6abee8eafb463a72ddc3aa25f35ac0c5b1eccdf75cd
Deleted:sha256:aee7c3304ef6ff620956850e0b6e6b1a5a5828b58334c1b82b1a1c21afa8651f
Deleted:sha256:dca8a433d31fa06ab72af63ae23952ff27b702186de8cbea51cdea579f9221e8
Deleted:sha256:cb9d58c66b63059f39d2e70f05916fe466e5c99af919b425aa602091c943d424
Deleted:sha256:f0534bdca48bfded3c772c67489f139d1cab72d44a19c5972ed2cd09151564c1
```

此输出显示组成 Docker 镜像的不同文件系统层。当删除镜像时，这些层也将被删除。有关 Docker 如何使用文件系统层构建其镜像的详细信息，请参阅[Docker 存储驱动程序](https://oreil.ly/wqNve)文档。

# 发布 Docker 镜像到 Docker 注册表

一旦在本地构建了 Docker 镜像，就可以将其发布到所谓的 Docker 注册表中。有几个公共注册表可供选择，本示例将使用 Docker Hub。这些注册表的目的是允许个人和组织共享可在不同机器和操作系统上重复使用的预构建 Docker 镜像。

首先，在[Docker Hub](https://hub.docker.com)上创建一个免费帐户，然后创建一个仓库，可以是公共的或私有的。我们在`griggheo`的 Docker Hub 帐户下创建了一个名为`flask-hello-world`的私有仓库。

然后，在命令行上运行`docker login`并指定您帐户的电子邮件和密码。此时，您可以通过`docker`客户端与 Docker Hub 进行交互。

###### 注意

在向 Docker Hub 发布您本地构建的 Docker 镜像之前，我们要指出的最佳实践是使用唯一标签对镜像进行标记。如果不明确打标签，默认情况下镜像将标记为`latest`。发布不带标签的新镜像版本将把`latest`标签移至最新镜像版本。在使用 Docker 镜像时，如果不指定所需的确切标签，将获取`latest`版本的镜像，其中可能包含可能破坏依赖关系的修改和更新。始终应用最小惊讶原则：在推送镜像到注册表时和在 Dockerfile 中引用镜像时都应使用标签。话虽如此，您也可以将所需版本的镜像标记为`latest`，以便对最新和最伟大的人感兴趣的人使用而不需要指定标签。

在上一节中构建 Docker 镜像时，它会自动标记为`latest`，并且仓库被设置为镜像的名称，表示该镜像是本地的：

```py
$ docker images hello-world-docker
REPOSITORY               TAG       IMAGE ID            CREATED          SIZE
hello-world-docker       latest    dbd84c229002        2 minutes ago    97.7MB
```

要为 Docker 镜像打标签，请运行`docker tag`：

```py
$ docker tag hello-world-docker hello-world-docker:v1
```

现在你可以看到`hello-world-docker`镜像的两个标签：

```py
$ docker images hello-world-docker
REPOSITORY               TAG      IMAGE ID           CREATED          SIZE
hello-world-docker       latest   dbd84c229002       2 minutes ago    97.7MB
hello-world-docker       v1       89bd38cb198f       42 seconds ago   97.7MB
```

在将`hello-world-docker`镜像发布到 Docker Hub 之前，您还需要使用包含您的用户名或组织名称的 Docker Hub 仓库名称对其进行标记。在我们的情况下，这个仓库是`griggheo/hello-world-docker`：

```py
$ docker tag hello-world-docker:latest griggheo/hello-world-docker:latest
$ docker tag hello-world-docker:v1 griggheo/hello-world-docker:v1
```

使用`docker push`将两个镜像标签发布到 Docker Hub：

```py
$ docker push griggheo/hello-world-docker:latest
$ docker push griggheo/hello-world-docker:v1
```

如果您跟随进行，现在应该能够看到您的 Docker 镜像已发布到您在帐户下创建的 Docker Hub 仓库，并带有两个标签。

# 在不同主机上使用相同镜像运行 Docker 容器

现在 Docker 镜像已发布到 Docker Hub，我们可以展示 Docker 的可移植性，通过在不同主机上基于已发布镜像运行容器来展示。这里考虑的场景是与一位没有 macOS 但喜欢在运行 Fedora 的笔记本上开发的同事合作。该场景包括检出应用程序代码并进行修改。

在 AWS 上启动基于 Linux 2 AMI 的 EC2 实例，该 AMI 基于 RedHat/CentOS/Fedora，并安装 Docker 引擎。将 EC2 Linux AMI 上的默认用户`ec2-user`添加到`docker`组，以便可以运行`docker`客户端命令。

```py
$ sudo yum update -y
$ sudo amazon-linux-extras install docker
$ sudo service docker start
$ sudo usermod -a -G docker ec2-user
```

确保在远程 EC2 实例上检出应用程序代码。在这种情况下，代码仅包括*app.py*文件。

接下来，运行基于发布到 Docker Hub 的镜像的 Docker 容器。唯一的区别是，作为`docker run`命令参数使用的镜像是`griggheo/hello-world-docker:v1`，而不仅仅是`hello-world-docker`。

运行`docker login`，然后：

```py
$ docker run --rm -d -v `pwd`:/app -p 5000:5000 griggheo/hello-world-docker:v1

Unable to find image 'griggheo/hello-world-docker:v1' locally
v1: Pulling from griggheo/hello-world-docker
921b31ab772b: Already exists
1a0c422ed526: Already exists
ec0818a7bbe4: Already exists
b53197ee35ff: Already exists
8b25717b4dbf: Already exists
d997915c3f9c: Pull complete
f1fd8d3cc5a4: Pull complete
10b64b1c3b21: Pull complete
Digest: sha256:af8b74f27a0506a0c4a30255f7ff563c9bf858735baa610fda2a2f638ccfe36d
Status: Downloaded newer image for griggheo/hello-world-docker:v1
9d67dc321ffb49e5e73a455bd80c55c5f09febc4f2d57112303d2b27c4c6da6a
```

请注意，EC2 实例上的 Docker 引擎会意识到本地没有 Docker 镜像，因此会从 Docker Hub 下载镜像，然后基于新下载的镜像运行容器。

在此时，通过向与 EC2 实例关联的安全组添加规则来授予对端口 5000 的访问权限。访问 http://54.187.189.51:5000¹（其中 54.187.189.51 是 EC2 实例的外部 IP）并查看问候语*Hello, World! (from a Docker container with modified code)*。

在远程 EC2 实例上修改应用程序代码时，运行在 Docker 容器内部的 Flask 服务器将自动重新加载修改后的代码。将问候语更改为*Hello, World! (from a Docker container on an EC2 Linux 2 AMI instance)*，并注意 Flask 服务器通过检查 Docker 容器的日志重新加载了应用程序：

```py
[ec2-user@ip-10-0-0-111 hello-world-docker]$ docker ps
CONTAINER ID  IMAGE                           COMMAND         CREATED
9d67dc321ffb  griggheo/hello-world-docker:v1  "python app.py" 3 minutes ago
STATUS        PORTS                    NAMES
Up 3 minutes  0.0.0.0:5000->5000/tcp   heuristic_roentgen

[ec2-user@ip-10-0-0-111 hello-world-docker]$ docker logs 9d67dc321ffb
 * Serving Flask app "app" (lazy loading)
 * Debug mode: on
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 306-476-204
72.203.107.13 - - [19/Aug/2019 04:43:34] "GET / HTTP/1.1" 200 -
72.203.107.13 - - [19/Aug/2019 04:43:35] "GET /favicon.ico HTTP/1.1" 404 -
 * Detected change in '/app/app.py', reloading
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 306-476-204
```

点击 http://54.187.189.51:5000²现在显示新的问候语*Hello, World! (from a Docker container on an EC2 Linux 2 AMI instance)*。

值得注意的是，为了使我们的应用程序运行，我们没有必要安装任何与 Python 或 Flask 相关的东西。通过简单地在容器内运行我们的应用程序，我们能够利用 Docker 的可移植性。Docker 选择“容器”作为技术的名字并不是没有原因的，其中一个灵感来源于运输容器如何革命了全球运输行业。

###### 提示

阅读[“Production-ready Docker images”](https://pythonspeed.com/docker)一书，由 Itamar Turner-Trauring 编写，涵盖了关于 Python 应用程序 Docker 容器打包的大量文章。

# 使用 Docker Compose 运行多个 Docker 容器

在本节中，我们将使用[“Flask By Example”](https://oreil.ly/prNg7)教程，该教程描述了如何构建一个 Flask 应用程序，根据给定 URL 的文本计算单词频率对。

首先克隆[Flask By Example GitHub 存储库](https://oreil.ly/M-pvc)：

```py
$ git clone https://github.com/realpython/flask-by-example.git
```

我们将使用`compose`来运行代表示例应用程序不同部分的多个 Docker 容器。使用 Compose，您可以使用 YAML 文件定义和配置组成应用程序的服务，然后使用`docker-compose`命令行实用程序来创建、启动和停止这些将作为 Docker 容器运行的服务。

示例应用程序的第一个依赖项是 PostgreSQL，在[教程第二部分](https://oreil.ly/iobKp)中有描述。

这是如何在*docker-compose.yaml*文件中运行 PostgreSQL Docker 容器的方法：

```py
$ cat docker-compose.yaml
version: "3"
services:
  db:
    image: "postgres:11"
    container_name: "postgres"
    ports:
      - "5432:5432"
    volumes:
      - dbdata:/var/lib/postgresql/data
volumes:
  dbdata:
```

关于这个文件的几个注意事项：

+   定义一个名为`db`的服务，基于在 Docker Hub 上发布的`postgres:11`镜像。

+   将本地端口 5432 映射到容器端口 5432。

+   为 PostgreSQL 存储其数据的目录（*/var/lib/postgresql/data*）指定一个 Docker 卷。这样做是为了确保 PostgreSQL 中存储的数据在容器重新启动后仍然存在。

`docker-compose`工具不是 Docker 引擎的一部分，因此需要单独安装。请参阅[官方文档](https://docs.docker.com/compose/install)以获取在各种操作系统上安装的说明。

要启动*docker-compose.yaml*中定义的`db`服务，请运行`docker-compose up -d db`命令，该命令将在后台启动`db`服务的 Docker 容器（使用了`-d`标志）。

```py
$ docker-compose up -d db
Creating postgres ... done
```

使用`docker-compose logs db`命令检查`db`服务的日志：

```py
$ docker-compose logs db
Creating volume "flask-by-example_dbdata" with default driver
Pulling db (postgres:11)...
11: Pulling from library/postgres
Creating postgres ... done
Attaching to postgres
postgres | PostgreSQL init process complete; ready for start up.
postgres |
postgres | 2019-07-11 21:50:20.987 UTC [1]
LOG:  listening on IPv4 address "0.0.0.0", port 5432
postgres | 2019-07-11 21:50:20.987 UTC [1]
LOG:  listening on IPv6 address "::", port 5432
postgres | 2019-07-11 21:50:20.993 UTC [1]
LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
postgres | 2019-07-11 21:50:21.009 UTC [51]
LOG:  database system was shut down at 2019-07-11 21:50:20 UTC
postgres | 2019-07-11 21:50:21.014 UTC [1]
LOG:  database system is ready to accept connections
```

运行`docker ps`命令可以显示运行 PostgreSQL 数据库的容器：

```py
$ docker ps
dCONTAINER ID   IMAGE   COMMAND    CREATED   STATUS   PORTS   NAMES
83b54ab10099 postgres:11 "docker-entrypoint.s…"  3 minutes ago  Up 3 minutes
        0.0.0.0:5432->5432/tcp   postgres
```

运行`docker volume ls`命令显示已为 PostgreSQL */var/lib/postgresql/data*目录挂载的`dbdata` Docker 卷：

```py
$ docker volume ls | grep dbdata
local               flask-by-example_dbdata
```

要连接到运行在与`db`服务相关联的 Docker 容器中的 PostgreSQL 数据库，运行命令`docker-compose exec db`并传递`psql -U postgres`命令行：

```py
$ docker-compose exec db psql -U postgres
psql (11.4 (Debian 11.4-1.pgdg90+1))
Type "help" for help.

postgres=#
```

参照[“Flask by Example, Part 2”](https://oreil.ly/iobKp)，创建一个名为`wordcount`的数据库：

```py
$ docker-compose exec db psql -U postgres
psql (11.4 (Debian 11.4-1.pgdg90+1))
Type "help" for help.

postgres=# create database wordcount;
CREATE DATABASE

postgres=# \l
```

```py
                            List of databases
     Name  |  Owner | Encoding |  Collate |   Ctype  |   Access privileges
-----------+--------+----------+----------+----------+--------------------
 postgres  | postgres | UTF8   | en_US.utf8 | en_US.utf8 |
 template0 | postgres | UTF8   | en_US.utf8 | en_US.utf8 | =c/postgres +
           |          |        |            |            |postgres=CTc/postgres
 template1 | postgres | UTF8   | en_US.utf8 | en_US.utf8 | =c/postgres +
           |          |        |            |            |postgres=CTc/postgres
 wordcount| postgres | UTF8| en_US.utf8 | en_US.utf8 |
(4 rows)
postgres=# \q
```

连接到`wordcount`数据库并创建一个名为`wordcount_dbadmin`的角色，该角色将被 Flask 应用程序使用：

```py
$ docker-compose exec db psql -U postgres wordcount
wordcount=# CREATE ROLE wordcount_dbadmin;
CREATE ROLE
wordcount=# ALTER ROLE wordcount_dbadmin LOGIN;
ALTER ROLE
wordcount=# ALTER USER wordcount_dbadmin PASSWORD 'MYPASS';
ALTER ROLE
postgres=# \q
```

下一步是为 Flask 应用程序创建一个 Dockerfile，安装所有的先决条件。

对*requirements.txt*文件进行以下修改：

+   将`psycopg2`包的版本从`2.6.1`修改为`2.7`以支持 PostgreSQL 11。

+   将`redis`包的版本从`2.10.5`修改为`3.2.1`以提供更好的 Python 3.7 支持。

+   将`rq`包的版本从`0.5.6`修改为`1.0`以提供更好的 Python 3.7 支持。

以下是 Dockerfile 的内容：

```py
$ cat Dockerfile
FROM python:3.7.3-alpine

ENV APP_HOME /app
WORKDIR $APP_HOME

COPY requirements.txt .

RUN \
 apk add --no-cache postgresql-libs && \
 apk add --no-cache --virtual .build-deps gcc musl-dev postgresql-dev && \
 python3 -m pip install -r requirements.txt --no-cache-dir && \
 apk --purge del .build-deps

COPY . .

ENTRYPOINT [ "python" ]
CMD ["app.py"]
```

###### 注意

这个 Dockerfile 与第一个*hello-world-docker*示例中使用的版本有一个重要的区别。这里将当前目录的内容（包括应用程序文件）复制到 Docker 镜像中。这样做是为了展示与之前开发工作流不同的场景。在这种情况下，我们更关注以最便携的方式运行应用程序，例如在暂存或生产环境中，我们不希望像在开发场景中通过挂载卷来修改应用程序文件。在开发目的上通常可以使用`docker-compose`与本地挂载卷，但本节重点是讨论 Docker 容器在不同环境（如开发、暂存和生产）中的可移植性。

运行`docker build -t flask-by-example:v1 .`来构建一个本地 Docker 镜像。由于该命令的输出内容相当长，因此这里不显示。

“Flask By Example”教程的下一步是运行 Flask 迁移。

在*docker-compose.yaml*文件中，定义一个名为`migrations`的新服务，并指定其`image`、`command`、`environment`变量以及它依赖于`db`服务正在运行的事实：

```py
$ cat docker-compose.yaml
version: "3"
services:
  migrations:
    image: "flask-by-example:v1"
    command: "manage.py db upgrade"
    environment:
      APP_SETTINGS: config.ProductionConfig
      DATABASE_URL: postgresql://wordcount_dbadmin:$DBPASS@db/wordcount
    depends_on:
      - db
  db:
    image: "postgres:11"
    container_name: "postgres"
    ports:
      - "5432:5432"
    volumes:
      - dbdata:/var/lib/postgresql/data
volumes:
  dbdata:
```

`DATABASE_URL`变量使用`db`作为 PostgreSQL 数据库主机的名称。这是因为在*docker-compose.yaml*文件中将`db`定义为服务名称，并且`docker-compose`知道如何通过创建一个覆盖网络使所有在*docker-compose.yaml*文件中定义的服务能够通过其名称互相交互。有关更多详细信息，请参阅[docker-compose 网络参考](https://oreil.ly/Io80N)。

`DATABASE_URL`变量定义引用了另一个名为`DBPASS`的变量，而不是直接硬编码`wordcount_dbadmin`用户的密码。*docker-compose.yaml*文件通常提交到源代码控制，最佳实践是不要将诸如数据库凭据之类的机密信息提交到 GitHub。相反，使用诸如[`sops`](https://github.com/mozilla/sops)之类的加密工具管理密钥文件。

这里是使用`sops`和 PGP 加密创建加密文件的示例。

首先，在 macOS 上通过`brew install gpg`安装`gpg`，然后使用空密码生成新的 PGP 密钥：

```py
$ gpg --generate-key
pub   rsa2048 2019-07-12 [SC] [expires: 2021-07-11]
      E14104A0890994B9AC9C9F6782C1FF5E679EFF32
uid                      pydevops <my.email@gmail.com>
sub   rsa2048 2019-07-12 [E] [expires: 2021-07-11]
```

接下来，从其[发布页面](https://github.com/mozilla/sops/releases)下载`sops`。

要创建一个名为*environment.secrets*的新加密文件，例如，运行带有`-pgp`标志的`sops`并提供上述生成的密钥的指纹：

```py
$ sops --pgp BBDE7E57E00B98B3F4FBEAF21A1EEF4263996BD0 environment.secrets
```

这将打开默认编辑器，并允许输入纯文本密钥。在此示例中，*environment.secrets*文件的内容是：

```py
export DBPASS=MYPASS
```

保存*environment.secrets*文件后，请检查文件以确保其已加密，这样可以安全地添加到源代码控制中：

```py
$ cat environment.secrets
{
	"data": "ENC[AES256_GCM,data:qlQ5zc7e8KgGmu5goC9WmE7PP8gueBoSsmM=,
  iv:xG8BHcRfdfLpH9nUlTijBsYrh4TuSdvDqp5F+2Hqw4I=,
  tag:0OIVAm9O/UYGljGCzZerTQ==,type:str]",
	"sops": {
		"kms": null,
		"gcp_kms": null,
		"lastmodified": "2019-07-12T05:03:45Z",
		"mac": "ENC[AES256_GCM,data:wo+zPVbPbAJt9Nl23nYuWs55f68/DZJWj3pc0
    l8T2d/SbuRF6YCuOXHSHIKs1ZBpSlsjmIrPyYTqI+M4Wf7it7fnNS8b7FnclwmxJjptBWgL
    T/A1GzIKT1Vrgw9QgJ+prq+Qcrk5dPzhsOTxOoOhGRPsyN8KjkS4sGuXM=,iv:0VvSMgjF6
    ypcK+1J54fonRoI7c5whmcu3iNV8xLH02k=,
    tag:YaI7DXvvllvpJ3Talzl8lg==,
    type:str]",
		"pgp": [
			{
				"created_at": "2019-07-12T05:02:24Z",
				"enc": "-----BEGIN PGP MESSAGE-----\n\nhQEMA+3cyc
        g5b/Hu0OvU5ONr/F0htZM2MZQSXpxoCiO\nWGB5Czc8FTSlRSwu8/cOx0Ch1FwH+IdLwwL+jd
        oXVe55myuu/3OKUy7H1w/W2R\nPI99Biw1m5u3ir3+9tLXmRpLWkz7+nX7FThl9QnOS25
        NRUSSxS7hNaZMcYjpXW+w\nM3XeaGStgbJ9OgIp4A8YGigZQVZZFl3fAG3bm2c+TNJcAbl
        zDpc40fxlR+7LroJI\njuidzyOEe49k0pq3tzqCnph5wPr3HZ1JeQmsIquf//9D509S5xH
        Sa9lkz3Y7V4KC\nefzBiS8pivm55T0s+zPBPB/GWUVlqGaxRhv1TAU=\n=WA4+
        \n-----END PGP MESSAGE-----\n",
				"fp": "E14104A0890994B9AC9C9F6782C1FF5E679EFF32"
			}
		],
		"unencrypted_suffix": "_unencrypted",
		"version": "3.0.5"
	}
}%
```

要解密文件，请运行：

```py
$ sops -d environment.secrets
export DBPASS=MYPASS
```

###### 注意

在 Macintosh 上使用`sops`与`gpg`交互存在问题。在能够使用`sops`解密文件之前，您需要运行以下命令：

```py
$ GPG_TTY=$(tty)
$ export GPG_TTY
```

这里的目标是运行先前在`docker-compose.yaml`文件中定义的`migrations`服务。为了将`sops`密钥管理方法集成到`docker-compose`中，使用`sops -d`解密*environments.secrets*文件，将其内容源化到当前 shell 中，然后使用一个命令调用`docker-compose up -d migrations`，该命令不会将密钥暴露给 shell 历史记录：

```py
$ source <(sops -d environment.secrets); docker-compose up -d migrations
postgres is up-to-date
Recreating flask-by-example_migrations_1 ... done
```

通过检查数据库并验证是否创建了两个表`alembic_version`和`results`来验证迁移是否成功运行：

```py
$ docker-compose exec db psql -U postgres wordcount
psql (11.4 (Debian 11.4-1.pgdg90+1))
Type "help" for help.

wordcount=# \dt
```

```py
                  List of relations
 Schema |      Name       | Type  |       Owner
--------+-----------------+-------+-------------------
 public | alembic_version | table | wordcount_dbadmin
 public | results         | table | wordcount_dbadmin
(2 rows)

wordcount=# \q
```

[第四部分](https://oreil.ly/UY2yw) 在“Flask 实例”教程中是部署一个基于 Python RQ 的 Python 工作进程，该进程与 Redis 实例通信。

首先，需要运行 Redis。将其作为名为`redis`的服务添加到*docker_compose.yaml*文件中，并确保其内部端口 6379 映射到本地操作系统的端口 6379：

```py
  redis:
    image: "redis:alpine"
    ports:
      - "6379:6379"
```

通过将其作为参数指定给 `docker-compose up -d` 单独启动 `redis` 服务：

```py
$ docker-compose up -d redis
Starting flask-by-example_redis_1 ... done
```

运行 `docker ps` 查看基于 `redis:alpine` 镜像运行的新 Docker 容器：

```py
$ docker ps
CONTAINER ID   IMAGE   COMMAND    CREATED   STATUS   PORTS   NAMES
a1555cc372d6   redis:alpine "docker-entrypoint.s…" 3 seconds ago  Up 1 second
0.0.0.0:6379->6379/tcp   flask-by-example_redis_1
83b54ab10099   postgres:11  "docker-entrypoint.s…" 22 hours ago   Up 16 hours
0.0.0.0:5432->5432/tcp   postgres
```

使用 `docker-compose logs` 命令检查 `redis` 服务的日志：

```py
$ docker-compose logs redis
Attaching to flask-by-example_redis_1
1:C 12 Jul 2019 20:17:12.966 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 12 Jul 2019 20:17:12.966 # Redis version=5.0.5, bits=64, commit=00000000,
modified=0, pid=1, just started
1:C 12 Jul 2019 20:17:12.966 # Warning: no config file specified, using the
default config. In order to specify a config file use
redis-server /path/to/redis.conf
1:M 12 Jul 2019 20:17:12.967 * Running mode=standalone, port=6379.
1:M 12 Jul 2019 20:17:12.967 # WARNING: The TCP backlog setting of 511 cannot
be enforced because /proc/sys/net/core/somaxconn
is set to the lower value of 128.
1:M 12 Jul 2019 20:17:12.967 # Server initialized
1:M 12 Jul 2019 20:17:12.967 * Ready to accept connections
```

下一步是在 *docker-compose.yaml* 中为 Python RQ 工作进程创建一个名为 `worker` 的服务：

```py
  worker:
    image: "flask-by-example:v1"
    command: "worker.py"
    environment:
      APP_SETTINGS: config.ProductionConfig
      DATABASE_URL: postgresql://wordcount_dbadmin:$DBPASS@db/wordcount
      REDISTOGO_URL: redis://redis:6379
    depends_on:
      - db
      - redis
```

运行工作服务，就像`redis`服务一样，使用`docker-compose up -d`：

```py
$ docker-compose up -d worker
flask-by-example_redis_1 is up-to-date
Starting flask-by-example_worker_1 ... done
```

运行 `docker ps` 将显示工作容器：

```py
$ docker ps
CONTAINER ID   IMAGE   COMMAND    CREATED   STATUS   PORTS   NAMES
72327ab33073  flask-by-example "python worker.py"     8 minutes ago
Up 14 seconds                             flask-by-example_worker_1
b11b03a5bcc3  redis:alpine     "docker-entrypoint.s…" 15 minutes ago
Up About a minute  0.0.0.0:6379->6379/tc  flask-by-example_redis_1
83b54ab10099  postgres:11      "docker-entrypoint.s…"  23 hours ago
Up 17 hours        0.0.0.0:5432->5432/tcp postgres
```

使用`docker-compose logs`查看工作容器日志：

```py
$ docker-compose logs worker
Attaching to flask-by-example_worker_1
20:46:34 RQ worker 'rq:worker:a66ca38275a14cac86c9b353e946a72e' started,
version 1.0
20:46:34 *** Listening on default...
20:46:34 Cleaning registries for queue: default
```

现在在自己的容器中启动主 Flask 应用程序。在 *docker-compose.yaml* 中创建一个名为 `app` 的新服务：

```py
  app:
    image: "flask-by-example:v1"
    command: "manage.py runserver --host=0.0.0.0"
    ports:
      - "5000:5000"
    environment:
      APP_SETTINGS: config.ProductionConfig
      DATABASE_URL: postgresql://wordcount_dbadmin:$DBPASS@db/wordcount
      REDISTOGO_URL: redis://redis:6379
    depends_on:
      - db
      - redis
```

将应用程序容器中的端口 5000（Flask 应用程序的默认端口）映射到本地机器的端口 5000。在应用程序容器中传递命令 `manage.py runserver --host=0.0.0.0`，以确保 Flask 应用程序在容器内正确地暴露端口 5000。

使用 `docker compose up -d` 启动 `app` 服务，同时在包含 `DBPASS` 的加密文件上运行 `sops -d`，然后在调用 `docker-compose` 之前源化解密文件：

```py
source <(sops -d environment.secrets); docker-compose up -d app
postgres is up-to-date
Recreating flask-by-example_app_1 ... done
```

注意到新的 Docker 容器正在运行应用程序，列表由 `docker ps` 返回：

```py
$ docker ps
CONTAINER ID   IMAGE   COMMAND    CREATED   STATUS   PORTS   NAMES
d99168a152f1   flask-by-example "python app.py"  3 seconds ago
Up 2 seconds    0.0.0.0:5000->5000/tcp   flask-by-example_app_1
72327ab33073   flask-by-example "python worker.py" 16 minutes ago
Up 7 minutes                             flask-by-example_worker_1
b11b03a5bcc3   redis:alpine     "docker-entrypoint.s…" 23 minutes ago
Up 9 minutes    0.0.0.0:6379->6379/tcp   flask-by-example_redis_1
83b54ab10099   postgres:11      "docker-entrypoint.s…"  23 hours ago
Up 17 hours     0.0.0.0:5432->5432/tcp   postgres
```

使用 `docker-compose logs` 检查应用程序容器的日志：

```py
$ docker-compose logs app
Attaching to flask-by-example_app_1
app_1         |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
```

运行 `docker-compose logs` 而不带其他参数允许我们检查 *docker-compose.yaml* 文件中定义的所有服务的日志：

```py
$ docker-compose logs
Attaching to flask-by-example_app_1,
flask-by-example_worker_1,
flask-by-example_migrations_1,
flask-by-example_redis_1,
postgres
1:C 12 Jul 2019 20:17:12.966 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 12 Jul 2019 20:17:12.966 # Redis version=5.0.5, bits=64, commit=00000000,
modified=0, pid=1, just started
1:C 12 Jul 2019 20:17:12.966 # Warning: no config file specified, using the
default config. In order to specify a config file use
redis-server /path/to/redis.conf
1:M 12 Jul 2019 20:17:12.967 * Running mode=standalone, port=6379.
1:M 12 Jul 2019 20:17:12.967 # WARNING: The TCP backlog setting of 511 cannot
be enforced because /proc/sys/net/core/somaxconn
is set to the lower value of 128.
1:M 12 Jul 2019 20:17:12.967 # Server initialized
1:M 12 Jul 2019 20:17:12.967 * Ready to accept connections
app_1         |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
postgres      | 2019-07-12 22:15:19.193 UTC [1]
LOG:  listening on IPv4 address "0.0.0.0", port 5432
postgres      | 2019-07-12 22:15:19.194 UTC [1]
LOG:  listening on IPv6 address "::", port 5432
postgres      | 2019-07-12 22:15:19.199 UTC [1]
LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
postgres      | 2019-07-12 22:15:19.214 UTC [22]
LOG:  database system was shut down at 2019-07-12 22:15:09 UTC
postgres      | 2019-07-12 22:15:19.225 UTC [1]
LOG:  database system is ready to accept connections
migrations_1  | INFO [alembic.runtime.migration] Context impl PostgresqlImpl.
migrations_1  | INFO [alembic.runtime.migration] Will assume transactional DDL.
worker_1      | 22:15:20
RQ worker 'rq:worker:2edb6a54f30a4aae8a8ca2f4a9850303' started, version 1.0
worker_1      | 22:15:20 *** Listening on default...
worker_1      | 22:15:20 Cleaning registries for queue: default
```

最后一步是测试应用程序。访问 http://127.0.0.1:5000 并在 URL 字段中输入 `python.org`。此时，应用程序向工作进程发送一个作业，要求其对 `python.org` 的主页执行函数 `count_and_save_words`。应用程序定期轮询作业以获取结果，完成后在主页上显示单词频率。

为了使 *docker-compose.yaml* 文件更具可移植性，将 `flask-by-example` Docker 镜像推送到 Docker Hub，并在 `app` 和 `worker` 服务的容器部分引用 Docker Hub 镜像。

使用 Docker Hub 用户名前缀为现有的本地 Docker 镜像 `flask-by-example:v1` 打标签，然后将新标记的镜像推送到 Docker Hub：

```py
$ docker tag flask-by-example:v1 griggheo/flask-by-example:v1
$ docker push griggheo/flask-by-example:v1
```

修改 *docker-compose.yaml* 以引用新的 Docker Hub 镜像。以下是 *docker-compose.yaml* 的最终版本：

```py
$ cat docker-compose.yaml
version: "3"
services:
  app:
    image: "griggheo/flask-by-example:v1"
    command: "manage.py runserver --host=0.0.0.0"
    ports:
      - "5000:5000"
    environment:
      APP_SETTINGS: config.ProductionConfig
      DATABASE_URL: postgresql://wordcount_dbadmin:$DBPASS@db/wordcount
      REDISTOGO_URL: redis://redis:6379
    depends_on:
      - db
      - redis
  worker:
    image: "griggheo/flask-by-example:v1"
    command: "worker.py"
    environment:
      APP_SETTINGS: config.ProductionConfig
      DATABASE_URL: postgresql://wordcount_dbadmin:$DBPASS@db/wordcount
      REDISTOGO_URL: redis://redis:6379
    depends_on:
      - db
      - redis
  migrations:
    image: "griggheo/flask-by-example:v1"
    command: "manage.py db upgrade"
    environment:
      APP_SETTINGS: config.ProductionConfig
      DATABASE_URL: postgresql://wordcount_dbadmin:$DBPASS@db/wordcount
    depends_on:
      - db
  db:
    image: "postgres:11"
    container_name: "postgres"
    ports:
      - "5432:5432"
    volumes:
      - dbdata:/var/lib/postgresql/data
  redis:
    image: "redis:alpine"
    ports:
      - "6379:6379"
volumes:
  dbdata:
```

要重新启动本地 Docker 容器，请运行 `docker-compose down`，然后跟着 `docker-compose up -d`：

```py
$ docker-compose down
Stopping flask-by-example_worker_1 ... done
Stopping flask-by-example_app_1    ... done
Stopping flask-by-example_redis_1  ... done
Stopping postgres                  ... done
Removing flask-by-example_worker_1     ... done
Removing flask-by-example_app_1        ... done
Removing flask-by-example_migrations_1 ... done
Removing flask-by-example_redis_1      ... done
Removing postgres                      ... done
Removing network flask-by-example_default

$ source <(sops -d environment.secrets); docker-compose up -d
Creating network "flask-by-example_default" with the default driver
Creating flask-by-example_redis_1      ... done
Creating postgres                 ... done
Creating flask-by-example_migrations_1 ... done
Creating flask-by-example_worker_1     ... done
Creating flask-by-example_app_1        ... done
```

注意使用 `docker-compose` 轻松启动和关闭一组 Docker 容器。

###### 提示

即使您只想运行单个 Docker 容器，将其包含在 *docker-compose.yaml* 文件中并使用 `docker-compose up -d` 命令启动它仍然是个好主意。当您想要添加第二个容器时，这将使您的生活更加轻松，并且还将作为基础设施即代码的一个小例子，*docker-compose.yaml* 文件反映了您的应用程序的本地 Docker 设置状态。

# 将 docker-compose 服务迁移到新主机和操作系统

现在我们将展示如何将前一节的 `docker-compose` 设置迁移到运行 Ubuntu 18.04 的服务器。

启动运行 Ubuntu 18.04 的 Amazon EC2 实例并安装 `docker-engine` 和 `docker-compose`：

```py
$ sudo apt-get update
$ sudo apt-get remove docker docker-engine docker.io containerd runc
$ sudo apt-get install \
  apt-transport-https \
  ca-certificates \
  curl \
  gnupg-agent \
  software-properties-common
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
$ sudo usermod -a -G docker ubuntu

# download docker-compose
$ sudo curl -L \
"https://github.com/docker/compose/releases/download/1.24.1/docker-compose-\
$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
```

将 *docker-compose.yaml* 文件复制到远程 EC2 实例并首先启动 `db` 服务，以便可以创建应用程序使用的数据库：

```py
$ docker-compose up -d db
Starting postgres ...
Starting postgres ... done

$ docker ps
CONTAINER ID   IMAGE   COMMAND    CREATED   STATUS   PORTS   NAMES
49fe88efdb45 postgres:11 "docker-entrypoint.s…" 29 seconds ago
  Up 3 seconds        0.0.0.0:5432->5432/tcp   postgres
```

使用 `docker exec` 在正在运行的 Docker 容器中运行 `psql -U postgres` 命令访问 PostgreSQL 数据库。在 PostgreSQL 提示符下，创建 `wordcount` 数据库和 `wordcount_dbadmin` 角色：

```py
$ docker-compose exec db psql -U postgres
psql (11.4 (Debian 11.4-1.pgdg90+1))
Type "help" for help.

postgres=# create database wordcount;
CREATE DATABASE
postgres=# \q

$ docker exec -it 49fe88efdb45 psql -U postgres wordcount
psql (11.4 (Debian 11.4-1.pgdg90+1))
Type "help" for help.

wordcount=# CREATE ROLE wordcount_dbadmin;
CREATE ROLE
wordcount=# ALTER ROLE wordcount_dbadmin LOGIN;
ALTER ROLE
wordcount=# ALTER USER wordcount_dbadmin PASSWORD 'MYPASS';
ALTER ROLE
wordcount=# \q
```

在启动在 *docker-compose.yaml* 中定义的服务的容器之前，有两件事是必需的：

1.  运行 `docker login` 以能够拉取之前推送到 Docker Hub 的 Docker 镜像：

    ```py
    $ docker login
    ```

1.  在当前 Shell 中设置`DBPASS`环境变量的正确值。在本地 macOS 设置中描述的`sops`方法可用，但是在本示例中，直接在 Shell 中设置：

    ```py
    $ export DOCKER_PASS=MYPASS
    ```

现在通过运行 `docker-compose up -d` 启动应用程序所需的所有服务：

```py
$ docker-compose up -d
Pulling worker (griggheo/flask-by-example:v1)...
v1: Pulling from griggheo/flask-by-example
921b31ab772b: Already exists
1a0c422ed526: Already exists
ec0818a7bbe4: Already exists
b53197ee35ff: Already exists
8b25717b4dbf: Already exists
9be5e85cacbb: Pull complete
bd62f980b08d: Pull complete
9a89f908ad0a: Pull complete
d787e00a01aa: Pull complete
Digest: sha256:4fc554da6157b394b4a012943b649ec66c999b2acccb839562e89e34b7180e3e
Status: Downloaded newer image for griggheo/flask-by-example:v1
Creating fbe_redis_1      ... done
Creating postgres    ... done
Creating fbe_migrations_1 ... done
Creating fbe_app_1        ... done
Creating fbe_worker_1     ... done

$ docker ps
CONTAINER ID   IMAGE   COMMAND    CREATED   STATUS   PORTS   NAMES
f65fe9631d44  griggheo/flask-by-example:v1 "python3 manage.py r…" 5 seconds ago
Up 2 seconds        0.0.0.0:5000->5000/tcp   fbe_app_1
71fc0b24bce3  griggheo/flask-by-example:v1 "python3 worker.py"    5 seconds ago
Up 2 seconds                                 fbe_worker_1
a66d75a20a2d  redis:alpine     "docker-entrypoint.s…"   7 seconds ago
Up 5 seconds        0.0.0.0:6379->6379/tcp   fbe_redis_1
56ff97067637  postgres:11      "docker-entrypoint.s…"   7 seconds ago
Up 5 seconds        0.0.0.0:5432->5432/tcp   postgres
```

此时，在允许 AWS 安全组中与我们的 Ubuntu EC2 实例关联的端口 5000 的访问后，您可以在该实例的外部 IP 上的 5000 端口访问并使用该应用程序。

再次强调一下 Docker 简化应用部署的重要性。Docker 容器和镜像的可移植性意味着您可以在任何安装了 Docker 引擎的操作系统上运行您的应用程序。在这里展示的示例中，在 Ubuntu 服务器上不需要安装任何先决条件：不需要 Flask，不需要 PostgreSQL，也不需要 Redis。也不需要将应用程序代码从本地开发机器复制到 Ubuntu 服务器上。在 Ubuntu 服务器上唯一需要的文件是 *docker-compose.yaml*。然后，只需一条命令就可以启动应用程序的整套服务：

`$ docker-compose up -d`

###### 提示

警惕从公共 Docker 仓库下载和使用 Docker 镜像，因为其中许多镜像存在严重的安全漏洞，其中最严重的可以允许攻击者突破 Docker 容器的隔离性并接管主机操作系统。一个良好的实践是从一个受信任的、预构建的镜像开始，或者从头开始构建你自己的镜像。随时关注最新的安全补丁和软件更新，并在这些补丁或更新可用时重新构建你的镜像。另一个良好的实践是使用众多可用的 Docker 扫描工具之一（其中包括 [Clair](https://oreil.ly/OBkkx)、[Anchore](https://oreil.ly/uRI_1) 和 [Falco](https://oreil.ly/QXRg6)）扫描所有的 Docker 镜像。这样的扫描可以作为持续集成/持续部署流水线的一部分进行，通常在构建 Docker 镜像时执行。

尽管 `docker-compose` 可以轻松地运行多个容器化服务作为同一应用的一部分，但它只适用于单台机器，这在生产环境中的实用性受到限制。如果你不担心停机时间并愿意在单台机器上运行所有内容，那么只能认为使用 `docker-compose` 部署的应用程序是“生产就绪”的（尽管如此，格里格看到一些托管提供者使用 `docker-compose` 在生产环境中运行 Docker 化应用程序）。对于真正的“生产就绪”场景，你需要一个像 Kubernetes 这样的容器编排引擎，这将在下一章讨论。

# 练习

+   熟悉 [Dockerfile 参考](https://oreil.ly/kA8ZF)。

+   熟悉 [Docker Compose 配置参考](https://oreil.ly/ENMsQ)。

+   创建一个 AWS KMS 密钥，并在 `sops` 中使用它，而不是本地的 `PGP` 密钥。这允许你将 AWS IAM 权限应用到密钥上，并将对密钥的访问限制为仅需要的开发人员。

+   编写一个 shell 脚本，使用 `docker exec` 或 `docker-compose exec` 来运行 PostgreSQL 命令，创建数据库和角色。

+   尝试其他容器技术，例如 [Podman](https://podman.io)。

¹ 这是一个示例 URL 地址——你的 IP 地址将会不同。

² 你的 IP 地址将会不同。
