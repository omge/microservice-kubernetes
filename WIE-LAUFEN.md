# Beispiel starten

Die ist eine Schritt-für-Schritt-Anleitung zum Starten der Beispiele.

## Installation

* Installiere
[minikube](https://github.com/kubernetes/minikube/releases). Minikube
bietet eine Kubernetes-Umgebung in einer virtuellen Maschine. Minikube
ist einfach zu benutzen und zu installieren. Es soll keine
Produktionsumgebung sein, sondern dient nur dazu, Kubernetes
auszuprobieren oder Entwicklermaschinen aufzubauen.

* Installiere
  [kubectl](https://kubernetes.io/docs/tasks/kubectl/install/). Das
  ist das Kommandozeilenwerkzeug für den Umgang mit Kubernetes.

## Docker Images bauen (optional)

Dieser Schritt ist *optional*. im öffentlichen Docker Hub liegen
bereits Docker Images bereit, die du nutzen kannst. 

* Die Beispiele sind in Java implementiert. Daher muss Java
  installiert werden. Die Anleitung findet sich unter
  https://www.java.com/en/download/help/download_options.xml . Da die
  Beispiele kompiliert werden müssen, muss ein JDK (Java Development
  Kit) installiert werden. Das JRE (Java Runtime Environment) reicht
  nicht aus. Nach der Installation sollte sowohl `java` und `javac` in
  der Eingabeaufforderung möglich sein.

* Die Projekte baut Maven. Zur Installation siehe
  https://maven.apache.org/download.cgi>. Nun sollte `mvn` in der
  Eingabeaufforderung eingegeben werden können.

* Die Beispiele laufen in Docker Containern. Dazu ist eine
  Installation von Docker Community Edition notwendig, siehe
  https://www.docker.com/community-edition/ . Docker kann mit
  `docker` aufgerufen werden. Das sollte nach der Installation ohne
  Fehler möglich sein.

Wechsel in das Verzeichnis `microservice-kubernetes-demo` und starte `mvn clean
package`. Das wird einige Zeit dauern:

```
[~/microservice-kubernetes/microservice-kubernetes-demo]mvn clean package
....
[INFO] 
[INFO] --- maven-jar-plugin:2.6:jar (default-jar) @ microservice-kubernetes-demo-order ---
[INFO] Building jar: /Users/wolff/microservice-kubernetes/microservice-kubernetes-demo/microservice-kubernetes-demo-order/target/microservice-kubernetes-demo-order-0.0.1-SNAPSHOT.jar
[INFO] 
[INFO] --- spring-boot-maven-plugin:1.4.5.RELEASE:repackage (default) @ microservice-kubernetes-demo-order ---
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO] 
[INFO] microservice-kubernetes-demo ....................... SUCCESS [  0.986 s]
[INFO] microservice-kubernetes-demo-hystrix-dashboard ..... SUCCESS [  2.494 s]
[INFO] microservice-kubernetes-demo-customer .............. SUCCESS [ 16.953 s]
[INFO] microservice-kubernetes-demo-catalog ............... SUCCESS [ 18.016 s]
[INFO] microservice-kubernetes-demo-order ................. SUCCESS [ 18.512 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 57.633 s
[INFO] Finished at: 2017-09-08T09:36:32+02:00
[INFO] Final Memory: 56M/420M
[INFO] ------------------------------------------------------------------------
```

Falls es dabei zu Fehlern kommt:

* Stelle sicher, dass die Datei `settings.xml` im Verzeichnis  `.m2`
in deinem Heimatverzeichnis keine Konfiguration für ein spezielles
Maven Repository enthalten. Im Zweifelsfall kannst du die Datei
einfach löschen.

* Die Tests nutzen einige Ports auf dem Rechner. Stelle sicher, dass
  im Hintergrund keine Server laufen.

* Führe die Tests beim Build nicht aus: `mvn clean package package
  -Dmaven.test.skip=true`.

* In einigen selten Fällen kann es vorkommen, dass die Abhängigkeiten
  nicht korrekt heruntergeladen werden. Wenn du das Verzeichnis
  `repository` im Verzeichnis `.m2` löscht, werden alle Abhängigkeiten
  erneut heruntergeladen.

Der Java-Code ist nun kompiliert. Der nächste Schritt ist, die Docker
Images zu erstellen und sie in den öffentlichen Docker Hub hochzuladen:

* Erstelle dir einen Account im öffentlichen
[Docker Hub](https://hub.docker.com/).

* Weise die Umgebungsvariable `DOCKER_ACCOUNT` den Namen des Accounts
zu, den du gerade erzeugt hast.

* Starte `docker-build.sh` im Verzeichnis
`microservice-kubernetes-demo`. Es erzeugt die Docker Images und lädt
sie in den Docker Hub hoch. Das dauert natürlich einige Zeit:

```
[~/microservice-kubernetes/microservice-kubernetes-demo]export DOCKER_ACCOUNT=ewolff
[~/microservice-kubernetes/microservice-kubernetes-demo]echo $DOCKER_ACCOUNT
ewolff
[~/microservice-kubernetes/microservice-kubernetes-demo]./docker-build.sh 
...
Removing intermediate container 36e9b0c2ac0e
Successfully built b76261d1e4ee
Successfully tagged microservice-kubernetes-demo-hystrix-dashboard:latest
The push refers to a repository [docker.io/ewolff/microservice-kubernetes-demo-hystrix-dashboard]
f4ffcb9c643d: Pushed 
14c5bfa09694: Mounted from ewolff/microservice-kubernetes-demo-order 
41a5c76632fc: Mounted from ewolff/microservice-kubernetes-demo-order 
5bef08742407: Mounted from ewolff/microservice-kubernetes-demo-order 
latest: digest: sha256:36d87ea5c8628da9a6677c1eafb9009c8f99310f5376872e7b9a1edace37d1a0 size: 1163
```

## Run the containers

* Installiere eine Minikube-Instanz mit `minikube start
--memory=4000`. Die Instanz hat dann
   4.000 MB RAM. Das sollte für das Beispiel ausreichend sein.

```
[~/microservice-kubernetes]minikube start --memory=4000
Starting local Kubernetes v1.7.5 cluster...
Starting VM...
Getting VM IP address...
Moving files into cluster...
Setting up certs...
Connecting to cluster...
Setting up kubeconfig...
Starting cluster components...
Kubectl is now configured to use the cluster.
```

* Falls du deine eigenen Docker Images erstellt hast, muss die
  Umgebungsvariable `DOCKER_ACCOUNT` den Namen deines Accounts beim
  Docker Hub enthalten.

* Starte `kubernetes-deploy.sh` im Verzeichnis
`microservice-kubernetes-demo` :

```
[~/microservice-kubernetes/microservice-kubernetes-demo]./kubernetes-deploy.sh
deployment "apache" created
service "apache" exposed
deployment "catalog" created
service "catalog" exposed
deployment "customer" created
service "customer" exposed
deployment "order" created
service "order" exposed
deployment "hystrix-dashboard" created
service "hystrix-dashboard" exposed
```

Das Skript deployt die Images und erzeugt die Pods. Pods können einen
oder mehrere Docker Container enthalten. In diesem Beispiel enthält
jeder Pod nur einen Docker container.

Außerdem erstellt das Skript Services. Services haben eine eindeutige
IP-Adresse und einen Eintrag im DNS-Namenssystem. Services setzten
außerdem Lastverteilung über mehrere Pods um. Man kann sich die
Services auch anschauen:

* Führe `kubectl get services` aus, um alle Services zu sehen:

```
[~/microservice-kubernetes/microservice-kubernetes-demo]kubectl get services
NAME                CLUSTER-IP   EXTERNAL-IP   PORT(S)          AGE
apache              10.0.0.90    <pending>     80:31214/TCP     46s
catalog             10.0.0.219   <pending>     8080:30161/TCP   46s
customer            10.0.0.163   <pending>     8080:30620/TCP   45s
hystrix-dashboard   10.0.0.220   <pending>     8080:32076/TCP   44s
kubernetes          10.0.0.1     <none>        443/TCP          3m
order               10.0.0.21    <pending>     8080:30616/TCP   45s
```


* `kubectl describe services` gibt mehr Details aus. Der Befehlt
  funktioniert auch für Pods (`kubectl describe pods`) und Deployments
  (`kubectl describe deployments`).

```
[~/microservice-kubernetes/microservice-kubernetes-demo]kubectl describe services
...

Name:			order
Namespace:		default
Labels:			run=order
Annotations:		<none>
Selector:		run=order
Type:			LoadBalancer
IP:			10.0.0.21
Port:			<unset>	8080/TCP
NodePort:		<unset>	30616/TCP
Endpoints:		172.17.0.7:8080
Session Affinity:	None
Events:			<none>
```

* Du kannst dir auch eine Liste der Pods ausgeben lassen:

```
[~/microservice-kubernetes/microservice-kubernetes-demo]kubectl get pods
NAME                                READY     STATUS    RESTARTS   AGE
apache-3412280829-k5z5p             1/1       Running   0          2m
catalog-269679894-60dr0             1/1       Running   0          2m
customer-1984516559-1ffjk           1/1       Running   0          2m
hystrix-dashboard-859915717-f0sxg   1/1       Running   0          2m
order-2204540131-nks5s              1/1       Running   0          2m
```

* ...und du kannst die Logs eines Pods sehen:

```
[wolff@TeraMacBook:~/microservice-kubernetes/microservice-kubernetes-demo]kubectl logs catalog-269679894-60dr0 

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.4.5.RELEASE)
...
2017-09-08 08:11:06.128  INFO 7 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Registering beans for JMX exposure on startup
2017-09-08 08:11:06.158  INFO 7 --- [           main] o.s.c.support.DefaultLifecycleProcessor  : Starting beans in phase 0
2017-09-08 08:11:06.746  INFO 7 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
2017-09-08 08:11:06.803  INFO 7 --- [           main] c.e.microservice.catalog.CatalogApp      : Started CatalogApp in 53.532 seconds (JVM running for 54.296)
...
```

* Außerdem kannst du ein Kommando in einem Pod ausführen:

```
[wolff@TeraMacBook:~/microservice-kubernetes/microservice-kubernetes-demo]kubectl exec catalog-269679894-60dr0  /bin/ls
bin
dev
etc
home
lib
lib64
media
microservice-kubernetes-demo-catalog-0.0.1-SNAPSHOT.jar
mnt
proc
root
run
sbin
srv
sys
tmp
usr
var
```

* Und du kannst eine Shell in einem Pod starten:

```
[wolff@TeraMacBook:~/microservice-kubernetes/microservice-kubernetes-demo]kubectl exec catalog-269679894-60dr0  -it /bin/sh
/ # ls
bin                                                      proc
dev                                                      root
etc                                                      run
home                                                     sbin
lib                                                      srv
lib64                                                    sys
media                                                    tmp
microservice-kubernetes-demo-catalog-0.0.1-SNAPSHOT.jar  usr
mnt                                                      var
/ # 
```

* Mit `minikube service apache` kannst du die Webseite des Apache
 httpd Server im Web-Browser öffnen. Dabei kannst du auch sehen, dass
 dem Service ein Port auf dem virutellen Minikube-Server zugewiesen
 worden ist.

Der Typ des Service ist `LoadBalancer`. Eigentlich sollte daher der
Service an einen externen Load Balancer angeschlossen werden. Das
funktioniert mit minikube nicht, so dass der Serice unter einem
bestimmten Port auf dem minikube-Host verfügbar ist.

* Um alle Services und Deployments zu löschen, starte
  `kubernetes-remove.sh`:

```
[~/microservice-kubernetes/microservice-kubernetes-demo]./kubernetes-remove.sh 
service "apache" deleted
service "catalog" deleted
service "customer" deleted
service "order" deleted
service "hystrix-dashboard" deleted
deployment "apache" deleted
deployment "catalog" deleted
deployment "customer" deleted
deployment "order" deleted
deployment "hystrix-dashboard" deleted
```
