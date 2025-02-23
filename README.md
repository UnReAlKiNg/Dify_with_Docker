# Dify_with_Docker
记录使用Docker Desktop本地化部署dify的各种坑。

## 工作环境
我的设备配置如下：  
1、Windows 11，i5-12500，24GB RAM(后续会同步更新 M1 Macbook Pro 16GB+1T上的配置过程)  
2、下载并安装 Docker Desktop 4.38.0  
3、根据官网教程，使用
‘’‘ bash
git clone https://github.com/langgenius/dify.git
’‘’
下载相关内容  

## 部署阶段（self-hosted）
1、进入 <cd dify/docker>  
2、复制环境配置文件 <cp .env.example .env>  
3、启动docker容器 <docker compose up -d>。这一步大概率是会报什么"... manifest ..."的错误。问题主要出在docker-compose.yaml中的版本名称和实际你在docker里能下载到的版本不一致。这时候打开docker desktop，在搜索框输入<langgenius/dify>，找到dify-web和dify-api对应的tag（20250218时的结果为->langgenius/dify-web:main，langgenius/dify-api:ed7851a4b3....），用VS Code打开docker-compose.yaml，搜索<1.0.0>，将web:和api:后的值替换为刚刚搜索到的值（或者可以统一改成:main），保存。再次运行<docker compose -f docker-compose.yaml up -d>就会正常开始拉取镜像并运行，完成后可在docker内查看该容器。  

## 配置阶段
首先先根据向导设置管理员账号密码并登陆，这一部份不会有什么问题。进来后，在<设置-模型提供商>里添加模型时会碰到一个坑，即模型添加不上去。我遇到的情况如下：在模型提供商里选择ollama，点击安装，在docker-plugin_deamon-1的输出里可以看到在自动下载相关的python library，最后会卡在<[ERROR]init environment failed: failed to install dependencies: signal: killed, ...>，然后就进入无限的”下载，安装，安装失败，重试”循环。这里可以参考官方repo下的解决方案（dify-issue-#12635-kurokobo），在.env中添加<PLUGIN_WORKING_PATH=/app/cwd>，然后重新运行<docker compose up -d>，即可正常添加。如果模型需要开放给LAN用户，需要在用户环境变量里添加<变量=OLLAMA_HOST，值=0.0.0.0>，并重启ollama，dify中添加模型时，模型的url需要填写为<http://局域网ip地址:11434>
