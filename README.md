# Socket IO 聊天室範例 tutorial

1. 在terminal 輸入

	```shell
	express socketIOChat
	```

2. 生成完畢後進入該專案folder以及安裝必需套件

	```shell
	cd socketIOChat && npm install
	```

3. 安裝 socket.io server library並紀錄在package.json裡

	```shell
	npm install socket.io --save
	```

4. 在開始之前，我們先將一些UI的東西做好設定，請在 views 的路徑下新建一個檔案'index.html'，在該檔案裡貼上[這些程式碼](https://raw.githubusercontent.com/pofat/Socket-IO-chat-example/master/views/index.html)
5. 在 public/stylesheets 的路徑下新增一個檔妹 'chat.css' ，並貼上[這些程式碼](https://raw.githubusercontent.com/pofat/Socket-IO-chat-example/master/public/stylesheets/chat.css)
6. 接著對 socket IO 進行設定，首先在server端我們要先設定socket IO的 server。首先找到Express 啟動server時第一個執行的檔案： bin/www ，裡面包含啟動一個http server 的設定（包含listen的port等）
7. 在`var http = require('http');` 這行後面加上以下內容，以匯入socket.io server modul

	 ```javascript
	 var socketIO = require('socket.io');
	 ```

8. 在`var server = http.createServer(app);`這行程式碼後加上以下內容，這裡我們啟動socket.IO的server，並設定監聽'connection'事件的handler（其中 on 為監聽事件，emit 為送出事件）
	```javascript
	/**
 	 *  Init socket io and set event handler
     */

	var io = socketIO(server); //run socket io server

	io.on('connection', function(socket) { // listen to 'connection'
  	  debug('client ' + socket.id + ' connected');
      socket.emit('system message', 'Connected with id:'+socket.id); //send a system message event
        socket.on('message', function(msg) { // listen to 'message'
        io.emit('message', "YOU SAID: " + msg); // send message
      });
    });
	
	```

9. 然後回到 index.html，我們在第26行的 &lt;script&gt;&lt;/script&gt; 裡加入以下code:

	```javascript
	var socket = io.connect('http://localhost:3000');
	
	socket.on('system message', function(msg){
	  console.log(msg);
	});
	```
9. 接下來設定路由，讓你輸入網址時能找到index.html，在 routes/index.js 這個檔案中，在上面加入`var path = require('path');`
10. 在 get home page 的endpoint裡，將response修改成如下

	```javascript
	router.get('/', function(req, res, next) {
	  res.sendFile(path.resolve('views/index.html'));
	});
	```

11. 回到terminal中，在你的project路徑下輸入 `DEBUG=socketIOChat:* npm start` 以啟動server，並在瀏覽器中打開console並輸入 `http://localhost:3000`
12. 你應該可以在terminal中和瀏覽器的console中各別看到連接成功的訊息
13. 接下我們讓client端能夠送訊息到server去，在  index.html中，步驟9所加入的code後面再加上下列程式碼，以讓送訊息button按出去時，能夠處理該個form的submit(其中的.append是在UI中增加你自己送出去的訊息， .animate是在畫面到底時能夠以動畫的方式向下滾到底)

	```javascript
	$('form').submit(function(){
      socket.emit('message', $('#m').val());
      $('#messages').append($('<div class="message to">').text($('#m').val()));
      $("#messages").animate({ scrollTop: $('#messages').prop("scrollHeight")}, 1000);
      $('#m').val('');
      return false;
    });
	
	```

14. 接下來還要讓client 有接收 server端訊息的能力，在上面的code後加上：

	```javascript
	socket.on('message', function(msg){
	  $('#messages').append($('<div class="message from">').text(msg));
	});
	```

15. 重複步驟12，你現在可送訊息出去，網頁上將看到你丟出去的訊息，server也會即時在你的訊息前面加上 "YOU SAID:" 後回傳並顯示