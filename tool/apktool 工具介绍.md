# apktool 工具介绍

apktool主要用于逆向apk文件。它可以将资源解码，并在修改后可以重新构建它们。它还可以执行一些自动化任务，例如构建apk。

## 功能

- 将资源解码成原来的形式（包括resources.arsc，class.dex，9.png和xml）
- 将解码的资源重新打包成apk/jar
- 组织和处理依赖于框架资源的APK
- Smali调试
- 执行自动化任务

## 安装方式

参考https://ibotpeaches.github.io/Apktool/install/


## 使用方式

```
usage: apktool
 -advance,--advanced   prints advance information.
 -version,--version    prints the version then exits
usage: apktool if|install-framework [options] <framework.apk>
 -p,--frame-path <dir>   Stores framework files into <dir>.
 -t,--tag <tag>          Tag frameworks using <tag>.
usage: apktool d[ecode] [options] <file_apk>
 -f,--force              Force delete destination directory.
 -o,--output <dir>       The name of folder that gets written. Default is apk.out
 -p,--frame-path <dir>   Uses framework files located in <dir>.
 -r,--no-res             Do not decode resources.
 -s,--no-src             Do not decode sources.
 -t,--frame-tag <tag>    Uses framework files tagged by <tag>.
usage: apktool b[uild] [options] <app_path>
 -f,--force-all          Skip changes detection and build all files.
 -o,--output <dir>       The name of apk that gets written. Default is dist/name.apk
 -p,--frame-path <dir>   Uses framework files located in <dir>.

```



反编译apk文件

```
apktool d test.apk
```



将反编译后的文件重新打包

```
apktool b test
```



