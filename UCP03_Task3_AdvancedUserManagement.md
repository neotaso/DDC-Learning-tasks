<!--
# Task 3 - Advanced User Management
//-->
# タスク 3 - 高度なユーザ管理

<!--
In this Task you will complete create new users, teams and resource labels. You will then deploy a web app and test access to the web apps resources. In order to do this you will complete the following steps:
//-->
このタスクでは、新しいユーザ、チーム、リソースラベルの作成を行います。そしてウェブアプリをデプロイし、ウェブアプリのリソースへのアクセステストを行います。そのためには次の手順を完了する必要があります。

<!--
1. Create additional users and teams
2. Deploy a Compose application
3. Check access
//-->
1. ユーザとチームの追加
2. Compose アプリケーションのデプロイ
3. アクセスの確認

<!--
## Pre-requisites
//-->
## 前提条件

<!--
- Must have completed all steps in Task 1 and Task 2
//-->

- タスク 1とタスク 2のすべての手順を完了していること

<!--
## Step 1 - Create additional users and teams
//-->
## 手順 1 - ユーザとチームの追加

<!--
In this step you will create a new user, a new team, and some new permissions labels. You will use these in the next step.
//-->
この手順では、新しいユーザ、新しいチーム、いくつか新しいパーミッションラベルを作成します。これらは次の手順で使います。

<!--
1. Make sure you are logged into UCP as the **admin** user.
//-->
1. UCP に **admin** ユーザでログインします。

<!--
2. Create an additional user with the following details.
//-->
2. 次の設定でユーザを追加します。

<!--
  Be sure to make a note of the password you set, as you will need this later.
//-->
  パスワードを覚えておいてください。後で使います。

| Username   | Full Name       | Default Permissions  |
| :--------- | :-----------    | :------------------- |
| chloe      | Chloe Operate   | Full Control         |


<!--
2. Create a new team called **"Operations"** and make sure **TYPE** is "Managed".
//-->
2. **Operations** という新しいチームを作成します。 **TYPE** は「Managed」とします。

<!--
3. Add user **Chloe** to the new **Operations** team.
//-->
3. **Chloe** ユーザを **Operations** チームに追加します。

<!--
4. With the **Operations** team selected, click on the **Permissions** tab and create the following **Labels**.
//-->
4. **Operations** チームを選択し、 **Permissions** タブをクリックし、次の **Labels** を作成します。

| Resource Label   |   Permission         |
| :-------------   |   :------------------|
| production       |   Full Control       |
| development      |   Full Control       |
| qa               |   Full Control       |

<!--
5. Switch over to the **Engineering** team and create the following **Labels**.
//-->
5. **Engineering** チームに切り替えて、次の **Labels** を作成します。

| Resource Label   |   Permission         |
| :-------------   |   :------------------|
| production       |   View Only          |
| development      |   Full Control       |
| qa               |   View Only          |


<!--
## Step 2 - Deploy a Compose application
//-->
## 手順 2 - Compose アプリケーションのデプロイ

<!--
In this step you will deploy a Dockerized web app and apply the "production" label to it.
//-->
この手順では、Docker 化したウェブアプリをデプロイし、「production」ラベルを適用します。

<!--
1. Login to UCP as the user **chloe** and download her client bundle.
//-->
1. ユーザ **chloe** で UCP にログインし、client bundle をダウンロードします。

<!--
2. Unzip the client bundle into a folder of your choice.
//-->
2. client bundle をお好みのディレクトリに unzip します。

<!--
3. Connect to UCP using the client bundle.
//-->
3. client bundle を使って UCP に接続します。

<!--
  If you need details on how to do this, refer back to Step 3 in Task 2.
//-->
  どのようにするかわからなければ、タスク 2 の手順 3 に戻って確認してください。

<!--
4. Clone the https://github.com/johnny-tu/HelloRedis repository from withint the directory that contains Chloe's unzipped client bundle.
//-->
4. Chloe の client bundle を unzip したディレトリ内に、https://github.com/docker-training-ja/HelloRedis レポジトリを clone します。

  ```
  git clone https://github.com/docker-training-ja/HelloRedis
  ```

<!--
5. Add the **production** lable to both services in the `docker-compose.prod.yml` file.
//-->
5. `docker-compose.prod.yml` ファイルの両方のサービスに **production** ラベルを追加します。

<!--
   When completed, your `docker-compose.prod.yml` file should look like the following:
//-->
   完了したら、 `docker-compose.prod.yml` ファイルは次のようになるはずです:

   ```
javaclient:
    image: dockertrainingja/hello-redis:20160712
    links:
    - redis:redisdb
    labels:
      com.docker.ucp.access.label: "production"
redis:
    image: redis:3.2
    labels:
      com.docker.ucp.access.label: "production"
   ```

<!--
6. Launch the application with the following command:
//-->
6. 次のコマンドでアプリケーションを起動します。

  ```
  $ docker-compose -f docker-compose.prod.yml up -d
  ```

<!--
  > This step may take a few minutes while images are pulled and the application is started.
//-->
  > この手順では、イメージを pull してアプリケーションが起動するまでしばらく時間がかかります。

<!--
  If you experience issues with this step, ensure that you have correctly set the environment variables using the script file supplied as part of Chloe's client bundle.
//-->
  もしこの手順で問題が起きたら、Chloe の client bundle が提供しているスクリプトファイルが環境変数を正しく設定しているか確認してください。

<!--
7. Click on the **Applications** link in the UCP Dashboard to see the **helloredis** application. Check both containers to make sure they have the "production" label.
//-->
7. UCP ダッシュボードの **Applications** をクリックして **helloredis** アプリケーションを確認します。両方のコンテナが「production」ラベルを持っていることを確認します。

<!--
8. Click the **helloredis** application and drill into each container to make sure it has the **com.docker.ucp.access.label: "production"** label.
//-->
8. **helloredis** アプリケーションをクリックし、各コンテナが **com.docker.ucp.access.label: "production"** ラベルを持っていることを確認します。

<!--
9. Click the **Console** tab on one of the containers and run a bash shell to confirm that you can get terminal access.
//-->
9. コンテナのどれか 1つの **Console** タブをクリックし、bash シェルを起動してターミナルアクセスができることを確認します。


<!--
## Step 3 - Check access
//-->
## 手順 3 - アクセスの確認

<!--
1. Login to UCP as **johnfull** from the **Engineering** team.
//-->
1. UCP に **Engineering** チームの **johnfull** ユーザでログインします。

<!--
2. Click on the **Applications** link in the left pane and verify that you can see the application.
//-->
2. 左ペインの **Applications** をクリックし、アプリケーションを確認します。

<!--
3. Drill into one of the containers and try and open a bash terminal from the **Console** tab.
//-->
3. コンテナのどれか 1つの **Console** タブをクリックし、bash ターミナルを開きます。

<!--
  This operation will fail. This is because members of the **Engineering** team only have "View Only" access to resources tagged with the **production** label.
//-->
  この処理は失敗するはずです。なぜなら **Engineering** チームのメンバーは、 **production** ラベルをつけられたリソースに「View Only」アクセスしかできないからです。
