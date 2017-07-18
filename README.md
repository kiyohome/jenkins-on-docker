# jenkins-on-docker

jenkins with sample pipeline on docker

## Tools

- Jenkins
- GitBucket

## Requirements

- SonarQube
- Rocket.Chat

## How to install

```
$ git clone https://github.com/kiyohome/jenkins-on-docker.git
$ cd jenkins-on-docker
$ docker-compose up -d
```

### Install sshpass to run ssh with password at deployment

```
$ docker-compose exec --user root jenkins bash -c "apt-get update -y && apt-get -y install sshpass"
```

## How to configure

### Jenkins

- Access "http://localhost/jenkins" in the browser
- Unlock Jenkins
  - Get administrator password
```
  $ docker-compose exec jenkins sh -c "cat /var/jenkins_home/secrets/initialAdminPassword"
```
- Configure proxy, if neccesary
  - no_proxy: gitbucket,proxy
- Install suggested plugins
  - Retry, if installation failures
- Create First Admin User
  - username: admin
  - password: admin
- Jenkinsの管理 > Global Tool Configuration
  - JDK
    - name: jdk8
  - Maven
    - name: mvn3
  - Docker

### GitBucket

- Access "http://localhost/gitbucket" in the browser
- Sign in
  - Username: root
  - Password: root
- System administration > System settings
  - Base URL: http://localhost/gitbucket
- System administration > New user
  - Name: your name
- Sign in by your name
- New group
  - Group name: sample
- New repository
  - Owner: sample
  - Repository name: nablarch-example-web
- Run the following commands on your local machine
```
  $ git clone https://github.com/nablarch/nablarch-example-web.git
  $ cd nablarch-example-web
  $ git remote rm origin
  $ git remote add origin http://localhost/gitbucket/git/sample/nablarch-example-web.git
  $ git push -u origin master
  $ git checkout -b develop
  $ git push origin develop
```

### Multibranch Pipeline

- 新規ジョブ作成
  - item name: nablarch-example-web
  - Multibranch Pipeline: ON
  - Branch Sources > Add source > Git
    - プロジェクトリポジトリ: http://proxy/gitbucket/git/sample/nablarch-example-web.git
  - Scan Multibranch Pipeline Triggers
    - 他のビルドが起動していなければ定期的に起動: ON

### Notify to Rocket.Chat

- Add channel/user for jenkins on Rocket.Chat
- Install RocketChat Notifier plugin
- Jenkinsの管理 > システムの設定 > Global RocketChat Notifier Settings
```
    stage('Unit test') {
      steps {
        echo 'Unit test'
        sh 'mvn -P gsp generate-resources'
        sh 'mvn test'
        junit 'target/surefire-reports/**/*.xml'
      }
      post {
        success { rocketSend message: 'Unit test', emoji: ':blush:' }
        failure { rocketSend message: 'Unit test', emoji: ':sob:' }
      }
    }
```