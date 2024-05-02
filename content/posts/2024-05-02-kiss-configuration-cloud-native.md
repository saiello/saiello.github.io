+++
title = 'A Simpler Approach to Configure Cloud-Native Applications in Quarkus and Spring Boot'
date = 2024-05-02T15:11:06+02:00
draft = false
tags = ['java', 'quarkus', 'kubernetes', 'springboot']
+++


Hey there, developers! 

I want to talk about something near and dear to my heart: a common practice that I often find applied when configuring and deploying cloud-native applications using frameworks such as Quarkus and Spring Boot, two of the main frameworks in the Java ecosystem. 

With the immutable nature of container images and the recommendations of the 12-factor app in mind, the use of environment variables to override configuration has become a common practice. 

However, especially in the context of Quarkus and Spring Boot, what is usually done to achive the result is more complex than it could be. 

That's why today, I want to shed some light on a KISS approach.


### Bad Approach

The widespread approach is defining custom placeholders in our `application.properties` files to apparently enabling overriding them with environment variables. 

```
my.application.property=${MY_APP_PROP:override-me}
quarkus.datasource.jdbc.url=${DB_URL:jdbc:postgresql://localhost/quarkus}
```

Then in the Kubernetes resources specifying the env variables one by one.


```yaml
---
apiVersion: apps/v1
kind: ConfigMap
metadata:
  name: my-app-config
data:
  db.url: jdbc:postgresql://dev/quarkus
  my.application.property: overridden


---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    spec:
      containers:
      - name: my-app
        image: my-app:latest
        env:

        - name: DB_URL
          valueFrom:
            configMapKeyRef:
              name: my-app-config
              key: db.url

        - name: MY_APP_PROP
          valueFrom:
            configMapKeyRef:
              name: my-app-config
              key: my.app-prop-override   
              
        # further configuration here ...
```              

By adhering to this approach, every time you want to add a property you will be forced to edit 3 different points and sometimes even rebuild your application.



### Better Approach


So, what's the solution to these common pitfalls? It's simple, really: embrace **convention over configuration**. 

Instead of drowning in placeholders or cluttering your Kubernetes deployments with configuration details, let's take a step back and rethink our approach.

Both Quarkus and Spring Boot offer mechanisms for reading configuration from multiple sources with a specific precedence. By leveraging these built-in features and sticking to established or your own conventions, we can streamline our configuration process and make our lives a whole lot easier.


The previous example can be rewritten in a much much simpler way, setting in the application properties the default values only:

```
# set default values only
my.application.property=my-default-value
quarkus.datasource.jdbc.url=jdbc:postgresql://localhost/quarkus
```

And mounting the whole config map in the Kubernetes resource:

```yaml
---
apiVersion: apps/v1
kind: ConfigMap
metadata:
  name: my-app-config-envs
data:
  QUARKUS_DATASOURCE_JDBC_URL: jdbc:postgresql://dev/quarkus
  MY_APPLICATION_PROPERTY: overridden

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    spec:
      containers:
      - name: my-app
        image: my-app:latest
        envFrom:
        - configMapRef:
          name: my-app-config-envs
```

The simplification may seem minimal, but it is worth noting that you will no longer need edit several places to add or modify a simple property. Expecially the one that you didn't see upfront.


### Wrapping Up

I hope most of you are already familiar with what I've discussed so far. But for my experience is not always true.

By avoiding the pitfalls of defining placeholders and cluttering our Kubernetes deployments, we can simplify our configuration process and focus on what really matters: building awesome software.

So, the next time you find yourself knee-deep in configuration woes, remember to keep it simple, stick to convention, and embrace the power of the framework you are using. Your future self (and me) will thank you for it!


### Resources

- [Quarkus Config Reference](https://quarkus.io/guides/config-reference)
- [Twelve Factor App - Config](https://12factor.net/config)
- [Define Environment Variable for Container](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/)