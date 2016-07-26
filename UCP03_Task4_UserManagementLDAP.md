<!--
# Task 4 - User Management with LDAP
//-->
# タスク 4 - LDAP でのユーザ管理

<!--
## Pre-requisites
//-->
## 前提条件
<!--
- Pre-configured LDAP server with users and groups setup. Your instructor will provide you with details.
- Basic understanding of LDAP structure
//-->

- ユーザとグループを事前に設定した LDAP サーバがあること。詳細は講師が提供します。
- LDAP 構造の基本的な理解。

<!--
## Step 1 - Examine the LDAP server
//-->
## 手順 1 - LDAP サーバの確認

<!--
1. Open a browser and go to http://LDAP_SERVER/phpldapadmin. LDAP_SERVER should be the domain name or public IP address of your LDAP node. This will open up the GUI for your LDAP server
2. Login to the server with the following credentials:
//-->
1. ブラウザで http://LDAP_SERVER/phpldapadmin を開きます。 LDAP_SERVER は LDAP ノードのドメイン名かパブリック IP アドレスです。これは LDAP サーバの GUI です。
2. 次の情報でサーバにログインします。

   **Login DN:** cn=admin,dc=test,dc=com
   **Password:** admin
   
<!--
3. Expand the left navigation bar and take note of the LDAP entries underneath `ou=all users`, `ou=engineering` and `ou=HR`
//-->
3. ナビゲーションバーを開き、 `ou=all users` 、 `ou=engineering` 、 `ou=HR` の LDAP エントリがあることを確認します。

   ![](images/ucp03_t4_LDAP.PNG)
   
<!--
   In this setup, we have 6 users.
//-->
   ここでは 6 ユーザを設定しています。
   
<!--
## Step 2 - Integrate UCP and LDAP
//-->
## 手順 2 - UCP と LDAP の統合

<!--
1\. Login to UCP as the admin user
//-->
1. UCP に管理者ユーザでログインします。

<!--
2\. Go to the "Settings" page of UCP
//-->
2. UCP の「Settings」ページに移動します。

<!--
3\. Click on the Auth section and change the Method from "Managed" to "LDAP"
//-->
3. Auth セクションをクリックし、Method を「Managed」から「LDAP」に変更します。

<!--
4\. Specify your LDAP server URL. Use the format "ldap://<domain name to your LDAP node>"
//-->
4. LDAP サーバの URL を指定します。「ldap://<LDAP ノードのドメイン名>」という形式を使います。

<!--
5\. Select "Full Control" on the **Default Permission for Newly Discovered Accounts** field
//-->
5. **Default Permission for Newly Discovered Accounts** フィールドの「Full Control」を選択します。

<!--
6\. For the **LDAP Server Configuration** section, fill in the following details.
//-->
6. **LDAP Server Configuration** セクションに次を入力します。

| Field                     | Value            |
| :---------                | :-----------     |
| Recovery Admin Username   | admin            |
| Recovery Admin Password   | orca         |
| Reader DN                 | cn=Chuck Norris,ou=all users,dc=test,dc=com |
| Reader Password           | password  |
   
<!--
7\. For the **LDAP Security Options**, leave both options unchecked.
//-->
7. **LDAP Security Options** は両方ともチェックしないでおきます。

<!--
8\. For the **User Search Configurations** section, fill in the following details.
//-->
8. **User Search Configurations** セクションに次を入力します。

| Field                     | Value            |
| :---------                | :-----------     |
| Base DN                   | dc=test,dc=com   |
| Username Attribute        | uid |
| Full Name Attribute       | cn  |
| Filter                    | objectClass=inetOrgPerson |

<!--
8\. Tick the **Scope Subtree** checkbox
//-->
9. **Scope Subtree** チェックボックスをチェックします。

   ![](images/ucp03_t4_LDAP_user_search.PNG)
   
<!--
9\. Scroll down to the **Test LDAP Connection** section and specify the following:
//-->
10. **Test LDAP Connection** セクションにスクロールダウンし、次を指定します:

| Field                     | Value            |
| :---------                | :-----------     |
| LDAP Test Username        | cnorris          |
| LDAP Test Password        | password         |

<!--
10\. Click the **Test** button and verify that you get a **Success** message
//-->
11. **Test** ボタンをクリックし、 **Success** メッセージが出ることを確認します。

<!--
11\. Click **Update Auth Settings**
//-->
12. **Update Auth Settings** をクリックします。
   
<!--
## Step 3 - Test User Access
//-->
## 手順 3 - ユーザのアクセステスト

<!--
1. In the **LDAP Sync Status** section, click on the **Sync Now** button.
//-->
1. **LDAP Sync Status** セクションの **Sync Now** ボタンをクリックします。

<!--
2. Click on **User and Teams** and check to see that all the users from our LDAP server are present. You will also notice that
   the previous users you created are still present.
//-->
2. **User and Teams** をクリックし、LDAP サーバに存在するユーザがすべて見えることを確認します。また、先に作成したユーザも一緒に見えることを確認します。
   
<!--
3. Logout of UCP as the admin user. Try and login as user `johnfull`. What do you notice?
//-->
3. UCP から管理者ユーザをログアウトします。 `johnfull` でログインし直します。何が起きましたか？

<!--
   Once UCP is integrated with LDAP, all user accounts will come from LDAP. **Managed** accounts that were previously setup in UCP will be disabled, accept for the account that your specified in the **Recovery Admin Username** 
   configuration field. 
//-->
   一旦 UCP と LDAP を統合すると、すべてのユーザアカウントは LDAP 由来のものとなります。先に UCP で設定した **Managed** アカウントは無効になります。**Recovery Admin Username** 設定フィールドで指定したアカウントは受け付けます。
   
<!--
4. Login as the user `cnorris`. The password is `password`
//-->
4. ユーザ `cnorris` でログインします。パスワードは `password` です。

<!--
5. Logout as `cnorris` and log back in as `admin`. Disable the LDAP integration by switching the Authentication back to **Managed**
//-->
5. `cnorris` をログアウトして `admin` でログインし直します。Authentication を **Managed** に戻して LDAP 統合を無効にします。


   






   
