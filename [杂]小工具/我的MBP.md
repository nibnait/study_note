# 我的mbp
[搜狗输入法](https://pinyin.sogou.com/mac/)  
[百度网盘](https://pan.baidu.com/)  
[Surge](http://nssurge.com/)  

## 触发角

## App Store
- 万年历
- Manico


## 百度云
- Spectacle+1.2
- Alfred     youdao
- [typora](https://www.typora.io/#download)
- [Snipaste](https://zh.snipaste.com/)

## 去官网下载最新版
### [XCode](https://developer.apple.com/download/more/?=xcode)
### [clean my mac](https://macpaw.com/cleanmymac)
### [MarginNote](https://www.marginnote.com/chinese/home)

[Sourcetree](https://www.sourcetreeapp.com/)

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

历史版本：[https://www.jetbrains.com/idea/download/other.html](https://www.jetbrains.com/idea/download/other.html)

```
"Configure" 或 "Help" -> "Edit Custom VM Options ..."：
-javaagent:/Users/nibnait/jetbrains-agent.jar
-javaagent:/Users/nibnait/Library/Mobile\ Documents/com\~apple\~CloudDocs/pirvate Files/jetbrains-agent.jar

License server激活：http://jetbrains-license-server

Code Templates：
/*

Created by ${USER} on ${DATE}
 */
public class ${NAME} extends TestCase {

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
     xcode-select --install
     ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

#### 安装Oh My Zsh
    brew install git
    brew install Zsh
    brew install wget
    sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-Zsh/master/tools/install.sh -O -)"
    
    添加alias:
    vim ~/.zshrc
    G 
    o
    alias study="cd /Users/nibnait/Library/Mobile\ Documents/com\~apple\~CloudDocs/Study_Note/"
    source ~/.zshrc


### [maven](https://maven.apache.org/download.cgi)
### [gradle](https://gradle.org/releases/)

ssh-keygen -t rsa -C "w@tianbin.org"  

git config --global user.name nibnait  
git config --global user.email  w@tianbin.org

```
git config --edit
[core]
        ignorecase = false
```

```
git config --global --edit
[user]
        name = nibnait
        email = w@tianbin.org
[core]
        ignorecase = false                  
```



### Chrome插件

Web前端助手（FeHelper）- crx文件安装方法

安装方法

1. 下载当前最新版`*.crx`  [https://github.com/zxlie/FeHelper/tree/master/apps/static/screenshot/crx](https://github.com/zxlie/FeHelper/tree/master/apps/static/screenshot/crx)
2. chrome浏览器地址栏输入：[chrome://extensions/](chrome://extensions/) 并打开
3. 右上角开启`开发者模式`
4. 拖拽crx到当前页面，完成安装
