### Deploy Microservices application in Kubernetes with Production & Security Best Practices

### Technologies used:

Kubernetes, Redis, Linux, Linode LKE

### Project Description:

1- Create K8s manifests for Deployments and Services for all microservices of an online shop application.

2- Deploy microservices to Linode’s managed Kubernetes cluster.

### Instruction:

###### Step 1 : Draw the maps of the microservices structure

![image](images/Screenshot%202023-05-13%20at%2010.12.53%20am.png)

###### Step 2: Highlight the ENV variables desired for application

![image](images/Screenshot%202023-05-13%20at%2010.13.04%20am.png)

###### Step 3: Check the images and port numbers

![image](images/Screenshot%202023-05-13%20at%2010.13.42%20am.png)

###### Step 4: Create and configure the microservices deployment and services configuration yaml

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: emailservice
spec:
  selector:
    matchLabels:
      app: emailservice
  template:
    metadata:
      labels:
        app: emailservice
    spec:
      containers:
        - name: server
          image: gcr.io/google-samples/microservices-demo/emailservice:v0.2.3
          ports:
            - containerPort: 8080
          env:
            - name: PORT
              value: "8080"
            - name: DISABLE_TRACING
              value: "1"
            - name: DISABLE_PROFILER
              value: "1"
          readinessProbe:
            periodSeconds: 5
            exec:
              command: ["/bin/grpc_health_probe", "-addr=:8080"]
          livenessProbe:
            periodSeconds: 5
            exec:
              command: ["/bin/grpc_health_probe", "-addr=:8080"]
          resources:
            requests:
              cpu: 100m
              memory: 64Mi
            limits:
              cpu: 200m
              memory: 128Mi
---
apiVersion: v1
kind: Service
metadata:
  name: emailservice
spec:
  type: ClusterIP
  selector:
    app: emailservice
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 8080

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: recommendationservice
spec:
  selector:
    matchLabels:
      app: recommendationservice
  template:
    metadata:
      labels:
        app: recommendationservice
    spec:
      containers:
        - name: server
          image: gcr.io/google-samples/microservices-demo/recommendationservice:v0.2.3
          ports:
            - containerPort: 8080
          readinessProbe:
            periodSeconds: 5
            exec:
              command: ["/bin/grpc_health_probe", "-addr=:8080"]
          livenessProbe:
            periodSeconds: 5
            exec:
              command: ["/bin/grpc_health_probe", "-addr=:8080"]
          env:
            - name: PORT
              value: "8080"
            - name: PRODUCT_CATALOG_SERVICE_ADDR
              value: "productcatalogservice:3550"
            - name: DISABLE_TRACING
              value: "1"
            - name: DISABLE_PROFILER
              value: "1"
            - name: DISABLE_DEBUGGER
              value: "1"
          resources:
            requests:
              cpu: 100m
              memory: 220Mi
            limits:
              cpu: 200m
              memory: 450Mi
---
apiVersion: v1
kind: Service
metadata:
  name: recommendationservice
spec:
  type: ClusterIP
  selector:
    app: recommendationservice
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: paymentservice
spec:
  selector:
    matchLabels:
      app: paymentservice
  template:
    metadata:
      labels:
        app: paymentservice
    spec:
      containers:
        - name: server
          image: gcr.io/google-samples/microservices-demo/paymentservice:v0.2.3
          ports:
            - containerPort: 50051
          env:
            - name: PORT
              value: "50051"
          readinessProbe:
            exec:
              command: ["/bin/grpc_health_probe", "-addr=:50051"]
          livenessProbe:
            exec:
              command: ["/bin/grpc_health_probe", "-addr=:50051"]
          resources:
            requests:
              cpu: 100m
              memory: 64Mi
            limits:
              cpu: 200m
              memory: 128Mi
---
apiVersion: v1
kind: Service
metadata:
  name: paymentservice
spec:
  type: ClusterIP
  selector:
    app: paymentservice
  ports:
    - protocol: TCP
      port: 50051
      targetPort: 50051

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: productcatalogservice
spec:
  selector:
    matchLabels:
      app: productcatalogservice
  template:
    metadata:
      labels:
        app: productcatalogservice
    spec:
      containers:
        - name: server
          image: gcr.io/google-samples/microservices-demo/productcatalogservice:v0.2.3
          ports:
            - containerPort: 3550
          env:
            - name: PORT
              value: "3550"
          readinessProbe:
            exec:
              command: ["/bin/grpc_health_probe", "-addr=:3550"]
          livenessProbe:
            exec:
              command: ["/bin/grpc_health_probe", "-addr=:3550"]
          resources:
            requests:
              cpu: 100m
              memory: 64Mi
            limits:
              cpu: 200m
              memory: 128Mi
---
apiVersion: v1
kind: Service
metadata:
  name: productcatalogservice
spec:
  type: ClusterIP
  selector:
    app: productcatalogservice
  ports:
    - protocol: TCP
      port: 3550
      targetPort: 3550

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: currencyservice
spec:
  selector:
    matchLabels:
      app: currencyservice
  template:
    metadata:
      labels:
        app: currencyservice
    spec:
      containers:
        - name: server
          image: gcr.io/google-samples/microservices-demo/currencyservice:v0.2.3
          ports:
            - containerPort: 7000
          env:
            - name: PORT
              value: "7000"
          readinessProbe:
            exec:
              command: ["/bin/grpc_health_probe", "-addr=:7000"]
          livenessProbe:
            exec:
              command: ["/bin/grpc_health_probe", "-addr=:7000"]
          resources:
            requests:
              cpu: 100m
              memory: 64Mi
            limits:
              cpu: 200m
              memory: 128Mi
---
apiVersion: v1
kind: Service
metadata:
  name: currencyservice
spec:
  type: ClusterIP
  selector:
    app: currencyservice
  ports:
    - protocol: TCP
      port: 7000
      targetPort: 7000

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: shippingservice
spec:
  selector:
    matchLabels:
      app: shippingservice
  template:
    metadata:
      labels:
        app: shippingservice
    spec:
      containers:
        - name: server
          image: gcr.io/google-samples/microservices-demo/shippingservice:v0.2.3
          ports:
            - containerPort: 50051
          env:
            - name: PORT
              value: "50051"
          readinessProbe:
            periodSeconds: 5
            exec:
              command: ["/bin/grpc_health_probe", "-addr=:50051"]
          livenessProbe:
            exec:
              command: ["/bin/grpc_health_probe", "-addr=:50051"]
          resources:
            requests:
              cpu: 100m
              memory: 64Mi
            limits:
              cpu: 200m
              memory: 128Mi
---
apiVersion: v1
kind: Service
metadata:
  name: shippingservice
spec:
  type: ClusterIP
  selector:
    app: shippingservice
  ports:
    - protocol: TCP
      port: 50051
      targetPort: 50051

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: adservice
spec:
  selector:
    matchLabels:
      app: adservice
  template:
    metadata:
      labels:
        app: adservice
    spec:
      containers:
        - name: server
          image: gcr.io/google-samples/microservices-demo/adservice:v0.2.3
          ports:
            - containerPort: 9555
          env:
            - name: PORT
              value: "9555"
          resources:
            requests:
              cpu: 200m
              memory: 180Mi
            limits:
              cpu: 300m
              memory: 300Mi
          readinessProbe:
            initialDelaySeconds: 20
            periodSeconds: 15
            exec:
              command: ["/bin/grpc_health_probe", "-addr=:9555"]
          livenessProbe:
            initialDelaySeconds: 20
            periodSeconds: 15
            exec:
              command: ["/bin/grpc_health_probe", "-addr=:9555"]
---
apiVersion: v1
kind: Service
metadata:
  name: adservice
spec:
  type: ClusterIP
  selector:
    app: adservice
  ports:
    - protocol: TCP
      port: 9555
      targetPort: 9555

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cartservice
spec:
  selector:
    matchLabels:
      app: cartservice
  template:
    metadata:
      labels:
        app: cartservice
    spec:
      containers:
        - name: server
          image: gcr.io/google-samples/microservices-demo/cartservice:v0.2.3
          ports:
            - containerPort: 7070
          env:
            - name: REDIS_ADDR
              value: "redis-cart:6379"
          resources:
            requests:
              cpu: 200m
              memory: 64Mi
            limits:
              cpu: 300m
              memory: 128Mi
          readinessProbe:
            initialDelaySeconds: 15
            exec:
              command:
                ["/bin/grpc_health_probe", "-addr=:7070", "-rpc-timeout=5s"]
          livenessProbe:
            initialDelaySeconds: 15
            periodSeconds: 10
            exec:
              command:
                ["/bin/grpc_health_probe", "-addr=:7070", "-rpc-timeout=5s"]
---
apiVersion: v1
kind: Service
metadata:
  name: cartservice
spec:
  type: ClusterIP
  selector:
    app: cartservice
  ports:
    - protocol: TCP
      port: 7070
      targetPort: 7070

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: checkoutservice
spec:
  selector:
    matchLabels:
      app: checkoutservice
  template:
    metadata:
      labels:
        app: checkoutservice
    spec:
      containers:
        - name: server
          image: gcr.io/google-samples/microservices-demo/checkoutservice:v0.2.3
          ports:
            - containerPort: 5050
          readinessProbe:
            exec:
              command: ["/bin/grpc_health_probe", "-addr=:5050"]
          livenessProbe:
            exec:
              command: ["/bin/grpc_health_probe", "-addr=:5050"]
          env:
            - name: PORT
              value: "5050"
            - name: PRODUCT_CATALOG_SERVICE_ADDR
              value: "productcatalogservice:3550"
            - name: SHIPPING_SERVICE_ADDR
              value: "shippingservice:50051"
            - name: PAYMENT_SERVICE_ADDR
              value: "paymentservice:50051"
            - name: EMAIL_SERVICE_ADDR
              value: "emailservice:5000"
            - name: CURRENCY_SERVICE_ADDR
              value: "currencyservice:7000"
            - name: CART_SERVICE_ADDR
              value: "cartservice:7070"
          resources:
            requests:
              cpu: 100m
              memory: 64Mi
            limits:
              cpu: 200m
              memory: 128Mi
---
apiVersion: v1
kind: Service
metadata:
  name: checkoutservice
spec:
  type: ClusterIP
  selector:
    app: checkoutservice
  ports:
    - protocol: TCP
      port: 5050
      targetPort: 5050

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: server
          image: gcr.io/google-samples/microservices-demo/frontend:v0.2.3
          ports:
            - containerPort: 8080
          readinessProbe:
            initialDelaySeconds: 10
            httpGet:
              path: "/_healthz"
              port: 8080
              httpHeaders:
                - name: "Cookie"
                  value: "shop_session-id=x-readiness-probe"
          livenessProbe:
            initialDelaySeconds: 10
            httpGet:
              path: "/_healthz"
              port: 8080
              httpHeaders:
                - name: "Cookie"
                  value: "shop_session-id=x-liveness-probe"
          env:
            - name: PORT
              value: "8080"
            - name: PRODUCT_CATALOG_SERVICE_ADDR
              value: "productcatalogservice:3550"
            - name: CURRENCY_SERVICE_ADDR
              value: "currencyservice:7000"
            - name: CART_SERVICE_ADDR
              value: "cartservice:7070"
            - name: RECOMMENDATION_SERVICE_ADDR
              value: "recommendationservice:8080"
            - name: SHIPPING_SERVICE_ADDR
              value: "shippingservice:50051"
            - name: CHECKOUT_SERVICE_ADDR
              value: "checkoutservice:5050"
            - name: AD_SERVICE_ADDR
              value: "adservice:9555"
          resources:
            requests:
              cpu: 100m
              memory: 64Mi
            limits:
              cpu: 200m
              memory: 128Mi
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  type: ClusterIP
  selector:
    app: frontend
  ports:
    - name: http
      port: 80
      targetPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: frontend-external
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
    - name: http
      port: 80
      targetPort: 8080

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-cart
spec:
  selector:
    matchLabels:
      app: redis-cart
  template:
    metadata:
      labels:
        app: redis-cart
    spec:
      containers:
        - name: redis
          image: redis:alpine
          ports:
            - containerPort: 6379
          readinessProbe:
            periodSeconds: 5
            tcpSocket:
              port: 6379
          livenessProbe:
            periodSeconds: 5
            tcpSocket:
              port: 6379
          volumeMounts:
            - mountPath: /data
              name: redis-data
          resources:
            requests:
              cpu: 70m
              memory: 200Mi
            limits:
              cpu: 125m
              memory: 256Mi
      volumes:
        - name: redis-data
          emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: redis-cart
spec:
  type: ClusterIP
  selector:
    app: redis-cart
  ports:
    - name: redis
      port: 6379
      targetPort: 6379
```

###### Step 5: Create an cluster with 3 node on Linode and download the kubeconfig.yaml file and move it to the root directory of the project.

###### Step 6: Change the permission of the kubeconfig.yaml for owner read only

```
chmod 600 online-shop-kubeconfig.yaml
```

###### Step 7: export online-shop-kubeconfig.yaml as a ENV variable KUBECONFIG

```
export KUBECONFIG=online-shop-kubeconfig.yaml
```

###### Step 8: Create a namespace

```
kubectl create ns microservices
```

###### Step 9: Create 11 deployment and service into the created namespace

```
kubectl apply -f config.yaml --namespace=microservices
```

###### Step 10: Check running pods in microservices namespace

![image](images/Screenshot%202023-05-13%20at%209.23.14%20am.png)

###### Step 11: Check svs in microservices namespace

![image](images/Screenshot%202023-05-13%20at%209.23.44%20am.png)

###### Step 12: Browser the app via loadbalancer dns provided by linode

![image](images/Screenshot%202023-05-13%20at%209.24.52%20am.png)
