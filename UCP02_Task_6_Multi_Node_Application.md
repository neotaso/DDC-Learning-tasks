<!--
# Task 6 - Deploy Application across multiple nodes
//-->
# タスク 6 - 複数ノードにわたるアプリケーションのデプロイ

<!--
## Pre-requisites
//-->
## 前提条件
<!--
- UCP installed with at least 2 nodes joined
- Client bundle downloaded
//-->

- 最低でも 2ノード参加している UCP をインストール済であること
- Client bundle をダウンロード済であること

<!--
## Step 1 - Download the Voting Application
//-->
## 手順 1 - 投票アプリケーションのダウンロード

<!--
1. Go to https://github.com/docker/swarm-microservice-demo-v1 and read the page to get a high level understanding of the application we are going
   to run.
2. Clone the application repository into a local folder on your machine.
3. Remove all existing applications you have deployed on UCP. 
//-->
1. https://github.com/docker/swarm-microservice-demo-v1 にアクセスし、これから実行しようとしているアプリケーションがどのようなものか十分に理解します。
2. このアプリケーションのレポジトリをローカルマシンに clone します。
3. UCP 上にデプロイしたすべての既存のアプリケーションを削除します。

<!--
## Step 2 - Run Docker Compose
//-->
## 手順 2 - Docker Compose を実行

<!--
1. Change directory to the Voting Application folder
2. Launch the application with Docker Compose. What can you observe?
//-->
1. 投票アプリケーションのディレクトリに移動します。
2. Docker Compose でアプリケーションを起動します。どうなりましたか？

   ```
   johnny@JT MINGW64 ~/Documents/GitHub/swarm-microservice-demo-v1 (master)
   $ docker-compose up -d
   ...
   ...
   Creating swarmmicroservicedemov1_vote-worker_1
   Creating swarmmicroservicedemov1_web-vote-app_1
   unable to find a node that satisfies node==frontend01
   ```
<!--
3. Try and identify the cause of the error and edit the `docker-compose.yml` file to fix it.
//-->
3. エラーの原因を特定して `docker-compose.yml` ファイルを修正しなければいけません。

<!--
   Let's take a quick look at the `docker-compose.yml` file
//-->
   `docker-compose.yml` ファイルを見てみましょう。
   
   ```
   version: '2'

   services:
     web-vote-app:
       build: web-vote-app
       environment:
         WEB_VOTE_NUMBER: "01"
         constraint:node: "=frontend01"

     vote-worker:
       build: vote-worker
       environment:
         FROM_REDIS_HOST: 1
         TO_REDIS_HOST: 1

     results-app:
       build: results-app

     redis01:
       image: redis:3

     store:
       image: postgres:9.5
       environment:
         - POSTGRES_USER=postgres
         - POSTGRES_PASSWORD=pg8675309
   ```
   
<!--
   Notice the line saying `constraint:node: "=frontend01"`. 
   This tells Docker that the service container must be run on a Node called `frontend01`. We don't have such a Node and hence the error message.
//-->
   `constraint:node: "=frontend01"` という行に注目してください。
   これはサービスのコンテナを `frontend01` というノード上で実行しなければならない、と Docker に指示するものです。そんなノードが存在しないので、エラーとなってしまうのです。
   
<!--
   This Compose file was written with a development machine in mind as opposed to a production deployment. Hence why each service has a `build` instruction instead
   of an `image` instruction. 
//-->
   この Compose は本番環境でのデプロイ向けではなく、開発マシンのために書かれています。そのため、各サービスは `image` 行の代わりに `build` 行を持っているのです。
   
<!--
   Delete the `constraint:node` line and save the file.
//-->
   `constraint:node` 行を削除してファイルを保存します。
   
<!--
3. Once the error has been fixed, launch the application again.
//-->
   エラーを修正したので、再度アプリケーションを起動します。
   
<!--
   This time there shouldn't be any errors.
//-->
   今度はエラーが出ないはずです。
   
<!--
   > **Note:** Sometimes, you may run into the following type of error. This depends on the setup of your AWS VM and usually occurs on Linux kernel version 3.13 or below
//-->
   > **注意:** ときおり、次のようなエラーになることがあります。これは AWS VM の設定が原因で、Linux Kernel のバージョンが 3.13 以下で起きることがあります。
   
   ```
   409 Conflict: subnet sandbox join failed for "70.28.0.0/16": overlay subnet 10.0.0.0/16 has conflicts in the host while running in host mode
   ```
   
<!--
   This is because Compose is trying to create an Overlay network on a subnet that is already being used by the host. This is due to the way AWS allocates
   addresses. If you run into this error, add the following configuration to the end of your `docker-compose.yml` file. 
//-->
   これは Compose がオーバーレイネットワークをホストが既に利用しているサブネットに作ろうとしているためです。AWS がアドレスを確保する方法によるものです。もしこのエラーが起きたら、 `docker-compose.yml` ファイルの最後尾に次の設定を追加します。
   
   ```
   networks:
     default:
       ipam:
         config:
           - subnet: 10.10.10.0/24
   ```
   
<!--
   The error can also be resolved by upgrading the Linux Kernel to version 3.19 or later. 
//-->
   このエラーは Linux Kernel を 3.19 以上にアップグレードすることでも解決できます。
   
<!--
4. Check the application is up and running and that the containers are running across different nodes. 
//-->
4. アプリケーションが起動して実行していることを確認し、異なるノードにわたってコンテナが実行していることも確認します。

<!--
5. Check the **Networks** page and verify that a network called `swarmmicroservicedemov1_default` has been created.
//-->
5. **Networks** ページを確認して `swarmmicroservicedemov1_default` というネットワークが作成されていることを確認します。

<!--
   You should see that the network exists. The name is derived from the name of the project folder. The network should be using the `overlay` driver.
   This tells us that it is a multi host network.
//-->
   このようなネットワークが存在していることがわかります。この名前はプロジェクトのディレクトリ名から付けられます。このネットワークは `overlay` ドライバを使っているはずです。
   これはマルチホストネットワークであることを示しています。
   
   ![](images/IG_ucp02_t6_Networks.PNG)
   
<!--
## Step 3 - Use a Production ready configuration
//-->
## 手順 3 - 本番環境向けの設定を使う
 
<!--
Our `docker-compose.yml` file didn't necessarily follow the best practices. Ideally you shouldn't build your images when deploying your application. The Compose
file should be running pre-built images from a registry. Remember, developers build the images and push them to the registry, while IT operations runs the images.
//-->
この `docker-compose.yml` ファイルはベストプラクティスに必ずしも従っていませんでした。理想的には、アプリケーションをデプロイするときにはイメージをビルドしないほうがいいでしょう。Compose ファイルはレジストリからビルド済みのイメージを取得するようにすべきです。開発者はイメージをビルドしてレジストリにプッシュし、IT 運用者はイメージを実行するということを覚えておいてください。

<!--
1. Delete the Voting Application from the **Applications** page in UCP.  
//-->
1. UCP の **Applications** ページから投票アプリケーションを削除します。
   
<!--
2. Take a look at the Compose file at [](https://github.com/nicolaka/voteapp-base/blob/master/docker-compose.yml)https://github.com/nicolaka/voteapp-base/blob/master/docker-compose.yml
//-->
2. [](https://github.com/nicolaka/voteapp-base/blob/master/docker-compose.yml)https://github.com/nicolaka/voteapp-base/blob/master/docker-compose.yml の Compose ファイルを見てください。

<!--
   Compare this to the Compose file in the Voting App repo. What do you notice as the difference?
//-->
   このファイルと投票アプリのレポ中の Compose ファイルを比較してください。違いは何でしょうか？

<!--
3. Clone the Voteapp Base repository into a folder on your PC / Mac
//-->
3. 投票アプリのベースレポジトリをローカルマシンに clone します。

   `$ git clone https://github.com/nicolaka/voteapp-base`
   
<!--
4. Open the `docker-compose.yml` file in the `voteapp-base` folder and change the port mapping of the `voting-app` service. Map port 80 in the container to port 80 on the host.
//-->
4. `voteapp-base` ディレクトリ内の `docker-compose.yml` ファイルを開き、 `voting-app` サービスのポートマッピングを変更します。コンテナの 80番ポートをホスト上の 80番ポートにマップします。

<!--
   To do this, look for the section:
//-->
   次の部分を:
   ```
   voting-app:
     image: docker/example-voting-app-voting-app
     ports:
       - "80"
   ```
   
<!--
   Change it to 
//-->
   次のように変更します:
   ```
   voting-app:
     image: docker/example-voting-app-voting-app
     ports:
       - "80:80"
   ```
   
<!--
   The reason for this is so that as IT operations, we control which ports services are exposed on as opposed to having the port automatically mapped. Also, if 
   we use automatic port mapping, Docker will choose a port that is most likely blocked by AWS, therefore preventing external access to our application.
//-->
   なぜこうするのかというと、IT 運用のため、ポートを自動的にマップするのではなく、どのポートでサービスを公開するかを制御するためです。また、自動ポートマッピングを使うと、Docker が選択するポートはだいたい AWS によってブロックされているので、外部からアプリケーションにアクセスできなくなるためです。
   
<!--
5. Repeat the same procedure, for the `result-app` service. Map port 80 to port 5000
//-->
5. `result-app` サービスにも同様の手順を行います。80番ポートを 5000番ポートにマップします。
   
<!--
6. Launch the application
//-->
6. アプリケーションを起動します。
   
<!--
7. Verify that the application containers are scheduled on different nodes.
//-->
7. アプリケーションのコンテナが異なるノードにスケジュールされていることを確認します。

<!--
8. Check the **Networks** page and verify that an overlay network with the name `voteappbase_voteapp` was created.
//-->
8. **Networks** ページを確認し、 `voteappbase_voteapp` という名前のオーバーレイネットワークが作成されていることを確認します。
   
<!--
## Step 4 - Test the application
//-->
## 手順 4 - アプリケーションのテスト

<!--
1. Open the `voteappbase_voting-app_1` on your web browser. You should see the following:
//-->
1. ウェブブラウザで `voteappbase_voting-app_1` を開きます。次のようになるはずです:

   ![](images/IG_ucp02_t6_voting_cats_or_dogs.PNG)

<!--
   This is the web application where we can cast a vote. From our configuration, we know that the container is available through port 80 on the host. Therefore
   we need to find out what node the container is running on and point our browser to the URL of that node.
//-->
   これは投票できるウェブアプリケーションです。設定より、ホストの 80番ポートを通してコンテナにアクセスできることがわかっています。よって、コンテナが動作しているのがどのノードか探して、そのノードを URL としてブラウザに入力する必要があります。
   
<!--
2. Cast your vote.
//-->
2. 投票します。

<!--
3. Open `voteappbase_result-app_1` on your web browser and see the results of your vote.
//-->
3. ウェブブラウザで `voteappbase_result-app_1` を開き、投票結果を見てみます。
   
<!--
   You should see the following screen, albeit with a different percentage allocation depending on how you cast your vote.
//-->
   次のような画面になります。どちらに投票したかによって、異なる割合が表示されるはずです。
   
   ![](images/IG_ucp02_t6_VoteResults.PNG)
   
<!--
## Step 4 - Convert an existing application to run on multiple nodes.
//-->
## Step 5 - 既存のアプリケーションを複数ノードで実行するよう変換する

<!--
Previously, when we ran the HelloRedis application, you may remember that both containers were scheduled onto the same node. The reason was because
the Compose file in that application was in the V1 format and that we used container links to get our services talking with each other. When links 
are used, Swarm will schedule a container on the same node as the container it is linked to.  
//-->
以前、HelloRedis アプリケーションを起動したとき、両方のコンテナが同じノードにスケジュールされていたことを思い出してください。なぜかというと、アプリケーションで使っていた Compose ファイルが V1 フォーマットで、サービス同士が通信する際にコンテナの links 機能を使っていたからです。links 機能を使うと、Swarm はコンテナをリンクさせるために同じノードにコンテナをスケジュールします。

<!--
This was the `docker-compose.prod.yml` file we used previously 
//-->
以前に使った `docker-compose.prod.yml` ファイルはこちらです:

```
javaclient:
  image: trainingteam/hello-redis:1.0
  links:
    - redis:redisdb
redis:
  image: redis
```

<!--
If we deleted the "links:" instruction on our `javaclient` service, we could get Swarm / UCP to run the containers on different nodes. However, 
the containers would be run on the default network and will not be able to communicate with each other using their service names. You would have to use
the Node IP and exposed ports, which is not ideal.
//-->
`javaclient` サービスの "links:" 部分を削除すると、Swarm と UCP はコンテナを異なるノード上に配置するでしょう。しかし、コンテナは異なるネットワーク上で動作するので、サービス名を使ってお互いに通信することができません。ノードの IP アドレスとポートを解放して使わなければいけませんが、それは理想的な形ではありません。
   
<!--
The best solution to get our `HelloRedis` example working on multiple nodes is to convert the `docker-compose.prod.yml` file to a `V2` file.
//-->
`HelloRedis` の例を複数ノードで動作させるためのベストな方法は、 `docker-compose.prod.yml` ファイルを `V2` ファイルに変換することです。

<!--
1. Convert the `docker-compose.prod.yml` file to a Version 2 Compose file.
//-->
1. `docker-compose.prod.yml` ファイルをバージョン 2 Compose ファイルに変換します。

<!--
   **Hint**: The `redis` service name will need to be changed to `redisdb` as the `javaclient` code uses this name to connect to the `redis` service
//-->
   **ヒント**: `redis` サービス名は `redisdb` に変更する必要があります。 `javaclient` は `redis` サービスに接続する際の名前として `redisdb` を使っているからです。

<!--
2. Run the HelloRedis application and verify that the containers are deployed on different nodes. 
//-->
2. HelloRedis アプリケーションを実行し、コンテナが異なるノードにデプロイされていることを確認します。

 

   
   
