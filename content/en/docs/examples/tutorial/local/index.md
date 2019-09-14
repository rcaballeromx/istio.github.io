---
title: Configurando tu sistema local
overview: Configura tu sistema local para este tutorial.
weight: 3
---

En este módulo prepararás tu sistema local para este tutorial.

1. Instala [curl](https://curl.haxx.se/download.html).

2. Instala [Node.js](https://nodejs.org/en/download/).

3. Instala [Docker](https://docs.docker.com/install/).

4. Instala [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

5. Establece la variable de ambiente `KUBECONFIG` para almacenar el valor `${NAMESPACE}-user-config.yaml`
    apuntando al archivo de configuración generado cuando [configuraste tu cluster](/docs/examples/tutorial/cluster):

    {{< text bash >}}
    $ export KUBECONFIG=./${NAMESPACE}-user-config.yaml
    {{< /text >}}

6. Verifica que la configuración tomó efecto mostrando la información del namespace actual:

    {{< text bash >}}
    $ kubectl config view -o jsonpath="{.contexts == \"$(kubectl config current-context)\")].context.namespace}"
    tutorial
    {{< /text >}}

    Debes de ver el nombre de tu namespace en la consola.

7. Descarga una [versión de Istio](https://github.com/istio/istio/releases) y extrae
    la herramienta de linea de comandos `istioctl`del directorio `bin`. Verifica que
    la herramienta `istioctl` funciona con la siguiente instrucción:

    {{< text bash >}}
    $ istioctl version
    version.BuildInfo{Version:"release-1.1-20190214-09-16", GitRevision:"6113e155ac85e2485e30dfea2b80fd97afd3130a", User:"root", Host:"4496ae63-3039-11e9-86e9-0a580a2c0304", GolangVersion:"go1.10.4", DockerHub:"gcr.io/istio-release", BuildStatus:"Clean", GitTag:"1.1.0-snapshot.6-6-g6113e15"}
    {{< /text >}}

Felicidades!

Configuraste tu sistema local y tienes tu  [Kubernetes cluster listo](/docs/examples/tutorial/setup-kubernetes-cluster) para aprender.
Estas listo para continuar y [ejecutar un servicio localmente](/docs/examples/tutorial/single/).
