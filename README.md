# 專案說明
此專案使用laravel-websockets（ https://docs.beyondco.de/laravel-websockets/1.0/getting-started/installation.html ）作為pusher服務，
並把它打包成docker image （yudada/pusher）
## 開啟環境步驟

1. 在.env中指定相關參數，如不使用SSL連線請將LARAVEL_WEBSOCKETS_SSL_LOCAL_CERT 和 LARAVEL_WEBSOCKETS_SSL_LOCAL_PK 指定null
2. 如使用SSL連線需先產生自簽憑證，打開ssl_crt/ssl.conf 條整 [alt_names]區段，把會用到域名填上去．
透過以下命令就可以建立出 私密金鑰 (server.key) 與 憑證檔案 (server.crt)：
`openssl req -x509 -new -nodes -sha256 -utf8 -days 3650 -newkey rsa:2048 -keyout server.key -out server.crt -config ssl.conf`
3. .env中的參數需設定跟laravel專案中的一至

4. config/broadcasting.php 中 pusher區塊需調整如下
   ```php
            'pusher' => [
                'driver' => 'pusher',
                'key' => env('PUSHER_APP_KEY'),
                'secret' => env('PUSHER_APP_SECRET'),
                'app_id' => env('PUSHER_APP_ID'),
                'options' => [
                    'cluster' => env('PUSHER_APP_CLUSTER'),
                    'host' => 'docker.for.mac.host.internal',
                    'port' => 6001,
                    'scheme' => (empty(env('LARAVEL_WEBSOCKETS_SSL_LOCAL_CERT')) && empty(env('LARAVEL_WEBSOCKETS_SSL_LOCAL_PK'))) ? 'http' : 'https',
                    'curl_options' => [
                            CURLOPT_SSL_VERIFYHOST => 0,
                            CURLOPT_SSL_VERIFYPEER => 0,
                        ]
                ],
            ],
    ```
5. resources/assets/js/bootstrap.js 中的Laravel Echo設定如下

    ```js
    import Echo from 'laravel-echo';
   
    window.Pusher = require('pusher-js');
    window.Echo = new Echo({
      broadcaster: 'pusher',
      key: $('meta[name="pusher-key"]').attr('content'),
      encrypted: true, //使用ws連線需給false
      wsHost: window.location.hostname,
      wsPort: 6001,
      wssPort: 6001,
      enabledTransports: ['ws', 'wss'],
      disableStats: true,
    });
   ```
   
6. 下指令 `docker-compose up -d` 運行服務

##Debug Dashboard

WebSocket儀表板預設位置是 `/ laravel-websockets`。如使用SSL連線,請改https協議進去儀表板

## dockerfile內容

yudada/pusher : https://github.com/yudada0312/dockerfile_pusher/blob/master/dockerfile