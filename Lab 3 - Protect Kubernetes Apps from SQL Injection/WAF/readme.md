### App Protect Logs ###
```
apiVersion: appprotect.f5.com/v1beta1
kind: APLogConf
metadata:
  name: logconf
spec:
  filter:
    request_type: all
  content:
    format: default
    max_request_size: any
    max_message_size: 5k
```
### Enforced Signature Policy ###
```
apiVersion: appprotect.f5.com/v1beta1
kind: APPolicy
metadata:
  name: sql-injetion-blocking
spec:
  policy:
    name: sql_injetion_blocking
    template:
      name: POLICY_TEMPLATE_NGINX_BASE
    applicationLanguage: utf-8
    enforcementMode: blocking
    signature-sets:
    - name: SQL Injection Signatures
      alarm: true
      block: true
    blocking-settings:
      violations:
      - name: VIOL_PARAMETER
        alarm: true
        block: true
      - name: VIOL_THREAT_CAMPAIGN
        alarm: true
        block: true
```
### Stagging some Signature ID ###
```
apiVersion: appprotect.f5.com/v1beta1
kind: APPolicy
metadata:
  name: sql-injetion-blocking
spec:
  policy:
    name: sql_injetion_blocking
    template:
      name: POLICY_TEMPLATE_NGINX_BASE
    applicationLanguage: utf-8
    enforcementMode: blocking
    signature-sets:
    - name: SQL Injection Signatures
      alarm: true
      block: true
    blocking-settings:
      violations:
      - name: VIOL_PARAMETER
        alarm: true
        block: true
      - name: VIOL_THREAT_CAMPAIGN
        alarm: true
        block: true
    signatures:
    - enabled: false
      signatureId: 200000099
    - enabled: false
      signatureId: 200000093
    - enabled: false
      signatureId: 200001087
```
### App Protect Logs ###
```
apiVersion: appprotect.f5.com/v1beta1
kind: APLogConf
metadata:
  name: logconf
spec:
  filter:
    request_type: all
  content:
    format: default
    max_request_size: any
    max_message_size: 5k
```
### Ingress with WAF ###
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: vul-ingress
  annotations:
    appprotect.f5.com/app-protect-policy: "default/sql-injetion-blocking"
    appprotect.f5.com/app-protect-enable: "True"
    appprotect.f5.com/app-protect-security-log-enable: "True"
    appprotect.f5.com/app-protect-security-log: "default/logconf"
    appprotect.f5.com/app-protect-security-log-destination: "syslog:server=syslog-svc.default:514"
spec:
  ingressClassName: nginx-web
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app
            port:
              number: 80
```
#### Policy for the VirtualServer ###
```
apiVersion: k8s.nginx.org/v1
kind: Policy
metadata:
  name: sql-injetion-blocking
spec:
  waf:
    enable: true
    apPolicy: "default/sql-injetion-blocking"
    securityLog:
      enable: true
      apLogConf: "default/logconf"
      logDest: "syslog:server=syslog-svc.default:514"
```

### VirtualServer with WAF ###
```
apiVersion: k8s.nginx.org/v1
kind: VirtualServer
metadata:
  name: vul-vs
spec:
  ingressClassName: nginx-web
  host: example.com
  policies:
  - name: sql-injetion-blocking
  upstreams:
    - name: app
      service: app
      port: 80
  routes:
    - path: /
      action:
        pass: app
```


