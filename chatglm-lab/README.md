## 基于阿里云的 PAI

## 基于我们已经训练好的模型制作镜像


#### 第一步：模型放到我们的镜像

```dockerfile
FROM registry.cn-beijing.aliyuncs.com/agi/pai-base:v1

# WORKDIR 指定工作目录
WORKDIR /home/

# 安装包
COPY chatglm2-6b chatglm2-6b
```

#### 第二步：构建基于模型的镜像并且指定镜像名字为：llms:v1
```
docker build -t llms:v1 . -f Dockerfile.llms
```

#### 第三步：把业务代码放进去镜像
```
FROM llms:v1

# WORKDIR 指定工作目录
WORKDIR /home/

# 业务代码
COPY openai-api.py openai-api.py
COPY requirements.txt requirements.txt

# 安装项目依赖的包
RUN pip install -r requirements.txt


# EXPOSE 暴露 80 端口
EXPOSE 80
EXPOSE 7000
```

#### 第四步：构建基于业务代码的镜像并且指定镜像名字为：agilab:v1
```
docker build -t agilab:v1 . -f Dockerfile
```

#### 第五步：把我们的镜像推送到阿里云

* 里面的对应的变量需要更改

```
## 登录阿里云 ACR
docker login --username=star*****@163.com registry.cn-beijing.aliyuncs.com

## 创建 tag
docker tag [ImageId] registry.cn-beijing.aliyuncs.com/agi/pai-base:[镜像版本号]

## push 镜像
docker push registry.cn-beijing.aliyuncs.com/agi/pai-base:[镜像版本号]
```


## 让 EAS 可以使用外网域名访问

#### 配置 NGINX 代理。

* 由于需要外网的域名需要 token 我们自己做一层透传

```
location /openai/ {
        proxy_pass http://v3.********.cn-beijing.pai-eas.aliyuncs.com/;
        proxy_set_header Authorization "MjU1ZjZiY****************A==";
        # 添加 CORS 头部
        if ($request_method = 'OPTIONS') {
            add_header 'Access-Control-Allow-Origin' *;
            #add_header 'Access-Control-Allow-Origin' 'https://chat.devagi.cn';
            add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS, PUT, DELETE';
            add_header 'Access-Control-Allow-Headers' 'Authorization, Content-Type, x-requested-with';  # 添加 x-requested-with
            add_header 'Access-Control-Max-Age' 1728000;
            add_header 'Content-Type' 'text/plain; charset=utf-8';
            add_header 'Content-Length' 0;
            return 204;
        }

    }

```


## 使用开源 webUI 测试我们的大模型接口

* https://github.com/Yidadaa/ChatGPT-Next-Web


