
### Les sondes : 
- La sonde **livenessProbe** 
- La sonde **readinessProbe**

**kubelet** utilise ces sondes pour superviser le Pod
- __livenessProbe__ : Pur rendre compte de la disponibilité du/des container(s) dans les Pods. **kubelet** fait un GET sur cette sonde, dans notre cas sur **/healthz** en HTTP:81 (TCP disponibile). Si le code de retour est compris entre 200 et 399 alors la sonde considère le système comme disponible. Pour tout autre code de retour kubelet commande un "kill" & "Restart" du contaianer
- __readinessProbe__ : Cette sonde renseigne sur la capacité d'un container du Pod a traiter les requêtes applicatves. En exposant sur la terminaison **/readiness** en HTTP:81 (TCP disponibile) un code entre 200 et 399, chaque container du Pod informe **kubelet** qu'il est __prêts__ à recevoir des requetes applicatives (via les services qui seront abordés plus tard) Pour tout autre code de retour, le trafic sera routé sur un autre container. 

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: auth
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: auth
        track: stable
    spec:
      containers:
        - name: auth
          image: "kelseyhightower/auth:2.0.0"
          ports:
            - name: http
              containerPort: 80
            - name: health
              containerPort: 81
          resources:
            limits:
              cpu: 0.2
              memory: "10Mi"
          livenessProbe:
            httpGet:
              path: /healthz
              port: 81
              scheme: HTTP
            initialDelaySeconds: 5
            periodSeconds: 15
            timeoutSeconds: 5
          readinessProbe:
            httpGet:
              path: /readiness
              port: 81
              scheme: HTTP
            initialDelaySeconds: 5
            timeoutSeconds: 1
```