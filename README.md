# Lantern
Rebuild the source code

蓝灯作为一个科学上网工具还是不错的，特别是可以免费试用，不过每个月有800MB流量的限制，对于大多数不看视频的人来说够用了。偶尔看到ilanyu的博客《lantern编译过程》里提到，从源码编译的没有流量限制，于是便想试一试。博客里面讲到了如何编译Linux和Windows版的，但正好我用的是Mac...，只能自己动手，丰衣足食了。

ilanyu是通过docker镜像来编译的，这样可以省去搭建环境的麻烦。但是我试了一下，镜像里面应该没有安装编译Mac版依赖的一些东西（这些依赖写在Makefile里面），然后我在容器里面去安装的时候，一直下载不下来，所以我就索性直接在自己本地编译了。所以主要是一些依赖环境的搭建，下面记录一下整个过程。

步骤1. 下载蓝灯代码。
```
git clone https://github.com/getlantern/lantern.git
```

步骤2. 修改Makefile。这里主要修改一下版本和时间，不然可能到时候会强制升级，升级后又会限制流量了。在Makefile里面找到GIT_REVISION和GIT_REVISION_DATE两个变量，修改的大一些。比如我的修改如下：

```
GIT_REVISION := 9.9.9
GIT_REVISION_DATE := 2020-11-15 00:54:06 -0800
```

步骤3. 安装一些依赖的包。

```
brew install go
brew install node
npm config set registry https://registry.npm.taobao.org
sudo npm install -g appdmg
sudo npm install -g svgexport
sudo npm install -g gulp-cli
```
如果还提示依赖其他的，请自行安装。

步骤4. 蓝灯的是用golang写的，但是不知道为啥，自己fork了一份golang。所以如果用官方的go编译会编译不通过。因为有个变量在官方的go标准库里面没有。所以需要稍微修改一下代码。修改src/github.com/getlantern/flashlight/proxied/proxied.go和src/github.com/getlantern/flashlight/client/reverseproxy.go，将里面的MaxIdleTime改为IdleConnTimeout，并且注释掉transport.EnforceMaxIdleTime()和tr.EnforceMaxIdleTime()。

步骤5. 执行以下命令编译即可：

```
VERSION=9.9.9
export VERSION
make package-darwin
```
然后可能出现下面这个错误：
```
➜  lantern git:(devel) ✗ make package-darwin
Building darwin/amd64...
Build tags:
Extra ldflags: -X github.com/getlantern/flashlight.compileTimePackageVersion=9.9.9
Generating distribution package for darwin/amd64 manoto...
Developer ID Application: Brave New Software Project, Inc: no identity found
make: *** [package-darwin-manoto] Error 1
```
如果看到这个，不用管，这是因为在生成dmg文件的时候因为签名的问题而失败了。但是这个时候蓝灯文件（Lantern.app）已经编译好了，只是没有打包成dmg文件而已。
当然，大家还是尽量支持正版...本文只是探索一下编译方法。
