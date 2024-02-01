# Voiceboxを使った音声合成　ずんだもん

## 実行手順


1. Dockerfileからイメージを生成してECRに登録する

```
docker build -t zunda .

aws ecr get-login-password --region ap-northeast-1 | docker login --username AWS --password-stdin 241912342838.dkr.ecr.ap-northeast-1.amazonaws.com

docker tag zunda:latest 241912342838.dkr.ecr.ap-northeast-1.amazonaws.com/zundamon:latest

docker push 241912342838.dkr.ecr.ap-northeast-1.amazonaws.com/zundamon:latest

```

2. ECSでタスク定義を行う
3. ECSでクラスター定義を行う
4. ECSでサービス定義を行う
5. ECSのタスク定義の詳細にアドレスがある
6. 利用しているセキュリティグループに50021を加える
7. 


AWS Fargateを使用してDockerコンテナを実行する手順は、大まかに以下のステップに分けられます。まずは、DockerイメージをビルドしてAmazon Elastic Container Registry (ECR)にプッシュするところから始めます。

### ステップ1: DockerイメージのビルドとECRへのプッシュ

1. **Dockerfileの準備**:
   - アプリケーションのDockerfileを準備します。これは、アプリケーションの実行環境を定義したファイルです。

2. **ECRリポジトリの作成**:
   - AWS管理コンソールにログインし、「Elastic Container Registry」に移動します。
   - 「リポジトリを作成」ボタンをクリックし、新しいリポジトリの名前を指定してリポジトリを作成します。

3. **Dockerイメージのビルド**:
   - ローカル環境でDockerfileがあるディレクトリに移動し、以下のコマンドを使用してDockerイメージをビルドします。
     ```
     docker build -t <イメージ名> .
     ```

4. **AWS CLIの設定**:
   - AWS CLIがまだ設定されていない場合は、`aws configure`コマンドを使用して設定します。

5. **ECRへのログイン**:
   - AWS CLIを使用してECRにログインします。以下のコマンドを実行し、出力されたログインコマンドをコピーして実行します。
     ```
     aws ecr get-login-password --region <リージョン> | docker login --username AWS --password-stdin <AWSアカウントID>.dkr.ecr.<リージョン>.amazonaws.com
     ```

6. **イメージのタグ付け**:
   - ビルドしたDockerイメージにタグを付けます。これにより、イメージをECRリポジトリに関連付けます。
     ```
     docker tag <イメージ名>:<タグ> <AWSアカウントID>.dkr.ecr.<リージョン>.amazonaws.com/<リポジトリ名>:<タグ>
     ```

7. **イメージのプッシュ**:
   - タグ付けしたイメージをECRにプッシュします。
     ```
     docker push <AWSアカウントID>.dkr.ecr.<リージョン>.amazonaws.com/<リポジトリ名>:<タグ>
     ```

### ステップ2: ECSタスク定義の作成

1. **ECSコンソールに移動**:
   - AWS管理コンソールで「Elastic Container Service」を選択します。

2. **新しいタスク定義の作成**:
   - 「タスク定義」タブに移動し、「新しいタスク定義の作成」を選択します。
   - 「Fargate」を起動タイプとして選択し、「次のステップ」をクリックします。

3. **タスクとコンテナの設定**:
   - タスク定義の名前、タスクのロール、タスクサイズ（CPUとメモリ）を設定します。
   - 「コンテナの追加」をクリックし、コンテナ名、ECRからプッシュしたイメージのURI、必要なポートマッピングなどのコンテナ設定を入力します。

4. **タスク定義の登録**:
   - すべての設定を確認し、「タスク定義の作成」をクリックしてタスク定義を登録します。

### ステップ3: ECSサービスの作成とタスクの実行

1. **ECSクラスターの作成**:
   - 「クラスター」タブに移動し、「クラスターの作成」を選択します。
   - 「ネットワーキングのみ」のテンプレートを選択し、クラスター名を設定してクラスターを作成します。

2. **サービスの作成**:
   - 作成したクラスター内で「サービス」タブに移動し、「サービスの作成」を選択します。
   - 起動タイプに「Fargate」を選択し、先ほど作成したタスク定義を選択します。
   - サービス名を設定し、サービスタイプ（通常は「REPLICA」）とタスク数を設定します。

3. **ネットワーキングとセキュリティグループの設定**:
   - VPC、サブネット、セキュリティグループを選択し、タスクが使用するロードバランサー（必要な場合）を設定します。

4. **サービスの作成**:
   - すべての設定を確認し、「サービスの作成」をクリックしてサービスを作成します。これにより、指定した数のタスクがFargate上で起動します。

これで、AWS Fargateを使用してDockerコンテナをオンデマンドに実行する準備が整いました。サービスが正常に作成されると、指定したタスクが自動的に起動し、アプリケーションが実行されます。




echo -n "こんにちは、音声合成の世界へようこそ" >text.txt

curl -s -X POST "127.0.0.1:50021/audio_query?speaker=1" --get --data-urlencode text@text.txt \  > query.json
curl -s -X POST "54.238.46.250:50021/audio_query?speaker=1" --get --data-urlencode text@text.txt \  > query.json
curl -s -H "Content-Type: application/json" -X POST -d @query.json "54.238.46.250:50021/synthesis?speaker=1" > audio.wav

http://54.238.46.250:50021/docs