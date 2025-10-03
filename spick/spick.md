# Kubernetes & Cloud Computing - Prüfungsspick

---

## 1. YAML Files für Kubernetes Components

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

## 2. Kubernetes Components

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

## 3. Cloud Computing - Grundlagen

### Wie ist der Begriff Cloud entstanden?
**Darstellung des Internets und komplexer Netzwerke als eine Wolke**

### NIST Definition der Cloud
Ein Modell, das den **bedarfsgerechten, netzwerkbasierten Zugriff** auf einen gemeinsamen Pool **konfigurierbarer IT-Ressourcen** ermöglicht, die **schnell bereitgestellt** und mit **minimalem Verwaltungsaufwand** genutzt werden können.

---

## 4. Die 5 Merkmale einer Cloud

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

## 5. Cloud Dienstleistungen

- **Webhosting** - Hosting von Websites/Anwendungen
- **Datenbankdienste** - Managed Databases (MySQL, PostgreSQL, NoSQL)
- **Speicherlösungen** - Object Storage (S3), Block Storage
- **Virtuelle Maschinen** - Compute Instances
- **Backup & Recovery** - Automatisierte Sicherungen
- **KI-/ML-Dienste** - Machine Learning Platforms
- **Container-Services** - Kubernetes, Container Registry

---

## 6. Cloud Anbieter

- **AWS (Amazon Web Services)** - Marktführer
- **Microsoft Azure** - Enterprise-Fokus
- **Google Cloud Platform (GCP)** - KI/ML stark
- **IBM Cloud** - Hybrid/Enterprise
- **Oracle Cloud** - Datenbank-fokussiert

---

## 7. Cloud Deployment Modelle

| Modell | Beschreibung | Vorteile | Nachteile |
|--------|-------------|----------|-----------|
| **Public Cloud** | Öffentlich verfügbar, Multi-Tenant | Günstig, skalierbar | Weniger Kontrolle |
| **Private Cloud** | Dediziert für eine Organisation | Volle Kontrolle, Sicherheit | Teurer, weniger Skalierung |
| **Hybrid Cloud** | Kombination Public + Private | Flexibilität, Best of Both | Komplex zu verwalten |
| **Community Cloud** | Gemeinsam genutzt von mehreren Orgs | Geteilte Kosten | Eingeschränkte Kontrolle |

---

## 8. Cloud Service Modelle

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

## 9. Monitoring vs. Logging

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

## 10. Cloud - Vorteile & Nachteile

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

## 11. Shared Responsibility Model

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

## 12. Container-Technologie

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

## 13. Produkte für VMs & Container

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

## 14. Self-Managed vs. Fully Managed

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

## 15. Container-Orchestrierung

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

## 16. Horizontale Skalierung

**Definition:**
Mehrere Instanzen der Anwendung statt grösserer Server

### Eigenschaften

- **Mehrere kleinere Instanzen** statt einem grossen Server
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
                                 Grösserer Server
```

**Vorteil Horizontal:** Besser für Cloud & Container!

---

## 17. Deployment Strategien

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

## Wichtige kubectl Befehle (Bonus)

```bash
# Ressourcen anzeigen
kubectl get pods
kubectl get services
kubectl get deployments

# Detaillierte Informationen
kubectl describe pod <pod-name>

# Logs anzeigen
kubectl logs <pod-name>

# In Container ausführen
kubectl exec -it <pod-name> -- /bin/bash

# YAML anwenden
kubectl apply -f deployment.yaml

# Löschen
kubectl delete pod <pod-name>

# Skalieren
kubectl scale deployment <name> --replicas=5

# Rollout-Status
kubectl rollout status deployment/<name>

# Rollback
kubectl rollout undo deployment/<name>
```

---

## Zusammenfassung: Wichtigste Punkte

✅ **YAML-Struktur:** apiVersion, kind, metadata, spec
✅ **5 Cloud-Merkmale:** On-Demand, Netzwerk, Pooling, Elastizität, Metering
✅ **Service-Modelle:** IaaS, PaaS, SaaS, FaaS
✅ **Container > VM:** Schneller, leichter, portabler (aber schwächere Isolation)
✅ **Orchestrierung:** Automatisierung, Skalierung, Self-Healing
✅ **Deployment:** Rolling Update am häufigsten, Blue-Green für kritische Apps

---
