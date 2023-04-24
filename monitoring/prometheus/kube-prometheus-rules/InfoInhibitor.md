### https://runbooks.prometheus-operator.dev/runbooks/general/infoinhibitor/
> This is an alert that is used to inhibit info alerts.
> 
> 
> By themselves, the info-level alerts are sometimes very noisy, but they are relevant when combined with other alerts.

info alert들을 inhibit하기 위한 alert이며, info 레벨 alert는 짜증나지만 다른 alert와 함께 볼 경우 관련성이 있을 수 있다고 한다.
의역하자면, info alert가 너무 많이 울리지 않도록 하기 위한 alert라는 뜻(뒷말은 그냥 info alert도 정보가 있는 alert라는 당연한 말을 하고 싶은 듯)

kube-prometheus-stack 특정 버전에서, 템플릿에 Inhibit rules가 생긴 것을 확인함 (InfoInhibitor 자체는 Alertmanager의 특정 버전에 추가됐지만, kube-prometheus-stack에 추가된건 kube-prometheus-stack에 명시된 AlertManager의 버전과 관계 없이 나중에 추가된 것 같음)

### 트리거 시점
그냥 info level의 alert가 발생하는 시점(라고 chatGPT가 말했다. 레퍼런스 문서는 정확히 찾지 못함)


### 관련 템플릿
```yaml
config:
    global:
      resolve_timeout: 5m
    inhibit_rules:
      - source_matchers:
          - 'severity = critical'
        target_matchers:
          - 'severity =~ warning|info'
        equal:
          - 'namespace'
          - 'alertname'
      - source_matchers:
          - 'severity = warning'
        target_matchers:
          - 'severity = info'
        equal:
          - 'namespace'
          - 'alertname'
      - source_matchers:
          - 'alertname = InfoInhibitor'
        target_matchers:
          - 'severity = info'
        equal:
          - 'namespace'
    route:
      group_by: ['namespace']
      group_wait: 30s
      group_interval: 5m
      repeat_interval: 12h
      receiver: 'null'
      routes:
      - receiver: 'null'
        matchers:
          - alertname =~ "InfoInhibitor|Watchdog"
    receivers:
    - name: 'null'
    templates:
    - '/etc/alertmanager/config/*.tmpl'
```

해석을 해보자면, `source_matchers` alert가 발생하는 경우에 namespace or alertname이 일치하는 `target_matchers` alert를 무시한다는 뜻이다

### Info alert를 무시하기 위해 InfoInhibitor alert를 울린다?

언뜻 봐서는 정말 이상한 문장이다.

Info alert가 annoying해서 무시하고 싶은데, 이를 위해 Inhibit rule을 설정해 InfoInhibitor가 억제하도록 해도

결국 InfoInhibitor alert는 울리기 때문이다 (InfoInhibitor alert를 안울리게 하려니, 또 새로운 level의 alert를 만들어서 InfoInhibitor 대신 울리게 해야하고,,,,,,,, 반복)

그런데 중요한 것은, InfoInhibitor가 한번 트리거된 이상 해당 alert가 끝날 때까지 모든 info level alert가 무시된다는 점이다!

그래서 특정 event로 인해 발생할 수 있는 여러 개의 alert들(심지어 여러 개의 event로 인한 무지막지하게 많은 alert들)을 InfoInhibitor 하나의 alert로 대체할 수 있다!!

### 왜 그럼에도 불구하고 annoying할까

info 레벨의 alert는 굉장히 많고, 그래서 시도떄도없이 InfoInhibitor alert가 트리거된다..

다시 말해 기존에 info alert가 울리던 때에는 InfoInhibitor가 추가된 때보다 몇배는 더 많았을 거라는 뜻이다.

그럼에도 불구하고 annyoing하다고 느껴지다면, InfoInhibitor alert가 active 한 상태를 유지하는 period를 조정하면 된다 (default 1시간이라고 한다)