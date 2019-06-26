# GCP

### GAE

#### deploy command
--- 
	gcloud app deploy --project PROJECT_ID --version VERSION_ID --no-promote
	gcloud app deploy ./appengine/backend/app.yaml --project miraca-estimate-dev --version 20190509t144616 --promote
--------

### Cloud SQL
-----
<strong>「proxyを使用してローカルから接続する手順」</strong><br><br>
[https://cloud.google.com/sql/docs/mysql/quickstart-proxy-test?hl=ja](https://cloud.google.com/sql/docs/mysql/quickstart-proxy-test?hl=ja)<br>
[https://cloud.google.com/sql/docs/mysql/connect-external-app?hl=ja#go](https://cloud.google.com/sql/docs/mysql/connect-external-app?hl=ja#go)<br><br>
1.gcloudコマンドラインツールのインストールと認証<br>
2.ローカルマシンにCloudSProxyクライアントをインストールする<br><br>

	wget https://dl.google.com/cloudsql/cloud_sql_proxy.linux.amd64 -O cloud_sql_proxy

	chmod +x cloud_sql_proxy
3.プロキシを開始する
	
	  
	./cloud_sql_proxy -instances=<INSTANCE_CONNECTION_NAME>=tcp:3306
	
	
	

4.mysql クライアントを使用してデータベースに接続する
	  
	mysql -u <USERNAME> -p --host 127.0.0.1 --port 3306
	
5.Go Proxy ライブラリ
	  
	  
	import  (  "github.com/GoogleCloudPlatform/cloudsql-proxy/proxy/dialers/mysql"  
	)  
	...cfg := mysql.Cfg(<INSTANCE_CONNECTION_NAME>,  <DATABASE_USER>,  <PASSWORD>)  
	cfg.DBName  =  <DATABASE_NAME>  
	db, err := mysql.DialCfg(cfg)

------
	import (
	"database/sql"
	"fmt"
	"log"
	"strconv"
	"time"
	"github.com/gin-gonic/gin"
	"google.golang.org/appengine"
	// _ "github.com/lib/pq"
	// cloudsqlproxy 	
	"github.com/GoogleCloudPlatform/cloudsql-proxy/proxy/dialers/mysql"
	_ "github.com/go-sql-driver/mysql"
	// "golang.org/x/net/context"
	// "github.com/jmoiron/sqlx"
	// "gopkg.in/gorp.v1"
	"gopkg.in/gorp.v1"
	_ "gopkg.in/gorp.v1"
	)

	userName  :=  "miraka-est"
	pass  :=  "miraca-est@2019"
	connectionName  :=  "miraca-estimate-dev:asia-northeast1:kame-evo-db"
	var  db  *sql.DB
	dbURI  := fmt.Sprintf("%s:%s@unix(/cloudsql/%s)/backend_ys", userName, pass, connectionName)

	db, err  := sql.Open("mysql", dbURI)
	if err !=  nil {
	log.Println(err)
	log.Println("==== SQL Open error occurred")
	}

	dbmap  :=  &gorp.DbMap{Db: db, Dialect: gorp.MySQLDialect{}}
	dbmap.AddTableWithName(TrialDetail{}, "est_trial_detail").SetKeys(false, "Company_code")

### 「実行」

	/usr/local/bin/cloud_sql_proxy -instances=miraca-estimate-dev:asia-northeast1:kame-evo-db=tcp:8888

	mysql -u miraka-est -p --host 127.0.0.1 --port 8888
	
	dev_appserver.py ./app.yaml --port 9999

-----

<strong>「swagger merger」</strong>

	swagger-merger -i ./yaml-split/docs/s003.yml -o ./appengine/openapi/s003.yaml

-------

### Gorp

・refrectionを用いているため、nativeSQLより2,3%遅い
・「AddTableWithName」で構造体とテーブルを紐付ける必要がある
・BulkInsert,BulkUpdate機能はないため、nativeSQLで対応

-------
### MySQL
<font color=red>構造体のフィールド名は、カラムのキャピタル大文字にしないとエラー→reflect.Value.Interface: cannot return value obtained from unexported field or method</font>
・tinyint = bool?
・DECIMAL = COMPLEX128?
・datetime = string : <font color=blue>time.Now().Format("20060102150405")</font>

-------
### Jenkins
<strong>起動</strong>

	java -jar jenkins.war --httpPort=60000


-------
### Shell
<strong>実行権限を与える</strong>

	chmod +x hello.sh


-------
###  Git
<strong>テンプレ</strong>

	git add .
	git commit -m "fix: 修正"
	git push origin HEAD
	
<strong>上記短縮シェル</strong>

	cd /Users/y_sato/go/src/[Project Name]
	git add .
	git commit -m $1
	git push origin $2
	
<strong>上記シェルコマンド</strong>

	/Applications/tool/lazyGitPush.sh "コメント" [Branch Name]
	


-------
### Go format and Coverage
<strong>フォーマットチェック</strong>

	gofmt #directory -e
	go vet package/path/name # package単位
	go tool vet /path/to/*.go # file単位
	go tool vet /path/to/directory # directory単位
	golint #directory

<strong>カバレッジ</strong>

	go test -cover ./feature/
	
	go test -coverprofile=cover.out ./feature/
	go tool cover -html=cover.out -o cover.html
	open cover.html

-------
