# 工具

## Charles

* 抓包工具，可抓取微信小程序中的高清无水印视频
  * Charles是一个HTTP代理服务器,HTTP监视器,反转代理服务器，当浏览器连接Charles的代理访问互联网时，Charles可以监控浏览器发送和接收的所有数据。它允许一个开发者查看所有连接互联网的HTTP通信，这些包括request, response和HTTP headers （包含cookies与caching信息）
* 安装好后，打开Charles，在菜单中打开代理Porxy-macOS Proxy，Charles就在本地直接起了一个代理服务，默认端口号是8888
* 打开命令行，输入ifconfig(win下输入ipconfig)，查看本机的局域网ip地址
* 将手机和电脑连入到统一局域网内，然后设置wifi连接的高级设置，将刚刚的本机ip地址和端口号填入到代理设置内
* 这时，手机端的一切网络请求就都可以在电脑端的Charles界面中展示出来了
* wget命令通过视频链接来下载视频到本地
* 当不抓包的时候，会将Charles关闭，这时候手机是访问不到网络的，因为设置了代理(就是Charles)，这时候需要将手机中的代理关闭

# 命令

* `zip/unzip`

  ```shell
  zip -q -r -e -m -o myfile.zip someThing
  # -q 表示不显示压缩进度状态
  # -r 表示子目录子文件全部压缩为zip
  # -e 表示你的压缩文件需要加密，终端会提示你输入密码的，zip -r -P Password01! modudu.zip SomeDir
  # -m 压缩完删除原文件
  # -0 设置所有被压缩文件的最后修改时间为当前压缩时间
  # zip -q -r -e -m -o '\user\someone\someDir\someFile.zip' '\users\someDir'
  ```

  ```shell
  unzip [选项] 压缩文件名.zip
  # -x 文件列表 解压缩文件，但不包括指定的file文件
  # -v 查看压缩文件目录，但不解压
  # -t 测试文件有无损坏，但不解压
  # -d 目录 把压缩文件解到指定目录下
  # -z 只显示压缩文件的注解
  # -n 不覆盖已经存在的文件
  # -o 覆盖已存在的文件且不要求用户确认
  # -j 不重建文档的目录结构，把所有文件解压到同一目录下
  # 将压缩文件text.zip在当前目录下解压缩
  unzip text.zip
  # 将压缩文件text.zip在指定目录/tmp下解压缩，如果已有相同的文件存在，要求unzip命令不覆盖原先的文件
  unzip -n text.zip -d /tmp 
  # 查看压缩文件目录，但不解压
  unzip -v text.zip
  ```

  