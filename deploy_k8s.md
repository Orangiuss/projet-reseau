# Fichier pour déployer une application sur Kubernetes

En reprenant le docker compose du TP2 sur le Docker Swarm et en le modifiant pour Kubernetes, nous obtenons le fichier suivant :

```yaml
apiVersion: v1
kind: Service
metadata:
  name: apache-service
spec:
  selector:
    app: apache
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: apache-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: apache
  template:
    metadata:
      labels:
        app: apache
    spec:
      containers:
        - name: apache
          image: {{ ansible_facts['default_ipv4']['address'] }}:5000/apache:v1
          ports:
            - containerPort: 80
          volumeMounts:
            - name: html-volume
              mountPath: /var/www/html
      volumes:
        - name: html-volume
          hostPath:
            path: /root/html

---

apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  selector:
    app: mysql
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: {{ ansible_facts['default_ipv4']['address'] }}:5000/mysql:latest
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: "root"
            - name: MYSQL_DATABASE
              value: "tp1"
            - name: MYSQL_USER
              value: "admintp1"
            - name: MYSQL_PASSWORD
              value: "admin"
          volumeMounts:
            - name: mysql-volume
              mountPath: /var/lib/mysql
            - name: setup-sql-volume
              mountPath: /docker-entrypoint-initdb.d/setup.sql
      volumes:
        - name: mysql-volume
          hostPath:
            path: /root/mysql
        - name: setup-sql-volume
          hostPath:
            path: /root/setup.sql
```
Nous avons des services et des deployments pour Apache et MySQL.