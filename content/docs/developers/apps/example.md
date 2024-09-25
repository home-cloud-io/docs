---
weight: 312
title: "App Example"
description: "Learn about developing apps through a quick example."
icon: "article"
draft: true
toc: true
---

## Let's build an app!

Follow the steps below to scaffold your first Home Cloud app!

### 1. `Chart.yaml`: The app's definition

The first piece of your app is the `Chart.yaml` file which defines the top-level specification of your app. Below is an example for a simple app:

```yaml
apiVersion: v2
name: hello-world
description: A simple "Hello World" application demonstrating a Home Cloud app.
icon: https://avatars.githubusercontent.com/u/1412239?s=48&v=4
type: application
version: 0.0.1
appVersion: "1.27.1"
home: https://nginx.org/
sources:
  - https://github.com/nginx/nginx
annotations:
  displayName: Hello World
```

Let's break down each component there so they make sense:

- `apiVersion`: This is the Helm API version and can always just be `v2`.
- `name`: This is the internal name of the app and should be globally unique (eventually this will be replaced with a UUID). This must match the folder of chart within the store repository.
- `description`: This is a short (single sentence) description of your app.
- `icon`: A URL to a publically accessible icon for your app. It should be fairly small (roughly 48x48px).
- `type`: This can always be left as `application`.
- `version`: This defines the *chart* version, **not** the *app* version. This needs to be updated whenever a change is made anywhere in the chart, even if you didn't make any change to the application itself.
- `appVersion`: This defines the version of the actual app itself. Usually this value matches the Docker image tag that is currently your latest stable release (more on this later).
- `home`: The homepage of your application. This can simply be a URL to your git repository if you don't have a dedicated website.
- `sources`: The URL(s) to the source code of your application.
- `annotations`: For now the only operative annotation is `displayName` which is the name of the app that will be shown to users.

### 2. `values.yaml`: The app's configuration

The second piece of a Home Cloud app is the `values.yaml` file which contains all the configurable values that are slotted into the templates (shown below) that define the actual application specification. Here's an example for our simple app:

```yaml
app:
  replicaCount: 1

homeCloud:
  routes:
    - name: hello
      service:
        name: hello-world
        port: 80
```

Let's break that down...

Everything under the `app` key is up to you as the developer to configure. These are values that you want to be flexible for different types of usecases by various users. You could allow options like changing the default language or enabling/disabling certain features. We'll see how to reference these values below.

Everything under the `homeCloud` key is used by Home Cloud to provision dependencies for your application so that you don't have to worry about them yourself. In this example, the app is requesting Home Cloud to create a route from `http://hello.local` to port `80` on the service itself. Keep reading to see how this link is made.

### 3. `templates/`: The app's specification

#### Deployment

By using Helm templates, we can create modifiable application specifications that can accept inputs from the above `Chart.yaml` and `values.yaml` files.

Let's start with a simple `templates/deployment.yaml` file which defines the way to run the Docker container that packages our app:

{{< prism lang="yaml" line="5,7,20" >}}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world
  namespace: {{ .Release.Name }}
spec:
  replicas: {{ .Values.app.replicaCount }}
  selector:
    matchLabels:
      app: hello-world
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      affinity:
        nodeAffinity: {{ toYaml .Values.homeCloud.nodeAffinity | nindent 12 }}
      containers:
        - name: hello-world
          image: nginx:{{ .Chart.AppVersion }}
          ports:
            - name: http
              containerPort: 80
{{< /prism >}}

In Kubernetes a [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) is a specification that defines how to run one or more Docker containers all at once. Here we're just running a single Docker image but we're actually able to define how many `replicas` of that Docker image to run at once by referencing the `app.replicaCount` we specified earlier in the `values.yaml` (see line 7).

Each Home Cloud application runs within its own [Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) and this is defined by the `name` which we specified earlier in the `Chart.yaml` file (see line 5).

On line 20, notice how we're referencing the `appVersion` to set the Docker image tag we previously defined in the `Chart.yaml` file. This means that whenever you want to update your Home Cloud app you can simply change the `appVersion` attribute in the `Chart.yaml` file and the system will automatically know to pull the updated Docker image.

#### Service

We have one final piece of this app to look at and that's the routing definition we requested in the [`values.yaml`](#2-valuesyaml-the-apps-configuration). Let's look at the `service.yaml` template file and see how it references our requested route:

{{< prism lang="yaml" line="4,8-12" >}}
apiVersion: v1
kind: Service
metadata:
  name: hello-world
  namespace: {{ .Release.Name }}
spec:
  type: ClusterIP
  ports:
    - name: http
      port: 80
  selector:
    app: hello-world
{{< /prism >}}

You can think of a [Service](https://kubernetes.io/docs/concepts/services-networking/service/) as a map between the Home Cloud external route and the internal target of your app's containers. The `metadata.name` must match the `homeCloud.routes[x].service.name` within the `values.yaml` file (see line 4). That tells Home Cloud where to route external traffic to.

You tell the Service the target container and port by defining the `ports` and `selector` that matches your specified values in the `deployment.yaml` (see lines 8-12).

### 4. `README.md`: The app's front page

This isn't necessary for your app to run, but by adding a `README.md` you can describe your app and how to use it to prospective users. This file should be in markdown and it will be rendered for users to see within the Home Cloud admin dashboard's app store.

### Conclusion

And that's it! With just those five files your app can now be deployed by Home Cloud! Continue reading through the rest of the documentation here to learn about adding persistent storage, databases, and more to your app.
