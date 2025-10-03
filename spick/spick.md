# Kubernetes & Cloud Computing - Prüfungsspick

# Inhaltsverzeichnis

1. [Command Reference: Minikube, kubectl & oc](#1-command-reference-minikube-kubectl--oc)
   - [Minikube Commands (Lokales Kubernetes Cluster)](#-minikube-commands-lokales-kubernetes-cluster)
   - [kubectl Commands (Kubernetes CLI)](#-kubectl-commands-kubernetes-cli)
   - [oc Commands (OpenShift CLI)](#-oc-commands-openshift-cli)
   - [Debugging-Workflow](#debugging-workflow)

2. [YAML Files für Kubernetes Components](#2-yaml-files-für-kubernetes-components)
   - [Grundaufbau einer Kubernetes YAML-Datei](#grundaufbau-einer-kubernetes-yaml-datei)
   - [Metadata Specifications](#metadata-specifications)

3. [Kubernetes Components](#3-kubernetes-components)
   - [Pod](#pod)
   - [Deployment](#deployment)
   - [Service](#service)
   - [ReplicaSet](#replicaset)
   - [StatefulSet](#statefulset)
   - [DaemonSet](#daemonset)
   - [Namespace](#namespace)
   - [Ingress](#ingress)
   - [PersistentVolume (PV) & PersistentVolumeClaim (PVC)](#persistentvolume-pv--persistentvolumeclaim-pvc)

4. [Cloud Computing - Grundlagen](#4-cloud-computing---grundlagen)
   - [Wie ist der Begriff Cloud entstanden?](#wie-ist-der-begriff-cloud-entstanden)
   - [NIST Definition der Cloud](#nist-definition-der-cloud)

5. [Die 5 Merkmale einer Cloud](#5-die-5-merkmale-einer-cloud)

6. [Cloud Dienstleistungen](#6-cloud-dienstleistungen)

7. [Cloud Anbieter](#7-cloud-anbieter)

8. [Cloud Deployment Modelle](#8-cloud-deployment-modelle)

9. [Cloud Service Modelle](#9-cloud-service-modelle)

10. [Monitoring vs. Logging](#10-monitoring-vs-logging)

11. [Cloud - Vorteile & Nachteile](#11-cloud---vorteile--nachteile)

12. [Shared Responsibility Model](#12-shared-responsibility-model)

13. [Container-Technologie](#13-container-technologie)
    - [Was ist Container-Technologie?](#was-ist-container-technologie)
    - [Container vs. Virtuelle Maschinen](#container-vs-virtuelle-maschinen)
    - [Können VMs immer durch Container ersetzt werden?](#können-vms-immer-durch-container-ersetzt-werden)

14. [Produkte für VMs & Container](#14-produkte-für-vms--container)

15. [Self-Managed vs. Fully Managed](#15-self-managed-vs-fully-managed)

16. [Container-Orchestrierung](#16-container-orchestrierung)
    - [Warum braucht man Container-Orchestrierung?](#warum-braucht-man-container-orchestrierung)
    - [Wie funktioniert Container-Orchestrierung?](#wie-funktioniert-container-orchestrierung)
    - [Container-Orchestrierung Technologien](#container-orchestrierung-technologien)

17. [Horizontale Skalierung](#17-horizontale-skalierung)

18. [Deployment Strategien](#18-deployment-strategien)

19. [Fragen zu Block 7](#19-fragen-zu-block-7)

## 1. Command Reference: Minikube, kubectl & oc

### 🔷 Minikube Commands (Lokales Kubernetes Cluster)

#### Cluster Management
```bash
# Minikube starten
minikube start
minikube start --driver=docker          # Mit Docker Driver

# Minikube stoppen
minikube stop

# Minikube löschen
minikube delete
minikube delete --all                   # Alle Profile löschen

# Cluster-Status anzeigen
minikube status

```

#### Cluster-Informationen
```bash
# Kubernetes Dashboard öffnen
minikube dashboard

# Cluster IP anzeigen
minikube ip

# Minikube Logs
minikube logs

```
---

### 🔷 kubectl Commands (Kubernetes CLI)

#### Cluster-Informationen
```bash
# Cluster-Info anzeigen
kubectl cluster-info
kubectl cluster-info dump               # Detaillierte Cluster-Informationen

# Nodes anzeigen
kubectl get nodes
kubectl get nodes -o wide               # Mit mehr Details
kubectl describe node <node-name>

# Konfiguration anzeigen
kubectl config view
kubectl config get-contexts             # Alle Contexts
kubectl config current-context          # Aktueller Context
kubectl config use-context <context>    # Context wechseln
```

#### Ressourcen anzeigen (GET)
```bash
# Pods
kubectl get pods
kubectl get pods -n <namespace>         # In bestimmtem Namespace
kubectl get pods --all-namespaces       # Alle Namespaces (oder -A)
kubectl get pods -o wide                # Mehr Details (Node, IP)
kubectl get pods -o yaml                # YAML-Output
kubectl get pods -o json                # JSON-Output
kubectl get pods --watch                # Live-Updates (oder -w)
kubectl get pods --show-labels          # Mit Labels
kubectl get pods -l app=nginx           # Filter nach Label

# Andere Ressourcen
kubectl get deployments
kubectl get services (oder svc)
kubectl get replicasets (oder rs)
kubectl get namespaces (oder ns)
kubectl get configmaps (oder cm)
kubectl get secrets
kubectl get ingress (oder ing)
kubectl get persistentvolumes (oder pv)
kubectl get persistentvolumeclaims (oder pvc)
kubectl get statefulsets (oder sts)
kubectl get daemonsets (oder ds)

# Alle Ressourcen in einem Namespace
kubectl get all
kubectl get all -n <namespace>
```

#### Detaillierte Informationen (DESCRIBE)
```bash
# Details zu Ressourcen
kubectl describe pod <pod-name>
kubectl describe deployment <deployment-name>
kubectl describe service <service-name>
kubectl describe node <node-name>
kubectl describe pv <pv-name>

```

#### Logs & Debugging
```bash
# Logs anzeigen
kubectl logs <pod-name>
```

#### Erstellen & Anwenden (CREATE/APPLY)
```bash
# YAML-Datei anwenden
kubectl apply -f deployment.yaml
kubectl apply -f ./config-folder/         # Ganzer Ordner
kubectl apply -f https://raw.githubusercontent.com/.../file.yaml

```

#### Löschen (DELETE)
```bash
# Ressourcen löschen
kubectl delete pod <pod-name>
kubectl delete deployment <deployment-name>
kubectl delete service <service-name>

# Mit Datei
kubectl delete -f deployment.yaml

# Nach Label
kubectl delete pods -l app=nginx

# Alle Pods in Namespace
kubectl delete pods --all
kubectl delete all --all                 # VORSICHT: Löscht alles!

# Namespace löschen (löscht alle Ressourcen darin)
kubectl delete namespace <namespace-name>

# Force delete (bei hängenden Pods)
kubectl delete pod <pod-name> --grace-period=0 --force
```

---

### 🔷 oc Commands (OpenShift CLI)

**Hinweis:** `oc` basiert auf `kubectl` und hat alle kubectl-Befehle + OpenShift-spezifische Features.

#### Login & Authentifizierung
```bash
# Login in OpenShift Cluster
oc login <cluster-url>
oc login https://api.openshift.example.com:6443
oc login --token=<token> --server=<server-url>

# Login mit Username/Password
oc login -u <username> -p <password>

# Logout
oc logout

# Aktuellen User anzeigen
oc whoami
oc whoami --show-token                  # Token anzeigen
oc whoami --show-server                 # Server anzeigen
oc whoami --show-console                # Web-Console URL
```

#### Projekte (OpenShift Namespaces)
```bash
# Projekte anzeigen
oc projects
oc get projects

# Neues Projekt erstellen
oc new-project <project-name>
oc new-project dev-environment --description="Dev Env" --display-name="Development"

# Projekt wechseln
oc project <project-name>

# Aktuelles Projekt anzeigen
oc project

# Projekt löschen
oc delete project <project-name>
```

#### Routes (OpenShift Ingress)
```bash
# Routes anzeigen
oc get routes
oc get route <route-name>

# Route löschen
oc delete route <route-name>
```

#### Nützliche oc-spezifische Features
```bash
# Alle Ressourcen eines Projekts anzeigen
oc get all
```

#### Conversion: kubectl ↔ oc
```bash
# Diese sind identisch:
kubectl get pods        = oc get pods
kubectl apply -f        = oc apply -f
kubectl logs            = oc logs
kubectl exec            = oc exec

# OpenShift-spezifisch (kein kubectl-Äquivalent):
oc new-app
oc new-project
oc expose
oc start-build
oc login
```

---

#### Debugging-Workflow
```bash
# 1. Pod-Status prüfen
kubectl get pods

# 2. Details ansehen
kubectl describe pod <pod-name>

# 3. Logs prüfen
kubectl logs <pod-name>

# 4. Events prüfen
kubectl get events --sort-by='.lastTimestamp'

# 5. In Container gehen
kubectl exec -it <pod-name> -- /bin/sh

# 6. Netzwerk testen
kubectl run -it --rm debug --image=busybox --restart=Never -- sh
```

---

## 2. YAML Files für Kubernetes Components

### Grundaufbau einer Kubernetes YAML-Datei

```yaml
apiVersion: <version>
kind: <resource-type>
metadata:
  name: <resource-name>
  namespace: <namespace>
  labels:
    <key>: <value>
  annotations:
    <key>: <value>
spec:
  <resource-specific-configuration>
```

### Metadata Specifications

**Wichtige Metadata-Felder:**
- `name`: Eindeutiger Name der Ressource
- `namespace`: Logische Gruppierung (Standard: "default")
- `labels`: Key-Value-Paare für Selektion und Organisation
- `annotations`: Metadaten für Tools (nicht zur Selektion)
- `uid`: Automatisch generierte eindeutige ID
- `resourceVersion`: Interne Versionsnummer

**ConfigMap & Secrets:**
```yaml
# ConfigMap - für nicht-sensitive Daten
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "postgres://db:5432"
  log_level: "info"

# Secret - für sensitive Daten (base64-kodiert)
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=  # base64
  password: cGFzc3dvcmQ=
```

---

## 3. Kubernetes Components

### Pod
**Kleinste deploybare Einheit in Kubernetes**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
    env:
    - name: DB_HOST
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database_url
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

**Eigenschaften:**
- Enthält einen oder mehrere Container
- Teilen sich Netzwerk und Storage
- Vergänglich (Ephemeral)

---

### Deployment
**Verwaltet Replikation und Updates von Pods**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
```

**Eigenschaften:**
- Gewährleistet gewünschte Anzahl Replicas
- Ermöglicht Rolling Updates
- Self-Healing bei Pod-Ausfall

---

### Service
**Netzwerk-Abstraktion für Pod-Zugriff**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP  # LoadBalancer, NodePort, ClusterIP
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80        # Service Port
    targetPort: 80  # Container Port
```

**Service-Typen:**
- **ClusterIP**: Intern erreichbar (Standard)
- **NodePort**: Extern über Node-IP:Port
- **LoadBalancer**: Cloud Load Balancer
- **ExternalName**: DNS CNAME-Mapping

---

### ReplicaSet
**Stellt sicher, dass definierte Anzahl Pods läuft**

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
```

**Hinweis:** Wird meist durch Deployments verwaltet!

---

### StatefulSet
**Für stateful Anwendungen (z.B. Datenbanken)**

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:13
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

**Eigenschaften:**
- Stabile Netzwerk-Identitäten
- Persistente Storage
- Geordnetes Deployment/Scaling

---

### DaemonSet
**Stellt sicher, dass auf jedem Node ein Pod läuft**

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-agent
spec:
  selector:
    matchLabels:
      app: monitoring
  template:
    metadata:
      labels:
        app: monitoring
    spec:
      containers:
      - name: agent
        image: monitoring-agent:latest
```

**Verwendung:** Logging, Monitoring, Netzwerk-Plugins

---

### Namespace
**Logische Trennung von Ressourcen**

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
```

---

### Ingress
**HTTP/HTTPS Routing zu Services**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
```

---

### PersistentVolume (PV) & PersistentVolumeClaim (PVC)

```yaml
# PersistentVolume
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-data
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: standard
  hostPath:
    path: /mnt/data

---
# PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-data
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: standard
```

---

## 4. Cloud Computing - Grundlagen

### Wie ist der Begriff Cloud entstanden?
**Darstellung des Internets und komplexer Netzwerke als eine Wolke**

### NIST Definition der Cloud
Ein Modell, das den **bedarfsgerechten, netzwerkbasierten Zugriff** auf einen gemeinsamen Pool **konfigurierbarer IT-Ressourcen** ermöglicht, die **schnell bereitgestellt** und mit **minimalem Verwaltungsaufwand** genutzt werden können.

---

## 5. Die 5 Merkmale einer Cloud

1. **On-Demand Self-Service**
   - Selbstständige Ressourcen-Bereitstellung ohne menschliche Interaktion

2. **Breiter Netzwerkzugang**
   - Zugriff über Standard-Mechanismen (HTTP, APIs)
   - Von verschiedenen Geräten (Laptop, Smartphone, etc.)

3. **Ressourcen-Pooling**
   - Multi-Tenant-Modell
   - Ressourcen dynamisch zugewiesen
   - Standortunabhängigkeit

4. **Schnelle Elastizität**
   - Automatische Skalierung nach Bedarf
   - Erscheint als unbegrenzt verfügbar

5. **Gemessener Service (Metering)**
   - Ressourcen-Nutzung wird überwacht und gemessen
   - Pay-per-Use Modell

---

## 6. Cloud Dienstleistungen

- **Webhosting** - Hosting von Websites/Anwendungen
- **Datenbankdienste** - Managed Databases (MySQL, PostgreSQL, NoSQL)
- **Speicherlösungen** - Object Storage (S3), Block Storage
- **Virtuelle Maschinen** - Compute Instances
- **Backup & Recovery** - Automatisierte Sicherungen
- **KI-/ML-Dienste** - Machine Learning Platforms
- **Container-Services** - Kubernetes, Container Registry

---

## 7. Cloud Anbieter

- **AWS (Amazon Web Services)** - Marktführer
- **Microsoft Azure** - Enterprise-Fokus
- **Google Cloud Platform (GCP)** - KI/ML stark
- **IBM Cloud** - Hybrid/Enterprise
- **Oracle Cloud** - Datenbank-fokussiert

---

## 8. Cloud Deployment Modelle

| Modell | Beschreibung | Vorteile | Nachteile |
|--------|-------------|----------|-----------|
| **Public Cloud** | Öffentlich verfügbar, Multi-Tenant | Günstig, skalierbar | Weniger Kontrolle |
| **Private Cloud** | Dediziert für eine Organisation | Volle Kontrolle, Sicherheit | Teurer, weniger Skalierung |
| **Hybrid Cloud** | Kombination Public + Private | Flexibilität, Best of Both | Komplex zu verwalten |
| **Community Cloud** | Gemeinsam genutzt von mehreren Orgs | Geteilte Kosten | Eingeschränkte Kontrolle |

---

## 9. Cloud Service Modelle

```
┌─────────────────────────────────────────┐
│ SaaS - Software as a Service           │  ← Du verwaltest: NUR DATEN
│ (Gmail, Office 365, Salesforce)        │
├─────────────────────────────────────────┤
│ PaaS - Platform as a Service           │  ← Du verwaltest: Anwendung + Daten
│ (Heroku, Google App Engine)            │
├─────────────────────────────────────────┤
│ FaaS - Function as a Service           │  ← Du verwaltest: NUR CODE
│ (AWS Lambda, Azure Functions)          │
├─────────────────────────────────────────┤
│ IaaS - Infrastructure as a Service     │  ← Du verwaltest: OS, Runtime, App
│ (AWS EC2, Azure VMs)                   │
└─────────────────────────────────────────┘
```

**IaaS:** Virtuelle Maschinen, Netzwerk, Storage
**PaaS:** Entwicklungsplattform ohne Infrastruktur-Management
**SaaS:** Fertige Anwendung, nur nutzen
**FaaS:** Event-basierte Funktionen, Serverless

---

## 10. Monitoring vs. Logging

| Aspekt | Monitoring | Logging |
|--------|-----------|---------|
| **Fokus** | Leistung & Verfügbarkeit | Detaillierte Ereignisse |
| **Daten** | Metriken (CPU, RAM, Response Time) | Log-Einträge (Fehler, Warnungen) |
| **Zeitraum** | Echtzeit, Trends | Historisch, Analyse |
| **Zweck** | Alarmierung, Dashboards | Debugging, Audit |
| **Tools** | Prometheus, Grafana, CloudWatch | ELK Stack, Splunk, Loki |

**Zusammenfassung:**
- **Logging** ist detailliert (Was ist passiert?)
- **Monitoring** beobachtet die Leistung (Wie läuft es?)

---

## 11. Cloud - Vorteile & Nachteile

### ✅ Vorteile

- **Günstiger** - Keine Hardware-Investitionen, Pay-per-Use
- **Weniger Arbeit** - Kein Datacenter-Management
- **Ausfallsicherheit** - Redundanz, hohe Verfügbarkeit
- **Skalierbarkeit** - Elastisch nach Bedarf
- **Ortsunabhängiger Zugriff** - Von überall erreichbar

### ❌ Nachteile

- **Nicht volle Kontrolle** - Abhängig vom Anbieter
- **Abhängigkeit vom Anbieter** - Vendor Lock-in
- **Datenschutz-/Sicherheitsbedenken** - Daten bei Dritten
- **Internetverbindung notwendig** - Offline nicht nutzbar

---

## 12. Shared Responsibility Model

**Verteilung der Verantwortung auf Cloud-Anbieter und Kunden**

```
┌─────────────────────────────────────────┐
│ KUNDE verantwortlich für:              │
│ - Daten & Inhalte                      │
│ - Anwendungen                          │
│ - Zugriffsverwaltung (IAM)             │
│ - Verschlüsselung                      │
├─────────────────────────────────────────┤
│ GEMEINSAM:                             │
│ - Patch Management                     │
│ - Konfiguration                        │
├─────────────────────────────────────────┤
│ ANBIETER verantwortlich für:           │
│ - Physische Infrastruktur              │
│ - Netzwerk-Infrastruktur               │
│ - Hypervisor                           │
│ - Rechenzentrum-Sicherheit             │
└─────────────────────────────────────────┘
```

**Variiert je nach Service-Modell (IaaS vs. SaaS)**

---

## 13. Container-Technologie

### Was ist Container-Technologie?

- **Anwendungen und alle ihre Abhängigkeiten** in isolierte Softwarepakete verpacken
- **Leichtgewichtige Virtualisierung** auf Betriebssystem-Ebene
- **Nutzung des gleichen OS-Kernels**, aber getrennte Laufzeitumgebungen
- **Schnellere Bereitstellung** und Skalierbarkeit von Anwendungen

### Container vs. Virtuelle Maschinen

| Aspekt | Container | Virtuelle Maschine (VM) |
|--------|-----------|------------------------|
| **Bereitstellung** | Sekunden | Minuten |
| **Speicherplatz** | MB (nur App + Deps) | GB (komplettes OS) |
| **Portabilität** | Hoch (läuft überall) | Eingeschränkt (Hypervisor) |
| **Effizienz** | Höher (direkter Zugriff) | Geringer (Hypervisor-Overhead) |
| **Kernel** | Gemeinsamer Host-Kernel | Eigener Kernel pro VM |
| **Isolation** | Schwächer (Kernel geteilt) | Stärker (komplett getrennt) |
| **Startzeit** | < 1 Sekunde | 30-60 Sekunden |

### ✅ Vorteile Container

- Geringerer Ressourcenverbrauch
- Schnellere Startzeiten
- Höhere Portabilität (unabhängig von Infrastruktur)
- Bessere Skalierbarkeit

### ❌ Nachteile Container

- Schwächere Isolation (teilen sich den Kernel)
- Komplexere Netzwerkkonfiguration
- Geringerer Funktionsumfang bei vollständigen OS-Funktionen

### Können VMs immer durch Container ersetzt werden?

**NEIN!**
- Bei Bedarf für unterschiedliche OS (Windows + Linux)
- Bei strikten Sicherheitsanforderungen (vollständige Isolation)
- Legacy-Anwendungen mit OS-spezifischen Anforderungen

---

## 14. Produkte für VMs & Container

### Container
- **Docker** - Standard Container-Runtime
- **Kubernetes (K8s)** - Container-Orchestrierung
- **Podman** - Daemonless Container Engine
- **AWS ECS/EKS** - Amazon Container Services
- **Azure Container Instances** - Serverless Container
- **Google Kubernetes Engine (GKE)** - Managed Kubernetes

### Virtuelle Maschinen
- **VMware vSphere / ESXi** - Enterprise Virtualisierung
- **VirtualBox** - Desktop-Virtualisierung
- **AWS EC2** - Cloud VMs
- **Azure Virtual Machines** - Microsoft Cloud VMs
- **Google Compute Engine** - Google Cloud VMs

---

## 15. Self-Managed vs. Fully Managed

### Self-Managed

**Definition:**
- Benutzer ist verantwortlich für Betrieb, Wartung und Updates
- Volle Kontrolle über die Infrastruktur
- Höherer Aufwand bei Konfiguration und Sicherheit

**Beispiele:**
- Eigener Kubernetes-Cluster
- Selbst gehostete VMs
- On-Premise Datenbanken

### Fully Managed

**Definition:**
- Anbieter übernimmt Betrieb, Wartung, Monitoring und Skalierung
- Reduzierter Administrationsaufwand
- Eingeschränkte Anpassungsmöglichkeiten

**Beispiele:**
- AWS Fargate (Serverless Container)
- Google App Engine
- Azure App Services
- AWS RDS (Managed Database)

### Diskussion

**Entscheidungskriterien:**
- Verfügbare Ressourcen & Know-how
- Benötigte Flexibilität vs. Aufwand
- Kosten (OpEx vs. CapEx)
- Self-Managed: Mehr Kontrolle, aber aufwändiger
- Fully Managed: Schneller einsatzbereit, weniger Einfluss

---

## 16. Container-Orchestrierung

### Warum braucht man Container-Orchestrierung?

- **Automatisierte Verwaltung** vieler Container
- **Skalierung** (horizontal), Load Balancing, Self-Healing
- **Updates ohne Downtime** (Rolling Updates)
- **Effiziente Nutzung** von Ressourcen
- **Service Discovery** & Networking

### Wie funktioniert Container-Orchestrierung?

1. **Deklarative Konfiguration** (z.B. YAML)
   - Gewünschter Zustand wird definiert
   
2. **Scheduling auf Hosts**
   - Intelligente Verteilung auf verfügbare Nodes

3. **Service Discovery & Networking**
   - Container finden sich automatisch

4. **Lastverteilung (Load Balancing)**
   - Traffic wird verteilt

5. **Auto-Recovery (Self-Healing)**
   - Neustart bei Fehlern

6. **Horizontal Scaling**
   - Automatische Anpassung der Replicas

### Container-Orchestrierung Technologien

- **Kubernetes (K8s)** - Industry Standard, sehr mächtig
- **Docker Swarm** - Einfacher, in Docker integriert
- **AWS ECS/EKS** - Amazon-eigene Orchestrierung

---

## 17. Horizontale Skalierung

**Definition:**
Mehrere Instanzen der Anwendung statt größerer Server

### Eigenschaften

- **Mehrere kleinere Instanzen** statt einem großen Server
- **Bessere Lastverteilung** über viele Nodes
- **Höhere Verfügbarkeit** (Ausfall einzelner Instanzen verkraftbar)
- **Elastisch anpassbar** (schnell hoch-/runterskalieren)

### Horizontal vs. Vertikal

```
Horizontal (Scale Out):          Vertikal (Scale Up):
┌───┐ ┌───┐ ┌───┐               ┌───────┐
│ A │ │ A │ │ A │               │   A   │
└───┘ └───┘ └───┘               │       │
Mehr Server                      │       │
                                 └───────┘
                                 Größerer Server
```

**Vorteil Horizontal:** Besser für Cloud & Container!

---

## 18. Deployment Strategien

### 1. Rolling Update (Schrittweises Ersetzen)
```
Alt: [V1] [V1] [V1] [V1]
     [V2] [V1] [V1] [V1]
     [V2] [V2] [V1] [V1]
     [V2] [V2] [V2] [V1]
Neu: [V2] [V2] [V2] [V2]
```
- ✅ Keine Downtime
- ✅ Ressourcen-effizient
- ❌ Gemischte Versionen temporär

### 2. Blue-Green (Zwei Umgebungen, Umschaltung)
```
Blue (V1):  [V1] [V1] [V1]  ← Live Traffic
Green (V2): [V2] [V2] [V2]  ← Idle
            ↓
            Switch Traffic
            ↓
Blue (V1):  [V1] [V1] [V1]  ← Idle
Green (V2): [V2] [V2] [V2]  ← Live Traffic
```
- ✅ Sofortiges Rollback
- ✅ Keine Downtime
- ❌ Doppelte Ressourcen nötig

### 3. Canary (Test mit kleinem Nutzeranteil)
```
V1: [V1] [V1] [V1] [V1]  ← 90% Traffic
V2: [V2]                  ← 10% Traffic
Wenn OK → mehr Traffic zu V2
```
- ✅ Risikominimierung
- ✅ Echtes User-Feedback
- ❌ Komplexes Monitoring nötig

### 4. Recreate (Stop + Start, Downtime)
```
[V1] [V1] [V1]  → STOP → [V2] [V2] [V2]
      ↓
   Downtime!
```
- ✅ Einfach
- ❌ Downtime
- Nur für nicht-kritische Apps

### 5. A/B Testing (Vergleich verschiedener Versionen)
```
V1: [V1] [V1]  ← 50% User
V2: [V2] [V2]  ← 50% User
(basierend auf User-Segment, nicht zufällig)
```
- ✅ Feature-Testing
- ✅ Datengetriebene Entscheidungen
- ❌ Komplexe Analyse nötig

### 6. Shadow (Neuer Code erhält Traffic-Kopie)
```
V1: [V1]  ← Production Traffic
     │
     └─→ [V2]  ← Kopie des Traffics (keine Response an User)
```
- ✅ Testen ohne Risiko
- ✅ Realistische Last
- ❌ Doppelte Ressourcen

---

## 19. Fragen zu Block 7

*Was macht die Anwendung?*  
- Die Anwendung besteht aus einem Backend, das Zitate bereitstellt, einer React-Frontend-Seite, die die Zitate anzeigt, und einer MariaDB-Datenbank, in der die Zitate gespeichert sind.

*Welche Programmiersprache wird für das Backend verwendet?*  
- Python

*Welche Programmiersprache wird für das Frontend verwendet?*  
- React

---

*Wohin leitet die Route ihre Anfragen weiter?*  
- Die Route leitet ihre Anfragen über den Service an das entsprechende Pod weiter.

*Wie weiss ein Service, an welche Pods er Anfragen weiterleiten muss?*  
- Der Service verwendet Labels, die den Pods zugewiesen sind, um zu bestimmen, welche Pods die Anfragen erhalten sollen. Dies geschieht über Key-Value-Paare wie zum Beispiel app: quotes.

---

*Wer erstellt die Anfragen an das Backend? Das Frontend oder der Browser?*  
- Die Anfragen an das Backend werden vom Browser erstellt.

---

*Zu welcher Ressource leitet die Route die Anfrage weiter?*  
- Die Route leitet die Anfragen an den Service, der dann weiter an das passende Pod sendet.

*Wie weiss ein Service, welche Pods er ansprechen muss?*  
- Der Service verwendet Labels, die in den Pods definiert sind, um festzulegen, welche Pods die Anfragen erhalten. Diese Labels sind Key-Value-Paare wie z.B. app: quotes.

---

*Wer erstellt die Anfragen an das Backend? Das Frontend oder der Browser?*  
- Die Anfragen an das Backend werden vom Browser erstellt.

---

*Wie weiss das Backend, mit welcher Datenbank es sich verbinden soll?*  
- Das Backend verwendet eine Umgebungsvariable (DB_SERVICE_NAME), die den Namen des MariaDB-Dienstes enthält, um die Verbindung zur richtigen Datenbank herzustellen.

---

*Was macht eine HPA (Horizontal Pod Autoscaler)?*  
- Eine HPA skaliert die Anzahl der Pods basierend auf der Auslastung der Ressourcen, z.B. der CPU-Auslastung.

*Warum kann eine HPA nützlich sein?*  
- Eine HPA ermöglicht es, die Anwendung automatisch zu skalieren, wenn die Auslastung zunimmt, und sorgt so für bessere Verfügbarkeit und Leistung.

*Was könnte das Risiko beim Einsatz einer HPA sein?*  
- Ein Risiko beim Einsatz einer HPA besteht darin, dass eine zu aggressive Skalierung die Infrastruktur überlasten oder zusätzliche Kosten verursachen kann.
