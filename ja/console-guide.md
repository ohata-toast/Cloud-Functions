## Compute > Cloud Functions > コンソール使用ガイド
この文書では、NHN Cloud Functionsコンソールで関数を作成し、管理する方法について説明します。

## 関数管理
関数を作成、修正、削除、コピーできます。

### 関数作成
関数設定を行い、コードを作成した後作成ボタンをクリックすると、コードがビルドされ、関数が作成されます。

![console-guide-07](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/console-guide-jp-01.png)
![console-guide-08](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/console-guide-jp-02.png)

#### 関数設定
<table class="it">
    <tr>
        <th>分類</th>
        <th>No.</th>
        <th>項目</th>
        <th>説明</th>
    </tr>
    <tr>
        <td rowspan="2">基本情報</td>
        <td>1.</td>
        <td>名前</td>
        <td>関数の名前<br> 重複した関数名は許可されません。<br> 関数エンドポイントURLとして使用されます。</td>
    </tr>
    <tr>
        <td>2.</td>
        <td>説明</td>
        <td>関数の説明、最大250文字</td>
    </tr>
    <tr>
        <td rowspan="7">関数設定</td>
        <td>3.</td>
        <td>エンドポイントURL</td>
        <td>関数呼び出しHTTPエンドポイントURL<br>関数名を入力すると自動的に変更されます。</td>
    </tr>
    <tr>
        <td>4.</td>
        <td>タイプ</td>
        <td>Pool Manager<br> New Deployment</td>
    </tr>
    <tr>
        <td>5.</td>
        <td>リソース</td>
        <td>関数動作環境リソース選択<br>Pool Managerタイプの場合、指定されたリソースのみ選択可能<br>New Deploymentタイプの場合、カスタムリソース入力可能</td>
    </tr>
    <tr>
        <td>6.</td>
        <td>実行制限時間</td>
        <td>関数実行時間を指定(Time out)。関数が実行され、指定された時間を超過すると関数は強制的に中断され、エラーでレスポンス</td>
    </tr>
    <tr>
        <td>7.</td>
        <td>(Pool Manager)同時実行設定</td>
        <td>関数の同時実行数を指定。同時に関数呼び出しが発生した場合、呼び出し回数分のインスタンスが同時に実行され、最大インスタンス数は制限されます。</td>
    </tr>
    <tr>
        <td>8.</td>
        <td>(New Deployment)最小インスタンス数</td>
        <td>維持する最小インスタンス数を指定。</td>
    </tr>
    <tr>
        <td>9.</td>
        <td>(New Deployment)最大インスタンス数</td>
        <td>自動的に拡張可能な最大インスタンス数指定。</td>
    </tr>
    <tr>
        <td>ログ設定</td>
        <td>10.</td>
        <td>ログサービス連動</td>
        <td>Log & Crash Searchサービスを使用して連携するかどうかを選択<br>関数のログはLog & Crash Searchサービスで直接確認可能です。<br>ログは10ライン単位で転送されます。</td>
    </tr>
</table>

> **[参考]** <br>
> **(Pool Manager)同時実行設定**


![console-guide-05](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-07-29/console-guide-05.png)

<br>

> **[参考]** <br>
> **(New Deployment)インスタンス数** - リソース使用量に応じて、指定されたインスタンス数だけ生成されない場合があります。

<br>


#### コード作成

![console-guide-09](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/console-guide-jp-03.png)
![console-guide-10](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/console-guide-jp-04.png)
![console-guide-11](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/console-guide-jp-05.png)

<table class="it">
    <tr>
        <th>分類</th>
        <th>No.</th>
        <th>項目</th>
        <th>説明</th>
    </tr>
    <tr>
        <td rowspan="2">ソースコード</td>
        <td>1.</td>
        <td>ランタイム環境</td>
        <td>関数のランタイム環境を選択</td>
    </tr>
    <tr>
        <td>2.</td>
        <td>Entry Point</td>
        <td>関数のエントリーポイントを指定。例：関数名 <br>ランタイム環境のテンプレートに基づいて自動的に補完されます。<br>任意に修正する場合は、作成したソースコードのエントリーポイントと一致している必要があります。<br>誤って入力した場合、**デプロイでは確認できず**、**テスト実行時のログを通じて確認できます**。</td>
    </tr>
    <tr>
        <td rowspan="3">コード</td>
        <td>3.</td>
        <td>コードエディタ</td>
        <td>ランタイム環境のテンプレートファイルが読み込まれ、そのファイルを編集して関数を作成<br>コードエディタでのディレクトリ追加はできません。ディレクトリ構造を編集するには、テンプレートファイルをダウンロードしてローカルで直接修正した後、ZIPファイルアップロード方式を使用する必要があります。</td>
    </tr>
    <tr>
        <td>4.</td>
        <td>ユーザーローカル環境</td>
        <td>ユーザーローカル環境で関数コードを作成し、ZIPファイル形式でアップロード</td>
    </tr>
    <tr>
        <td>5.</td>
        <td>デプロイ</td>
        <td>ユーザーが作成またはアップロードしたコードを配布し、ビルドが正常かどうかを確認するために使用されます。<br>参考：- 関数を作成する際には、該当のデプロイバージョンは使用されず、作成時点で最新の更新済みソースコードで再ビルドされます。</td>
    </tr>
    <tr>
        <td rowspan="2">テスト</td>
        <td>6.</td>
        <td>テストイベント</td>
        <td>関数に渡すJSON Bodyを作成</td>
    </tr>
    <tr>
        <td>7.</td>
        <td>テスト</td>
        <td>イベントに作成したJSON Bodyを渡して関数をテストします(事前にデプロイが必要です)。ログで関数の動作を確認できます。<br>GETメソッドで呼び出します。</td>
    </tr>
    <tr>
        <td></td>
        <td>8.</td>
        <td>作成</td>
        <td>作成ボタンを利用して、関数設定と作成した関数に合わせて関数を作成</td>
    </tr>
</table>

<br>

> **[参考]** <br>配布ボタンは、ユーザーが作成したソースコードのビルド確認専用であり、関数作成時に配布でテストしたビルドではなく、最新のソースコードで再ビルドされて生成されます。

### 関数修正
既存の関数の設定とコードを修正するために、修正ボタンを使用して関数を修正します。
#### 修正不可項目
- 名前、ランタイム環境
    - 該当項目を除く全ての項目を修正できます。
#### ソースコード
- コードエディタを使用する場合既存コードが読み込まれます。
- ユーザーローカル環境のZIPファイルをアップロードして関数を生成した場合、コードエディタに変更するとZIPファイルを表示せず、基本テンプレートコードが読み込まれます。

### 関数削除
![console-guide-14](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/console-guide-jp-06.png)
既存の関数を選択して削除します。一度に複数の関数を削除可能です。

### 関数コピー
![console-guide-13](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/console-guide-jp-07.png)
既存の関数と同一の関数をコピーします。名前は重複不可のため、コピー前に新しい名前を指定できます。
- トリガーはコピーされません。(HTTPトリガーはデフォルトで提供)

## 関数情報
### 関数リスト
![console-guide-01](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/console-guide-jp-08.png)
- ユーザーが作成した関数リストを確認できます。
- ビルドの状態は自動的に更新され,関数が使用可能かどうか確認できます。
### 関数基本情報
![console-guide-02](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/console-guide-jp-09.png)
- 関数の基本情報を確認できます。
- ログ管理項目からLog & Crash Searchボタンをクリックして、Log & Crash Searchサービスに移動してログを確認できます。
- ビルド状態項目からビルドログ確認ボタンをクリックして,ビルドログを確認できます。
- ユーザーが作成したコードをZIPファイル形式でダウンロードできます。
### 関数トリガー管理
- 関数を実行できるトリガーを管理できます。
- HTTPトリガーは関数の作成時にデフォルトで提供されます。
    - 有効化/無効化により、使用するかどうかを設定できます。
    
![console-guid-12](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/console-guide-jp-10.png)

- 指定されたHTTPトリガーを使用して作成した関数を実行できます。
    - 例: `https://{userdomain}/{関数名}`
    - Method : GET, POST

#### トリガー作成/修正
- Timer
    - Value: Cron文字列で周期を入力します。
- API Gateway
    - API Gatewayサービスを利用してHTTPエンドポイントを追加できます。
        
#### トリガー削除
- 複数のトリガーを選択して削除できます。デフォルトのトリガーであるHTTPトリガーは削除できません。

### 関数モニタリング
![console-guide-06](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/console-guide-jp-11.png)
- 関数の使用量を確認できます。
- 関数呼び出し回数、呼び出し拒否数、エラー発生回数、成功率、関数実行時間指標を提供します。
- 設定された時間内の指標を提供します。
- ログが連携されている場合、Log & Crash Searchボタンを利用してLog & Crash Searchサービスに移動できます。

| 項目 | 説明 |
| --- | --- |
| 関数呼び出し回数 | 関数の総呼び出し回数(1秒あたり) |
| 呼び出し拒否数 | インスタンスが作成されずに関数呼び出しに失敗した回数（1秒あたり） |
| エラー発生回数 | レスポンスコードが200でないレスポンス回数(1秒あたり) |
| 成功率 | 関数の総呼び出し回数に対する成功した呼び出しの割合 |
| 関数実行時間 | 関数呼び出しに対するレスポンス時間 |

> **[参考]**
> <br>呼び出し拒否数は、Pool Managerタイプの場合のみ該当します。
> <br>関数呼び出し回数、呼び出し拒否回数、エラー発生回数は、指標のステップ範囲内における1秒あたりの平均回数として表示されます。例えば、ステップが15秒で、その期間に1件発生した場合、0.0667 (1 ÷ 15)と表示されます。
> <br>関数実行時間の平均は、全体の累積平均です。最大値は、該当期間中における最大の実行時間です。
