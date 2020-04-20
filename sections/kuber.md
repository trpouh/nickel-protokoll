# Aufgabenstellung

Die Idee des Projektes ist es einen Kubernetes Cluster auf 3 VMS zu deployen und die Möglichkeiten die dadurch entstehen zu erkunden.  

Ein optionales Ziel ist es, diese Infrastruktur auf eine Web-Applikation mit Anbindung einer Datenbank anzuwenden und Aktivitäten zu überprüfen. 

## Kubernetes

Kubernetes ist ein Open-Source Orchestrierungs Tool, das dazu dient Container automatisiert Bereitzustellen und zu verwalten.

> Der Name Kubernets kommt aus dem griechischen und steht für Steuermann.

Wir verwenden in unserem Projekt [Docker](https://www.docker.com/) als Container Runtime.

### Aufbau und Architektur

![Kubernetes Architektur](kuber.png)

Der Vorteil bei Kubernetes liegt darin, dass die es _Pods_ orchestriert. Sie stellen die kleinstmögliche Steuerbare Einheit im Kubernetes Universum dar. Sie laufen auf _Nodes_ - also VMs oder physischen Maschinen). Ein Pod kann einen oder mehrere Container beinhalten.

Die Architektur ist auf dem Master-Slave System aufgebaut. 

Der Master ist die _Control Plane_, auf ihr wird Inventur über alle Objekte in einem Cluster geführt. Der Master steuert außerdem alle Slaves (Minions). Wir haben in unserem Fall nur einen Master-Node. Es können aber auch - zwecks Redundanz - mehre Master-Nodes in einem Kubernetes Cluster konfiguriert werden.

Auf jedem Minion muss zusätzlich auch die zu verwendente Container-Runtime installiert werden. 

# Installation

## Validierung bevor es losgeht

Bei jedem Node (VM oder physische Maschine), die im Cluster verwendet werden soll, müssen sich folgende Eigenschaften unterscheiden:

|Eigenschaft|Befehl zum Prüfen|
| - | - |
|MAC Adresse|`ip link`|
|Produkt UUID|`cat /sys/class/dmi/id product_uuid`|

Da wir für dieses Beispiel Debian 10 verwenden, müssen wir zusätzlich noch iptables in den legacy-mode schalten.

```shell
update-alternatives --set iptables /usr/sbin/iptables-legacy
update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
update-alternatives --set arptables /usr/sbin/arptables-legacy
update-alternatives --set ebtables /usr/sbin/ebtables-legacy
```

# Installation

## Docker
Als Container Runtime benutzen wir für dieses Beispiel Docker.

Zuerst müssen wir alle Dependencies von Docker installieren.

```shell
apt update
apt -y install apt-transport-https ca-certificates curl gnupg2 software-properties-common
```

Anschließend fügen für den offiziellen GPG-Key von Docker zu unserem Package-Manager hinzu. Mit diesem Schlüssel sind die Packete signiert.

```shell
curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
```

Nun können wir die offiziellen Docker-Repositories hinzufügen.

```shell
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
```

Die Installation selbst ist nun ziemlich einfach.

```shell
apt-get update && apt-get install -y \
  containerd.io=1.2.10-3 \
  docker-ce=5:19.03.4~3-0~debian-$(lsb_release -cs) \
  docker-ce-cli=5:19.03.4~3-0~debian-$(lsb_release -cs)
```

Zu guter Letzt passen wir noch die Einstellungen für Docker an.

```shell
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

mkdir -p /etc/systemd/system/docker.service.d
```

Einen kleinen Restart brauchen wir noch.

```shell
systemctl daemon-reload
systemctl restart docker
```

Wie jeder gute IT-Admin werden verifizieren wir natürlich noch am Ende die gelungene Installation.

```shell
docker version
```

Falls hier vernünftige Informationen angezeigt werden und keine Fehlermeldung ist die Installation gelungen.

## Kubernetes

Als nächsten Schritt installieren wir Kubernetes. Oder besser gesagt die 3 Services, aus denen unsere Kubernetes Installation bestehen wird.

* kubeadm
* kubelet
* kubectl

Dafür benutzen wir einen ähnlichen Ablauf, wie bei Docker. Der Einfachheit halber ist nun nicht jeder Schritt kommentiert, sondern ein fertiges Skript zu sehen.

```shell
apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
```

Auch hier folgt natürlich wieder die Verifikation.

```shell
kubeadm version
kubelet --version
kubectl version
```

# Erstellung eines Clusters

## Einstellungen

Ein kleiner Schritt hält uns noch vom Cluster ab: Wir müssen swap abschalten.

```shell 
swapoff -a
cp /etc/fstab /etc/fstab.orig
cat /etc/fstab.orig | grep -v 'swap' > /etc/fstab
```

Jetzt noch ein Reboot. 

```shell
reboot 0
```

## Kubeadm

Bevor man den Cluster installiert muss man sich für ein pod network add-on entscheiden. Die Auswahl ist da groß, weswegen wir uns schlichtweg für das beliebteste Add-On entschieden haben: Flannel

Auf unserem Master führen wir nun folgenden Befehl aus:

```shell
kubeadm init --pod-network-cidr=10.244.0.0/16
```

Mit den anderen Nodes können wir jetzt joinen. Den Befehl dafür finden wir im Output von kubeadm init am Master.

```shell
kubeadm join 10.0.0.88:6443 --token <token> \
    --discovery-token-ca-cert-hash sha256:<hash>
```


# Troubleshooting

Bei der Verifizierung am Master mit `kubectl get nodes` kann es zu folgendem Fehler kommen:

```shell
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

Das lässt sich mit den folgenden Befehlen beheben. (Credits an den [GitHub post von user csarora](https://github.com/kubernetes/kubernetes/issues/44665#issuecomment-295216655))

```shell
cp /etc/kubernetes/admin.conf $HOME/
chown $(id -u):$(id -g) $HOME/admin.conf
export KUBECONFIG=$HOME/admin.conf
```
