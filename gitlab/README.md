# GitLab

## Runnerの登録

うまく登録できない場合は、トークンをリセットすると成功することがある。

```bash
docker exec -it gitlab_runner_1 bash
# runnerコンテナ内で実行する
gitlab-runner register
# GitLab instance URLは http://web/ とする
# tokenは GitLabのUIで確認した値とする
# executorは shell とする
```

## Pipeline

レポジトリ内に`.gitlab-ci.yml`というファイル名でCI定義を作成する

```yaml
stages:
  - build
  - exec

build:
  stage: build
  script:
    - mvn clean package
  artifacts:
    paths:
      - target/*.jar

exec:
  stage: exec
  script:
    - java -cp target/hello-1.0-SNAPSHOT.jar com.example.Main
```