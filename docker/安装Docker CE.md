# Ubuntu下安装docker CE

1.

```
sudo apt-get update
```

2.

```
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
```

3.

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

4.

```
sudo apt-key fingerprint 0EBFCD88
```

5.

```
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

6.

```
sudo apt-get update
```

7.这一步就安装完成了

```
sudo apt-get install docker-ce
```

8.测试一下

```
sudo docker run hello-world
```



