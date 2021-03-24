# helloworld

[![Build Status](https://cloud.drone.io/api/badges/go-training/helloworld/status.svg)](https://cloud.drone.io/go-training/helloworld)

Hello World for Golang

## Simple Command

Run golang program

```bash
go run main.go
```

Testing

```bash
go test
```

Build binary

```bash
go build
```

Install binary

```bash
go install
```
# 弄懂 drone
## 參考資料
*  ref[Drone CI/CD 配合 Github 使用 Rsync 進行 Deploy](https://cola.workxplay.net/drone-ci-cd-and-github-rsync-deploy/), tutorial[link](https://www.youtube.com/watch?v=-U2EXs0tmN0)

## 系統需求
*  drone 是基於 docker 跟 docker-compose,先裝 docker 並啟用
*  也需要 git, 如果 server 沒有對外ip 需要 ngrok 安裝並申請帳號

## 安裝流程
*  先下載一個參考用的 docker-compose.yml[link](https://github.com/wyubin/helloworld/blob/master/drone/docker-compose.yml)
*  建立一個 .env 跟 docker-compose.yml 放在一起，把docker-compose.yml 的參數先填好
```shell
DRONE_SERVER_HOST=
DRONE_SERVER_PROTO=
DRONE_RPC_SECRET=
DRONE_RPC_HOST=
DRONE_RPC_PROTO=

DRONE_GITHUB_CLIENT_ID=
DRONE_GITHUB_CLIENT_SECRET=
```
*  先建立好 ngrok 設定檔提供開啟 service, 然後啟用
```text
authtoken: <ngrok-auth-token>
tunnels:
    default:
        proto: http
        addr: 8081
```
```shell
nohup ./tools/ngrok start --all --log=stdout --config="/home/yubin/.ngrok2/drone.yml" > ./logs/ngrok.log &
```
到 ngrok.log 確認 ngrok 所 forward 的 address(exp: 8b4478c18035.ngrok.io)

*  到 github 去註冊 oauth 服務
  *  github -> settings -> develop settings -> oauth apply
  *  fill ngrok.io https into homepage and authorization callback(with /login)
  *  記住 CLIENT_ID 跟 CLIENT_SECRET

*  編輯 .env
將相關參數填寫如下
```text
DRONE_SERVER_HOST=<ngrok.io addr>
DRONE_SERVER_PROTO=https
DRONE_RPC_SECRET=trial2drone
DRONE_RPC_HOST=drone-server
DRONE_RPC_PROTO=http

DRONE_GITHUB_CLIENT_ID=<github client_id>
DRONE_GITHUB_CLIENT_SECRET=<github client_secret>
```

*  開啟服務
```shell
docker-compose -f drone/docker-compose.yml up -d
```

*  連到 drone 的介面，用瀏覽器開 8b4478c18035.ngrok.io
  *  將想要追蹤的repo activate 之後，再寫好要執行 .drone.yml 放在 repo 中
  ```yml
  ---
  kind: pipeline
  name: default

  steps:
  - name: backend
    image: golang:1.13-alpine
    environment:
      USERNAME:
        from_secret: username
    commands:
    - go build
    - go test
  ```

*  再來就是PR跑測試了

# 注意事項
*  drone 開啟時會存 database.sqlite 在 docker-compose.yml 的資料夾，如果重開一個 drone 會需要清掉這個database，不然會無法連接 github 的 push
*  如果只有設定 master 就不會啟動其他 branch 的幾個行為：push, 但對 master 的 pull request 會啟動