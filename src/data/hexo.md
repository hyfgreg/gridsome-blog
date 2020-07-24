---
title: 折腾hexo小记，hexo+next+七牛+python
date: 2019-07-06 15:17:03
categories:
- 小记
tags:
- hexo
- next
- 七牛
- python
---

## 在所有东西之前
想着重启博客，在调研了一番后，决定选用hexo作为博客框架，放弃了自建前后端方案以及使用过的ghost框架，原因大概有以下这些

1. hexo使用较为便捷，可以集中主要精力在记录内容上而不是“写”博客架构本身
2. hexo的主题确实比较多，并且我本人比较喜欢NexT主题，调研了一圈，ghost上没有比较完整的移植
3. 我比较懒

总结：hexo方便、好看，就选它了

## 用到的工具
1. 框架: hexo
2. 主题: NexT.Mist
3. 对象存储: 七牛。将所有静态文件存在七牛，并且通过域名绑定的形式访问
4. 上传工具: qshell，七牛官方出品的命令行工具
5. python(未来可能会替换为nodejs)，用于生成需要刷新CDN的文件的脚本

## 步骤
### 安装NodeJS，[官网](http://nodejs.cn/)
### 安装hexo
```bash
> npm install hexo-cli -g
> hexo init blog
> cd blog
> npm install
> hexo server
```
### 安装NexT

```bash
> cd blog
> git clone https://github.com/theme-next/hexo-theme-next themes/next
```

修改blog的默认主题
```yml
# blog/_config.yml
theme: next
```

修改NexT的scheme为Mist
```yml
# blog/themes/next/_config.yml
scheme: Mist
```

此时，可以用hexo server命令在本地跑一个服务，在浏览器中预览博客的效果

## 七牛

### 准备工作
1. 注册、实名认证、开通对象存储、融合CDN
2. 在*融合CDN/域名管理/添加域名*下进行域名绑定，我一路默认设置。绑定好后会得到一个域名的CNAME，记下来
3. 由于我是在阿里云注册域名，所以需要到阿里云加入一条解析到七牛，在记录类型内选择CNAME，记录值就是第2步的CNAME，我的主机记录选择了@，那么不用输入www就可以访问了
4. 在七牛的*对象存储/空间设置*下开启默认首页设置，这样访问域名时会直接将空间内的index.html作为默认首页展示

### 使用qshell进行文件的上传
1. 安装, [官网](https://developer.qiniu.com/kodo/tools/1302/qshell), windows下建议在*环境变量/系统变量/path*下加入qshell的路径，这样在powershell内可以直接调用qshell命令
2. 使用, [官网](https://developer.qiniu.com/kodo/tools/1302/qshell), 主要是获取ak/sk值，并且指定bucket
3. 上传脚本，在blog文件夹下创建upload.conf文件, 我的文件内容如下。在本地写完博客后，调用hexo generate，会创建所有静态文件并且存储在blog/public文件夹下，qshell将这个文件夹下的所有文件上传至你的七牛
```json
{
  "src_dir": "D:\\Projects\\hexo\\blog\\public",
  "bucket": "blog",
  "overwrite" : true,
  "rescan_local" : true
}
```
### 使用qshell更新CDN缓存
1. 更新博客后，静态文件的名称其实保持不变，所以上传七牛后，由于CDN缓存的原因，导致无法在线看到更新，需要对CDN缓存进行更新
2. 在blog文件下创建一个新的txt文件，例如blog_file.txt，在里面填写需要更新的文件链接, 例如
```
http://yoursite/index.html
http://yoursite/
```
3. 命令
```bash
> qshell cdnrefresh -i blog_file.txt
```
4. 粗暴的做法，直接更新整个文件夹，创建一个新的txt，例如blog_dir.txt，内容如下
    ```
    http://yoursite/
    ```
    ```bash
    > qshell cdnrefresh --dirs -i blog_dir.txt
    ```
    你的域名下的所有文件的缓存都会被更新
    > 注意: 对于免费用户，七牛提供每日500个文件更新，10个文件夹更新

## 使用python生成blog_file.txt
由于七牛对于CDN更新的文件数量有限制，因此在更新博客后，尽量只更新那些实际内容有变动的静态文件的CDN缓存。我的策略很粗暴，假设我只是写了新的博客，那么我只要更新和新文章有关的.html文件即可，而我直接简单粗暴的更新所有的.html文件
```python
import os

url = "http://yoursite/"  # 更换成你的域名
files = []

for root, dirs, _files in os.walk(r"public"):
    for file in _files:
        # 获取文件所属目录
        # print(root)
        # 获取文件路径
        if "index.html" in file:
            tmp = os.path.join(root, file).replace(
                '\\', '/').replace('public', url)
            files.append(tmp)
            files.append(tmp.replace('index.html', ''))

with open('blog_file.txt', 'w') as f:
    f.truncate()
    print('清除原文件内容')
    for file in files:
        print('发现文件{}'.format(file))
        f.write(file)
        f.write('\n')
```
## 一键命令
在写完一篇博客后，需要做以下操作
1. 生成新的静态文件
   ```bash
   > hexo generate
   ```
2. 七牛云上传
   ```bash
   > qshell qupload upload.conf
   ```
3. 更新blog_file.txt
   ```bash
   > python generate_blog_file.py
   ```
4. 更新CDN缓存
   ```bash
   > qshell cdnrefresh -i blog_file.txt
   ```
    我们可以在blog/package.json文件内加一条命令，一键搞定所有操作
    ```json
    "scripts": {
        "build": "hexo generate && qshell qupload upload.conf && python generate_blog_file.py && qshell cdnrefresh -i blog_file.txt"
    }
    ```
    命令行输入以下命令
    ```bash
    > npm run build
    ```
## 总结
以上就是折腾的主要过程，由于hexo是基于NodeJS框架，因此node是必须安装的，所以在未来考虑使用NodeJS来生成blog_file.txt，由于我比较懒，暂时就不写了

这一次重启博客，目的是对生活、学习有一个记录，希望能够借着这个过程来巩固自己对于各种知识的认知。希望自己能够播下一个行动，收获一个习惯吧！

2019/7/6 16:26 上海