```
brew cask install docker
```

http://localhost/tutorial/

```
docker build -t getting-started .
	-t  起名
docker run -dp 3000:3000 getting-started
	-d 后台运行
	-p 端口映射
```

### Removing a container using the CLI[¶](http://localhost/tutorial/updating-our-app/#removing-a-container-using-the-cli)

```
1. docker ps
2. docker stop <the-container-id>
3. docker rm <the-container-id>
或者 2-3连起来
docker rm -f <the-container-id>
```

[docker Hub](https://hub.docker.com/r/nibnait/getting-started)

```
docker push nibnait/getting-started:tagname

docker image ls
docker login -u nibnait
docker tag getting-started nibnait/getting-started
docker push nibnait/getting-started
```

**[Play with Docker](https://labs.play-with-docker.com/)**

