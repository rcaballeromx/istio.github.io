---
title: Prepara tu Kubernetes cluster en 
overview:  Configura tu Kubernetes cluster para este tutorial.
weight: 2
---

Completa este módulo para configurar un Kubernetes cluster con Istio installado y un namespace para este tutorial.

1. Cerciórate de que tienes accesso al [Kubernetes cluster](https://kubernetes.io/docs/tutorials/kubernetes-basics/).
    Puedes usar [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/docs/quickstart) o
    [IBM Cloud Kubernetes Service](https://cloud.ibm.com/docs/containers?topic=containers-getting-started).

2. Crea una variable de ambiente para almacenar el nombre de tu namespace para ejecutar las instrucciones de este tutorial.
    Puedes usar cualquier nombre, por ejemplo, `tutorial`, pero `cosas-chidas` también funciona.

    {{< text bash >}}
    $ export NAMESPACE=tutorial
    {{< /text >}}

3. Crea el namespace:

    {{< text bash >}}
    $ kubectl create namespace $NAMESPACE
    {{< /text >}}

4. Instala Istio con TLS mutua en modo estricto habilitado. Recuerda que debes usar la pestaña `strict mutual TLS` cuando sigas
    [las instrucciones de instalación para Kubernetes](/docs/setup/kubernetes/install/kubernetes/#installation-steps).

5. [Habilita la bitácora de accesos de Envoy](/docs/tasks/telemetry/logs/access-log/#enable-envoy-s-access-logging).

6. Crea un recurso de ingreso en Kubernetes para los siguientes servicios que son comunes en Istio:

    - [Grafana](https://grafana.com/docs/guides/getting_started/)
    - [Jaeger](https://www.jaegertracing.io/docs/1.13/getting-started/)
    - [Prometheus](https://prometheus.io/docs/prometheus/latest/getting_started/)
    - [Kiali](https://www.kiali.io/documentation/getting-started/)

    No tienes idea de que son esos servicios? No te preocupes! En el futuro próximo este tutorial incluirá módulos enseñando cada uno de ellos.

    {{< text bash >}}
    $ kubectl apply -f - <<EOF
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: istio-system
      namespace: istio-system
    spec:
      rules:
      - host: my-istio-dashboard.io
        http:
          paths:
          - path: /
            backend:
              serviceName: grafana
              servicePort: 3000
      - host: my-istio-tracing.io
        http:
          paths:
          - path: /
            backend:
              serviceName: tracing
              servicePort: 80
      - host: my-istio-logs-database.io
        http:
          paths:
          - path: /
            backend:
              serviceName: prometheus
              servicePort: 9090
      - host: my-kiali.io
        http:
          paths:
          - path: /
            backend:
              serviceName: kiali
              servicePort: 20001
    EOF
    {{< /text >}}

7. Crea un rol para darle acceso de lectura al namespace `istio-system`. Este
   rol se require si vas a limitar permisos para múltiples participantes.

    {{< text bash >}}
    $ kubectl apply -f - <<EOF
    kind: Role
    apiVersion: rbac.authorization.k8s.io/v1beta1
    metadata:
      name: istio-system-access
      namespace: istio-system
    rules:
    - apiGroups: ["", "extensions", "apps"]
      resources: ["*"]
      verbs: ["get", "list"]
    EOF
    {{< /text >}}

8. Cada participante necesita un cuenta de servicio que represente su identidad
   en la malla. Crea una cuenta de servicio para cada participante:

    {{< text bash >}}
    $ kubectl apply -f - <<EOF
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: ${NAMESPACE}-user
      namespace: $NAMESPACE
    EOF
    {{< /text >}}

9. Limita los permisos de cada participante. Durante el, participantes solo necesitan crear recursos en su namespace
    y leer recursos en el namespace `istio-system`.
    Incluso si trabajas en tu propio cluster, estos límites ayudan a prevenir que tu aprendizaje
    interfiera con namespaces en tu cluster.

    Crea un rol para permitir acceso de lectura y escritura para el namespace de cada participante.
    Liga la cuenta de servicio del participante con este rol que creaste y con el rol que tiene los
    permisos de lectura para los recursos de `istio-system`:

    {{< text bash >}}
    $ kubectl apply -f - <<EOF
    kind: Role
    apiVersion: rbac.authorization.k8s.io/v1beta1
    metadata:
      name: ${NAMESPACE}-access
      namespace: $NAMESPACE
    rules:
    - apiGroups: ["", "extensions", "apps", "networking.k8s.io", "networking.istio.io", "authentication.istio.io",
                  "rbac.istio.io", "config.istio.io"]
      resources: ["*"]
      verbs: ["*"]
    ---
    kind: RoleBinding
    apiVersion: rbac.authorization.k8s.io/v1beta1
    metadata:
      name: ${NAMESPACE}-access
      namespace: $NAMESPACE
    subjects:
    - kind: ServiceAccount
      name: ${NAMESPACE}-user
      namespace: $NAMESPACE
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: Role
      name: ${NAMESPACE}-access
    ---
    kind: RoleBinding
    apiVersion: rbac.authorization.k8s.io/v1beta1
    metadata:
      name: ${NAMESPACE}-istio-system-access
      namespace: istio-system
    subjects:
    - kind: ServiceAccount
      name: ${NAMESPACE}-user
      namespace: $NAMESPACE
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: Role
      name: istio-system-access
    EOF
    {{< /text >}}

10. Cada participante necesita usar su propio archivo de configuración de Kubernetes. Ese archivo de configuracón especifica
    los detalles del cluster, de la cuenta de servicio, las credenciales and el namespace del participante.
    La instrucción `kubectl` usa el archivo de configuración para operar el cluster.

    Genera el archivo de configuración deKubernetes para cada participante:

    {{< text bash >}}
    $ cat <<EOF > ./${NAMESPACE}-user-config.yaml
    apiVersion: v1
    kind: Config
    preferences: {}

    clusters:
    - cluster:
        certificate-authority-data: $(kubectl get secret $(kubectl get sa ${NAMESPACE}-user -n $NAMESPACE -o jsonpath={.secrets..name}) -n $NAMESPACE -o jsonpath='{.data.ca\.crt}')
        server: $(kubectl config view -o jsonpath="{.clusters[?(.name==\"$(kubectl config view -o jsonpath="{.contexts[?(.name==\"$(kubectl config current-context)\")].context.cluster}")\")].cluster.server}")
      name: ${NAMESPACE}-cluster

    users:
    - name: ${NAMESPACE}-user
      user:
        as-user-extra: {}
        client-key-data: $(kubectl get secret $(kubectl get sa ${NAMESPACE}-user -n $NAMESPACE -o jsonpath={.secrets..name}) -n $NAMESPACE -o jsonpath='{.data.ca\.crt}')
        token: $(kubectl get secret $(kubectl get sa ${NAMESPACE}-user -n $NAMESPACE -o jsonpath={.secrets..name}) -n $NAMESPACE -o jsonpath={.data.token} | base64 --decode)

    contexts:
    - context:
        cluster: ${NAMESPACE}-cluster
        namespace: ${NAMESPACE}
        user: ${NAMESPACE}-user
      name: ${NAMESPACE}

    current-context: ${NAMESPACE}
    EOF
    {{< /text >}}

11. Si eres instructor, envía el archivo de configuración generado a cada participante. Si estas configrando el cluster para tí,
    usa el archivo de configuración como describimos en
    [el módulo para configurar tu sistema local](/docs/examples/tutorial/local).

Felicidades¡

Haz completado la configuración de tu cluster y puedes continuar con la [configuración de tu sistema](/docs/examples/tutorial/local).

Ya que esté configurado tu sistema local, puedes [ejecutar tu primer servicio](/docs/examples/tutorial/single/)!
