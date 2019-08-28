# Jenkins自動編譯Github專案
Use Jenkins for auto export ipa
Jenkins auto build projecet and export .ipa on Github by Xcode plugin and xcodebuild script.

以下為建置步驟，我會針對我覺得比較容易卡關的部分來強調，網路資源偏多的就不太著重了。

## 1. Xcode設置
產生一個Xcode專案後將Automaticallty manage signing關閉，並在[Apple Developer](https://developer.apple.com/account/)分別設置好Debug & Release的Provisioning files下載下來些收好之後會使用。  
![Auto_01](_v_images/20190826153737402_546377414.png)  

## 2. Jenkins建置
可以到[這裡](https://jenkins.io/download/)來下載安裝檔。
另外Jenkins需要有Java但Mac並沒有內建，可以到[這裡](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)安裝Java，安裝過程就順勢一路下去就好，就不特別做教學了。  
打開Jenkins後先去安裝PlugIn，如果打不開請確認一下Java有沒有成功安裝。  
> Jenkins -> 管理Jenkins -> 管理外掛程式 -> 可用的   
![Auto_03](_v_images/20190827092843823_23822181.png)  

搜尋 **Xcode Integration** 將PlugIn進行安裝，之後重啟Jenkins。  

### 2-1 Proviosiong files
因為jenkins也是user的關西，所以會有自己的鑰匙圈與provisioning files的位置，我們可以直接把兩個資料夾從自己的使用者位置移動到jenkins底下的位置
```path
/Users/{YOUR_USER_NAME}/Library/MobileDevice/Provisioning\ Profiles
/Users/{YOUR_USER_NAME}/Library/Keychains
```
to
```path
/Users/Shared/Jenkins/Library/MobileDevice/Provisioning\ Profiles
/Users/Shared/Jenkins/Library/Keychains
```
到這一步驟就完成了環境的建置了。

### 2-2 Download Code
接下來會用兩種方式來完成可以Export IPA，請先把本地建好的專案上傳到Github上。 
> Jenkins -> 新增作業 -> 建置Free-Style專案  
![Auto_06](_v_images/20190827114903116_790962237.png)  

把Git網址填入，然後選擇用master branch當標準，有別的需求的話也可以選別的branch。
然後儲存後進行第一次建置專案，確認成功後，我們可以在以下路徑看到我們的專案已經被導入了
```path
/Users/Shared/Jenkins/Home/workspace
```
![Auto_07](_v_images/20190827140515207_343062332.png)  
到這裡基本上不應該有建制失敗的情況，如果有請確認一下是否有什麼事情多做或是少做了。  

### 2-3 Xcode Integration
選擇**組態**回到管理的頁面，然後拉到建置的部分選擇下拉選單到**Xcode**
為了統一編譯環境一樣加入script，若是沒有上傳Pods資料夾在加入第二行script
```script
POD_PATH=/usr/local/bin/pod
${POD_PATH} update --verbose --no-repo-update
```

#### General build settings
![Auto_08](_v_images/20190827144818413_744138648.png)   
Setting -> Xcode Schema File -> 填入專案名稱  -> Generate Archive? -> YES
![Auto_12](_v_images/20190828150825017_1628820302.png)

#### Code signing & OS X keychain options  
![Auto_09](_v_images/20190827144931779_1781909255.png)  

![Auto_10](_v_images/20190827145013553_160067401.png)  
Keychain path請輸入
```PATH
${HOME}/Library/Keychains/login.keychain
```
#### Advanced Xcode build options
用Pod或其他Workspace的情況  
***Xcode Workspace File > 輸入專案名稱***  
有多個Project要指定的情況  
***Xcode Project File > 輸入專案名稱***  
指定編譯資料夾
***${WORKSPACE}/build***
這樣會建置在 
```path
/Users/Shared/Jenkins/Home/workspace/{PROJECT_NAME}/build
```

完成以上進行建置後，就會在上述位置看到完全完成的檔案了。
之後可以再使用shell script把檔案搬移到指定位置，會是最初在build folder就指定到想要的位置。
![Auto_13](_v_images/20190828151141723_841582754.png)


### 2-4 Xcodebuild script
請先確認電腦中有安裝xcodebuild不過通常有安裝Xcode10以上的版本都會內建，所以應該不用擔心。
