## Struct  
- server, workflow-controller, postgress, minio(ストレージミドルウェア)の4つのpodで構成されている  
- serverの中にUIを含む, imageはargo-cli  
- 同一port(2746)の `/api` に開いているrestAPIでsubmitなどを行なっている
  - ref https://github.com/argoproj/argo/blob/master/server/apiserver/argoserver.go#L264-L270  
- ref server source https://github.com/argoproj/argo/tree/master/cmd/argo  
- ref manifests https://raw.githubusercontent.com/argoproj/argo/stable/manifests/quick-start-postgres.yaml  
## UI  
- manifestにキーが足りていないとinternalServerErrrorになる(検証したのはWorkflowTemplate)  
- templateからworkflow作成時にtemplate側のentrypointが参照されない  
  - entry point追加 https://github.com/argoproj/argo/blob/master/ui/src/models/workflow-templates.ts
  - https://github.com/argoproj/argo/blob/master/ui/src/app/workflow-templates/components/workflow-template-details/workflow-template-details.tsx#L138

## Integrations  
### metric  

### datadog  
- datadog側にintegrationあり、eventもcollectできそう

## CustomResource  
- apiはどこだろう?  
  - このへんを参照できそう？  
  - https://github.com/argoproj/argo/blob/master/pkg/apis/workflow/v1alpha1/workflow_types.go#L167  
- docsにありました
  - https://argoproj.github.io/argo/fields/
- default値
  - https://argoproj.github.io/argo/default-workflow-specs/
## stop/terminateは何をしているか調査  
- ~docker-for-macだとterminate機能しない可能性あり?~
  - stop http://localhost:2746/api/v1/workflows/argo/workflow-template-submittable-mt6hx/stop
    - https://github.com/argoproj/argo/blob/1b757ea9bc75a379262928be76a4179ea75aa658/workflow/util/util.go#L721
  - terminate http://localhost:2746/api/v1/workflows/argo/workflow-template-stoppable-kkb6v/terminate
    - https://github.com/argoproj/argo/blob/1b757ea9bc75a379262928be76a4179ea75aa658/workflow/util/util.go#L747
- ここでpod deleteしてる
  - https://github.com/argoproj/argo/blob/d07a0e74888254fe78fa653e532c5f1963056598/workflow/controller/exec_control.go#L35
- operate called here
  - https://github.com/argoproj/argo/blob/13b1d3c19e94047ae97a071e4468b1050b8e292b/workflow/controller/controller.go#L508
- Pod statusがPending(pod数制限などで)にならない限り停止不可
  - Podがrunningの場合はpod自体をdeleteして停止させる必要があるっぽい
- stepsだとterminateになった！dagだとpendingを挟まないからなのかダメっぽい
- どのみちpod終了単位でしか終了できないことを注意

## fail時の挙動
- retryはstepsじゃないと使えなさそう

## notice
### exit notice
- k8s eventとしては上がっているのでそちらを集計 or onExitHookを利用する

## SSO
- https://argoproj.github.io/argo/argo-server-sso/
- configもろもろ設定、実環境ないと試しづらいのでAzureADで試したい

## TODO my argo manifests
