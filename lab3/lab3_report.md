University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)  
Year: 2023/2024  
Group: K4110c  
Author: Trapezin Andrey  
Lab: Lab3  
Date of create: 29.09.2023  
Date of finished: 31.09.2023
---

```bash
openssl req -new -newkey rsa:2048 -nodes \
  -subj "/C=GB/ST=England/L=Brighton/O=Example/CN=*.mydomainitmo.com/emailAddress=info@example.com" \
  -reqexts SAN \
  -config <(cat /etc/ssl/openssl.cnf \
        <(printf "\n[SAN]\nsubjectAltName=DNS:mydomainitmo.com,DNS:www.mydomainitmo.com")) \
  -keyout mydomainitmo.key \
  -x509 -days 3650 -extensions SAN -out mydomainitmo.crt
```

```bash
openssl x509 -in /tmp/example.com.crt -noout -text
```


```bash
➜  lab3 git:(main) ✗ k create secret tls react-app-tls --cert=server.crt --key=private.key -n labs
secret/react-app-tls created
```

values.yaml
```yaml
ingress:
  enabled: true
  className: "nginx"
  hosts:
    - host: mydomainitmo.com
      paths:
        - path: /
          pathType: ImplementationSpecific
          backend:
            service:
              name: react-app
              port:
                number: 3000
  tls:
    - secretName:  react-app-tls
      hosts:
        - mydomainitmo.com
```

```bash
➜  ~ minikube tunnel
✅  Tunnel successfully started

📌  NOTE: Please do not close this terminal as this process must stay alive for the tunnel to be accessible ...

❗  The service/ingress react-app requires privileged ports to be exposed: [80 443]
🔑  sudo permission will be asked for it.
🏃  Starting tunnel for service react-app.
```

/etc/hosts
```bash
127.0.0.1    mydomainitmo.com
```

![app_web.png](screenshots%2Fapp_web.png)

![crt.png](screenshots%2Fcrt.png)
