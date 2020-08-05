# circleci  

## 導入

[公式ページ](https://circleci.com/docs/ja/2.0/configuration-reference/)  
アカウントは以下のものから連携
- Github
- Bitbucket

対象のレポジトリーに `.circleci/config.yml` をホームディレクトリーに作成します  
[Add Project] -> [Set Up Project] -> [Use Existing Settings] から [Start Building]


## 通知先設定

### Slack

Webhook から通知できる  
[通知設定](https://circleci.com/docs/ja/2.0/notifications/)を参照する  
[Project Settings] -> [Slack Integration] から Slack の Webhook を設定する

[slack の Webhook 設定](https://slack.com/services/new/incoming-webhook)

 Web 通知の有効化を実施  
CircleCI のユーザー設定から権限を追加します  
