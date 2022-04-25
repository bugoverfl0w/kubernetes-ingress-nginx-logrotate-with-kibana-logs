# main purpuses
> rotate log for ingress nginx
> send ingress nginx logs to kibana via redis

# *Please do test on test environment first*

# install
> build filebeat image for rotate nginx log, then push to your private registry or dockerhub
```sh
cd filebeat-with-logrotate
docker build -t filebeat-log-rotate .
```

> create config map
```sh kubectl apply -f configmap.yaml ```

> update ingress-nginx-controller deployment
```plain update your filebeat-with-logrotate image on deployment-ingress-nginx-controller.yaml ```

- add to controller container
```plain
        volumeMounts:
        - mountPath: /usr/local/certificates/
          name: webhook-cert
          readOnly: true
        - mountPath: var/log/nginx/
```

- add new filebeat sidecar container
```plain
      - image: https://registry-gitlab/path/to/filebeat-with-logrotate
        imagePullPolicy: IfNotPresent
        name: filebeat-sidecar
        resources: {}
        securityContext:
          runAsUser: 0
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: var/log/nginx/
          name: nginx-logs
        - mountPath: /usr/share/filebeat/filebeat.yml
          name: filebeat-config
          subPath: filebeat.yml
```

- controller deployment config
```plain
      volumes:
      - name: webhook-cert
        secret:
          defaultMode: 420
          secretName: ingress-nginx-admission
      - emptyDir: {}
        name: nginx-logs
      - configMap:
          defaultMode: 420
          items:                                                                                                                       
          - key: filebeat.yml
            path: filebeat.yml
          name: ingress-configmap
        name: filebeat-config

```

Then apply your modified deployment ingress-nginx-controller