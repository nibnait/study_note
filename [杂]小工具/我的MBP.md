# 我的mbp

[搜狗输入法](https://pinyin.sogou.com/mac/)
[百度网盘](https://pan.baidu.com/)
[Surge](http://nssurge.com/)

## 触发角

## App Store

- 万年历
- Manico

## 百度云

- [Spectacle+1.2](https://www.spectacleapp.com/)
- Alfred youdao
- [typora](https://www.typora.io/#download)
- [Snipaste](https://zh.snipaste.com/)
- AirServer
  
## 去官网下载最新版

### [XCode](https://developer.apple.com/download/more/?=xcode)

### [clean my mac](http://www.mycleanmymac.com/xiazai.html)

[Paragon NTFS for Mac](https://my.paragon-software.com/)

### [MarginNote](https://www.marginnote.com/chinese/home)

[Sourcetree](https://www.sourcetreeapp.com/)

### [Logitech Options](https://www.logitech.com.cn/zh-cn/product/options)

[visual paradigm 社区版](https://www.visual-paradigm.com/cn/download/community.jsp)

[ezip](https://ezip.awehunt.com/)

[Visual VM](https://visualvm.github.io/download.html)

### [Sublime Text 3](https://www.sublimetext.com/)

   ```
   ctrl + `
   
   获取package control最新安装代码：[https://packagecontrol.io/installation](https://packagecontrol.io/installation)
   
   cmd+shift+p
   package control: install package
   cmd+shift+p
   pretty json
   
   Preferences —> Key Bindings：
   [
    { "keys": ["super+i"], "command": "copy_path" }
   ]
   ```

### [idea](https://www.jetbrains.com/idea/)

历史版本：https://www.jetbrains.com/idea/download/other.html

```
Fully Expand Tree Node  alt+鼠标滚轮下
collapse tree node   alt+鼠标滚轮上
```



Code Templates：

   ```
import org.junit.Test;

/*

Created by ${USER} on ${DATE}
*/
public class ${NAME} {

    @Test
    public void testCase() {

    }


}
   ```

   JDK 1.8：http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
   JDK 1.7：http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html
   JDK 1.6：http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase6-419409.html
JDK 1.5：http://www.oracle.com/technetwork/java/javasebusiness/downloads/java-archive-downloads-javase5-419410.html

### [iterm2](https://iterm2.com/)

#### 安装HomeBrew

   ```
    xcode-select --install
    ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
   ```

#### 安装Oh My Zsh

   ```
   brew install git
   brew install Zsh
   brew install wget
   brew install unrar
   brew search 7z
   brew install p7zip
   
   sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-Zsh/master/tools/install.sh -O -)"
   
   添加alias:
   vim ~/.zshrc
   G 
   o
   
alias study="cd /Users/nibnait/github/Study_Note"
alias redis="/Users/nibnait/redis-6.0.9/src/redis-server /Users/nibnait/redis-6.0.9/redis.conf"
alias rcli="/Users/nibnait/redis-6.0.9/src/redis-cli"
alias gcu="git checkout master_uat"
alias gcr="git checkout release"
alias nrm="sudo npm run mall-dev"
alias rime="cd /Users/nibnait/Library/Rime"
# alias mvp="mvn clean package -DskipTests"
# alias mvd="mvn clean deploy -DskipTests"
alias mvp="mvn clean package -Dmaven.test.skip=true"
alias mvd="mvn clean deploy -Dmaven.test.skip=true"
alias mvc="mvn clean"

export MAVEN_HOME=/Users/nibnait/apache-maven-3.6.3
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home
export PATH=$PATH:$MAVEN_HOME/bin:/usr/local/mysql/bin:$JAVA_HOME/bin
   
   
   source ~/.zshrc
   ```

### [maven](https://maven.apache.org/download.cgi)

[MySQL](https://dev.mysql.com/downloads/mysql/5.7.html)

   ```
   mysql -u root -p
   set PASSWORD =PASSWORD('root');
   ```

### [gradle](https://gradle.org/releases/)

ssh-keygen -t rsa -C "[w@tianbin.org](mailto:w@tianbin.org)"

   git config --global user.name nibnait
git config --global user.email [w@tianbin.org](mailto:w@tianbin.org)



git config --global --edit

   ```
   [user]
           name = nibnait
           email = w@tianbin.org
   [core]
           ignorecase = false                  
   ```

## 系统快捷键

系统偏好设置->键盘->快捷键->应用快捷键

```
mail
Format->Lists->Insert Bulleted List     cmd+shift+b
Format->Lists->Insert Numbered List     cmd_shift+n

note
Format->Bulleted List     cmd+shift+b
Format->Numbered List     cmd_shift+n

```





### Chrome插件

Web前端助手（FeHelper）- crx文件安装方法

安装方法

   1. 下载当前最新版`*.crx` https://github.com/zxlie/FeHelper/tree/master/apps/static/screenshot/crx
   2. chrome浏览器地址栏输入：chrome://extensions/ 并打开
   3. 右上角开启`开发者模式`
   4. 拖拽crx到当前页面，完成安装



https://zhuzhengyuan.xyz/2018/11/23/the-compatibility-issues-in-macos-mojave/

```
终端启动报错
问题描述
打开终端时，直接报错 manpath: error: unable to read SDK settings for '/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk'
解决方案
升级 xcode 和 commandline tools.
在应用商店里升级 xcode 后，运行 xcode-select --install. 再重启终端即可。
```

