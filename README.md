# minikube-ingress

      ---
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        annotations: {}
        labels:
          addonmanager.kubernetes.io/mode: Reconcile
          app.kubernetes.io/name: nginx-ingress-controller
          app.kubernetes.io/part-of: kube-system
        name: nginx-ingress-controller
        namespace: kube-system
      spec:
        replicas: 1
        selector:
          matchLabels:
            addonmanager.kubernetes.io/mode: Reconcile
            app.kubernetes.io/name: nginx-ingress-controller
            app.kubernetes.io/part-of: kube-system
        template:
          metadata:
            annotations:
              prometheus.io/port: '10254'
              prometheus.io/scrape: 'true'
            labels:
              addonmanager.kubernetes.io/mode: Reconcile
              app.kubernetes.io/name: nginx-ingress-controller
              app.kubernetes.io/part-of: kube-system
          spec:
            containers:
            - args:
              - "/nginx-ingress-controller"
              - "--configmap=$(POD_NAMESPACE)/nginx-load-balancer-conf"
              - "--tcp-services-configmap=$(POD_NAMESPACE)/tcp-services"
              - "--udp-services-configmap=$(POD_NAMESPACE)/udp-services"
              - "--annotations-prefix=nginx.ingress.kubernetes.io"
              - "--report-node-internal-ip-address"
              env:
              - name: POD_NAME
                valueFrom:
                  fieldRef:
                    fieldPath: metadata.name
              - name: POD_NAMESPACE
                valueFrom:
                  fieldRef:
                    fieldPath: metadata.namespace
              image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.25.1
              imagePullPolicy: IfNotPresent
              livenessProbe:
                httpGet:
                  path: "/healthz"
                  port: 10254
                  scheme: HTTP
                initialDelaySeconds: 10
                timeoutSeconds: 1
              name: nginx-ingress-controller
              ports:
              - containerPort: 80
                hostPort: 80
              - containerPort: 443
                hostPort: 443
              - containerPort: 18080
                hostPort: 18080
              readinessProbe:
                httpGet:
                  path: "/healthz"
                  port: 10254
                  scheme: HTTP
              securityContext:
                capabilities:
                  add:
                  - NET_BIND_SERVICE
                  drop:
                  - ALL
                runAsUser: 33
            serviceAccountName: nginx-ingress
            terminationGracePeriodSeconds: 60
