## Restful API



### POST 起手式

準備好一個 Json 放到最大的包的 body。

```js
		let retJson = { key : data }; // something
        const options = {
            method: "POST"
        };
        options.headers = {
            'Accept': 'application/json',
            'Content-Type': 'application/json'
        };
        options.body = JSON.stringify(retJson);
        var path = "/record";       
        const response = await fetch(path, options);
```



### GET 起手式

```js
		const options = {
            method: "GET"
        };
        options.headers = {
            'Accept': 'application/json',
            'Content-Type': 'application/json'
        };

        var path = "/record";
        const response = await fetch(path, options).then(function (response) {
            return response.json()
        }).catch(function (err) {
            return JSON.stringify(err);
        });
        console.log(response['data']);
```



## MongoDB

安裝 : ``npm install`` ，我自己是有指定好 package 所以直接這樣做



### mLab

部屬到 heroku 的話通常會搭配 mlab (現在不能創建了，他會跳轉到 mongo DB atlas )
不過 Atlas 也是大同小異，設定上差不多而且只要照著他的 step 引導就能建立 DB。

在開發的時候，mongo DB 可能會用本地端的做測試
不過我自己是直接連過去雲端的了，想說直接串。

按照他的步驟建立完 user、porject、設定 IP 之後，可以從 Collections 建立 DB

這邊特別註明一下，設定 IP 很關鍵，如果你只是想要先做自我測試那就讓他偵測你的 IP 就好。



![](https://i.imgur.com/WbWCXBZ.png)



建好 DB 之後在建立 Collection。



### 實作 Server.js



<https://www.w3schools.com/nodejs/nodejs_mongodb_insert.asp>



這邊附上比較簡單的寫法
同時放上 post 的 code

其餘的照著 w3c 就可以刻出來了。

** 注意 : 這個寫法不太好，通常是 new 一個 ``MongoClient`` 之後用那個 obj 一直做下去。

```js
const bodyParser = require('body-parser');
const express = require('express');
const MongoClient = require('mongodb').MongoClient;
const ObjectID = require('mongodb').ObjectID;

const app = express();
const jsonParser = bodyParser.json();
app.use(express.static('public'));

async function main() {
  // The "process.env.PORT" is needed to work with Heroku.
  const port = process.env.PORT || 9000;
  await app.listen(port);
  console.log(`Server listening on port ${port}!`);
};

main();

app.post('/record', jsonParser, (req, res) => {
  console.log(req.body);
  console.log(DB_URL);
  
  MongoClient.connect(MONGO_URL, function (err, db) {
    if (err) throw err;
    var dbo = db.db(dbName);
    dbo.collection(collectionName).insertOne(req.body, function (err, res) {
      if (err) throw err;
      console.log("1 document inserted");
      db.close();
    });
  });
})
```



常見問題

* ``MongoError: connection 5 to :27017 closed``
  * 我因為都使用 mlab，所以這個 Local 端的設定反而不該出現在我的 log 之中
    確保一下自己連到的是 DB_URL 吧
    ``MongoClient.connect(MONGO_URL, ...)``
    * 每個人 Server 這邊連線 code 可能不一樣



## Heroku

這邊其實有超多東西要講，因為部屬上去我自己摸了很久 ... 

但是很基礎的建立 Project 我不太想說 XD    
仿造各大教學文章，至少到建立 App 應該不難

建立完 App 之後你可能才會比較疑惑，然後要怎麼把東西丟上去和串服務之類的。

### Deploy (部屬)

實際上他要用 git 會比較簡單，你也是可以用 Github 連結的方式，但如果你有隱私檔案就不太方便。
所需工具 : ``Git``、``Heroku Cli``，都找官方安裝一下就好 ( [Heroku CLI](https://devcenter.heroku.com/articles/heroku-command-line))

登入

```
$ heroku login
```

選 repo 

```
$ heroku git:clone -a mvform
$ cd mvform
```

deploy

```
$ git add .
$ git commit -m "update"
$ git push heroku master
```



### 連上 mLab ? 

你可能會疑問，那我丟上去 heroku 之後，怎麼跟 mLab 做連結 ? 

大部分的教學文都是教，在 heroku 裡面裝一個 mLab app
是沒錯，但我還是不知道要怎麼跟我已經建好的 altas 裡面的 DB 連動阿 !! 

在我們寫 ``server.js`` 的時候，通常 URL 也不會直接設定值，會再多一個變數 : 

``const MONGO_URL = process.env.MONGODB_URI ||  MY_DB_URL;``

這邊的 ``MY_DB_URL`` 可能是你的直連 url，也可能是 ``mongodb://localhost:27017/myDB``，之類的。
而那個 ``process.env.MONGODB_URI``，就是 heroku 的 local config

當你在裝好 mLab for heroku 時，會自己創一個給你
也就是說，其實他已經都幫你用好了，你就算不去弄 altas 也沒關係 

但如果是像我一樣去 altas 建了 DB 拿到網址
用環境變數方式去設定的話 (``const MONGO_URL = process.env.MONGODB_URI``)
就到 heroku 的 settings 內新增 Var，``MONGODB_URI`` 對應 ``網址``



![(此圖是截自別人 proj)](https://i.imgur.com/VvUWvZy.png)

### Error

然後阿，如果你是個新手，大概有 80% 的機率會噴 Error。

以下列一些我曾經遇到的

* NPM_CONFIG_LOGLEVEL=error、" semver "

  * package.json 可能有問題，這跟後面的一起說

* ```
  remote:        engines.node (package.json):  7x 
  remote:        engines.npm (package.json):   unspecified (use default) remote:        Resolving node version 7x... 
  remote:        Error: Invalid semantic version "7x"
  ```

  * 我覺得在 deploy 最常遇到的問題就是這個，跟上面那個算是同類的
    問題在於，heroku 需要完整的 package 版本
    這部分直接看官方說法比較快 
    [build failing because of an invalid semver](<https://help.heroku.com/0ZIOF3ST/why-is-my-node-js-build-failing-because-of-an-invalid-semver-requirement>)

    實際上也很簡單，查一下主機端的版本然後寫回 ``package.json`` 就好

* ``MongoNetworkError: connection 5 to cluster0-someName.mongodb.net:27017 closed``

  - 類似這類的 error。通常，這是發生在你 Local 連 mLab 都沒問題
    但是上傳上去 heroku 卻跑出來的奇怪問題。

    解決方法 : 這是 Heroku 的 IP 被擋了。
    面對這個問題，目前有幾種方法 : 

    1. 把 IP 白名單設成 anywhere can connect，這樣肯定沒問題。
       方法也很簡單，去 altas 的 IP 設定那邊 (之前嚮導其實有帶你做過) 新增

       > Step 1) Login 
       > Step 2) Under the security tab click on IP Whitelist
       > Step 3) Edit IP address or press add new button at top right
       > Step 4) Choose allow access from anywhere or add current IP address

    2. 設定 private space 
       請參考這篇文章，看了一下想要跟著做發現有點複雜，
       若有人寫相關教學歡迎在留言通知我 XD
       [Private Space Peering](<https://devcenter.heroku.com/articles/private-space-peering#creating-a-peering-connection-to-your-private-space>)

    3. Add-on (類似插件的感覺)
       可以參考此篇最後一個 reply 
       [Reference](https://stackoverflow.com/questions/42159175/connecting-heroku-app-to-atlas-mongodb-cloud-service)

* 過程可能會需要清 cache，可以參考這篇
  [How do I clear the build cache?](https://help.heroku.com/18PI5RSY/how-do-i-clear-the-build-cache)



Okay, 那理論上可以成功 deploy 到 heroku，而上面的 權限問題 也解決了的話
基本上你的 Service 就能成功運作。



# DEMO : 

https://mvform.herokuapp.com/


以下是 GIF 展示，在錄製之前 DB 即有一條資料，故第一 turn 新增兩支跑出三支。

![https://i.imgur.com/cHPh2SD.gif](https://i.imgur.com/cHPh2SD.gif)

### Ref : 

http://www.dayid.org/comp/tm.html
https://blog.longwin.com.tw/2011/04/tmux-learn-screen-config-2011/
https://gist.github.com/spicycode/1229612
https://ithelp.ithome.com.tw/articles/10129761


<div style="text-align: center">End</div>
-----------------------------------

![](https://i.imgur.com/888jFLr.gif)

