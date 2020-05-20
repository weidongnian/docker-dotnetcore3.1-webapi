create app by use dotnet core 3.1 webapi and docker for macbook
使用core3.1 webapi +docker 创建的一个小例子

1，创建一个docker工作目录

    mkdir docker
    cd docker

2，创建core webapi 记得下载dotnet core sdk 3.1

    dotnet new webapi -o myMicroservice --no-https
    cd myMicroservice

3，修改一下WeatherForecastController 设置默认主页

        [Route("/")]
        [Route("/index")]
        [HttpGet]
        public IEnumerable<WeatherForecast> Get()
        {
            var rng = new Random();
            return Enumerable.Range(1, 5).Select(index => new WeatherForecast
            {
                Date = DateTime.Now.AddDays(index),
                TemperatureC = rng.Next(-20, 55),
                Summary = Summaries[rng.Next(Summaries.Length)]
            })
            .ToArray();
        }

4，运行,并访问http://localhost:5000

    dotnet run

5，确保mac下安装了docker,下载地址：https://hub.docker.com/editions/community/docker-ce-desktop-mac

    docker info

6，进入项目目录里 

    cd myMicroservice

7，创建DockerFile文件

    FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS build
    WORKDIR /src
    COPY myMicroservice.csproj .
    RUN dotnet restore
    COPY . .
    RUN dotnet publish -c release -o /app

    FROM mcr.microsoft.com/dotnet/core/aspnet:3.1
    WORKDIR /app
    COPY --from=build /app .
    ENTRYPOINT ["dotnet", "myMicroservice.dll"]

8，创建docker过滤文件.dockerignore 过滤掉bin和obj目录

    Dockerfile
    [b|B]in
    [O|o]bj

9，生成docker 镜像 -t：镜像名称，.:当前目录下找dockerfile文件

    docker build -t mymicroservice .

10，查看本地所有镜像，如果没有问题一个发现一个名称叫：mymicroservice的镜像

    docker iamges

11，运行镜像,临时生成一个叫mymicroservicecontainer的镜像实例

    docker run -it --rm -p 3000:80 --name mymicroservicecontainer mymicroservice
    
    docker run -it --rm -p 3000:80 --name 服务名称 影像名称:Tag

12，查看正在运行的docker实例

    docker ps

13，在浏览器中查看，如果没有问题，大功告成

    http://localhost:3000/

14，docker的一些指令:https://www.cnblogs.com/duanxz/p/7905233.html

    Docker环境信息 — docker [info|version]
    容器生命周期管理 — docker [create|exec|run|start|stop|restart|kill|rm|pause|unpause]
    容器操作运维 — docker [ps|inspect|top|attach|wait|export|port|rename|stat]
    容器rootfs命令 — docker [commit|cp|diff]
    镜像仓库 — docker [login|pull|push|search]
    本地镜像管理 — docker [build|images|rmi|tag|save|import|load]
    容器资源管理 — docker [volume|network]
    系统日志信息 — docker [events|history|logs]

15，docker的结构图
    ![Image text](https://images2018.cnblogs.com/blog/285763/201711/285763-20171127173336972-172418048.png)

16，建议在解决方案根目录执行docker build

    docker build -t 影像名称 解决方案根目录/项目目录/.
