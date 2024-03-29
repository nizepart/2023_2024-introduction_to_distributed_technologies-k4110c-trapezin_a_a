University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)  
Year: 2023/2024  
Group: K4110c  
Author: Trapezin Andrey  
Lab: Lab2  
Date of create: 28.09.2023  
Date of finished: 31.09.2023
---
### Выполнение лабораторной работы

Создан новый шаблон helm с именем react-app.
```bash
➜  lab2 git:(main) ✗ helm create react-app
```
Отредактирован values.yaml. Указаны значения для образа, скейлинга, сервиса и переменных.

values.yaml
```yaml
image:
  repository: ifilyaninitmo/itdt-contained-frontend
  tag: "master"
configmaps:
  app-env:
    REACT_APP_USERNAME: "nizepart"
    REACT_APP_COMPANY_NAME: "ITMO"
#  another_configmap:
#    complex_key: | # multiple lines value
#      This is a complex value
#      spanning multiple lines
#  yet_another_configmap:
#    nested_object:
#      key1: value1
#      key2: value2
autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 2
  targetCPUUtilizationPercentage: 80
service:
  type: ClusterIP
  port: 3000
```

Написан шаблон для создания configmap, который будет принимать на вход map, список key-value или key-многострочное знечение.

configmap.yaml
```yaml
{{- range $name, $configmap := .Values.configmaps}}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ $name }}
  labels:
    {{- include "react-app.labels" $ | nindent 4 }}
data:
{{- range $key, $value := $configmap }}
  {{- if eq (kindOf $value) "string" }}
  {{ $key }}: {{ if contains "\n" $value }}|{{ $value | nindent 4 }}
    {{ else }}{{ $value | quote }}{{ end }}
  {{- else if eq (kindOf $value) "map" }}
  {{ $key }}:
{{- range $innerKey, $innerValue := $value }}
    {{ $innerKey }}: {{ $innerValue | quote }}
{{- end }}
  {{- end }}
{{- end }}
---
{{- end }}
```

В Deployment указан какой configmap будет использоваться для дополнения переменных в контейнере.

deployment.yaml
```yaml
...
      containers:
        - name: {{ .Chart.Name }}
          securityContext:
            {{- toYaml .Values.securityContext | nindent 12 }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          envFrom:
            {{- range $configmap_name, $data := .Values.configmaps }}
            - configMapRef:
                name: {{ $configmap_name }}
            {{- end }}
...
```

Установлены компоненты k8s в окружение labs.
```bash
➜  lab2 git:(main) ✗ helm install react-app -n labs react-app
```

Получен список подов в окружении labs.
```bash
➜  lab2 git:(main) ✗ k -n labs get po
NAME                         READY   STATUS    RESTARTS   AGE
react-app-574799657d-5nkl4   1/1     Running   0          33s
react-app-574799657d-sbzpx   1/1     Running   0          33s
vault-7f9b5f988c-2s5tk       1/1     Running   0          69m
```

Проверено наличие переменных непосредственно внутри контейнера. Выбор контейнера для проверки не повлияет на результат.
```bash
➜  lab2 git:(main) ✗ k -n labs exec -ti pods/react-app-574799657d-2ccbp -- sh
/frontend # echo $REACT_APP_USERNAME
nizepart
/frontend # echo $REACT_APP_COMPANY_NAME
ITMO
```

Локальный порт 3000 прокинут в контейнер.
```bash
➜  lab2 git:(main) ✗ k -n labs port-forward services/react-app 3000:3000
Forwarding from 127.0.0.1:3000 -> 3000
Forwarding from [::1]:3000 -> 3000
```

Проверены заданные переменные на странице в веб браузере.
![react_app_web.png](screenshots%2Freact_app_web.png)

Просмотрены логи контейнера.
```bash
➜  lab2 git:(main) ✗ k -n labs logs react-app-574799657d-5nkl4
Builing frontend
Browserslist: caniuse-lite is outdated. Please run:
  npx update-browserslist-db@latest
  Why you should do it regularly: https://github.com/browserslist/update-db#readme
Browserslist: caniuse-lite is outdated. Please run:
  npx update-browserslist-db@latest
  Why you should do it regularly: https://github.com/browserslist/update-db#readme
build finished
Server started on port 3000
```

### Схема организации контейнеров и сервисов 
![lab2.drawio.svg](lab2.drawio.svg)