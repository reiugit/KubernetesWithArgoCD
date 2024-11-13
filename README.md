# Kubernetes with ArgoCD

### argocd-my-app.yaml

In the 'argocd-my-app.yaml' file, the repoURL and path specifiy, where the Kubernetes deployment definition files (mere my-app-k8s.yaml) are located.

<pre>
apiVersion: argoproj.io/v1alpha1
kind: Application # 'Application' (Anwendung, die in ArgoCD verwaltet wird)
metadata:
  name: my-app # Name der Anwendung, wird in der ArgoCD-Oberfläche angezeigt
  namespace: argocd # ArgoCD Namespace
spec:
  project: default # Projekt, in dem die Anwendung definiert ist (default ist das Standardprojekt)

  # Quelle des Repositories, in dem die Anwendung definiert ist
  source:
    repoURL: "https://github.com/benutzername/my-repo.git" # URL des Git-Repositorys
    targetRevision: HEAD # oder zB. 'main', 'master' oder spezifische Branches
    path: manifests # Pfad zum Ordner, in dem die YAML-Dateien liegen, könnte auch zB. 'k8s' sein

  # Zielcluster und Zielnamespace
  destination:
    server: "https://kubernetes.default.svc" # Lokales Cluster, kann auch ein externer Cluster sein
    namespace: my-app-namespace # Namespace für die Anwendung, muss mit dem Service und dem Deployment übereinstimmen

  # Synchronisierungsrichtlinien
  syncPolicy:
    automated: # Automatische Synchronisierung
      prune: true # Lösche Ressourcen, die nicht mehr in der Quelle definiert sind
      selfHeal: true # Stelle die Anwendung automatisch wieder her, wenn sie nicht mehr im gewünschten Zustand ist
      #zB. wenn ein pod manuell geändert wurde, wird er automatisch wieder korrigiert
</pre>

### my-app-k8s.yaml

<pre>
#1. Deployment

apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment # Name des Deployments, 
  namespace: my-app-namespace # Namespace des Deployments, muss mit dem Service übereinstimmen
spec:
  replicas: 3 # Anzahl der Replikate
  selector:
    matchLabels: # Selektiert den Pod, der vom Deployment verwaltet wird
      app: my-app-pod # Ziel-Label des Pods
  template: # Pod-Template
    metadata:
      labels: # Label des Pods, um den Pod selektieren zu können im Deployment und im Service
        app: my-app-pod # Label des Pods
    spec:
      containers: # Container-Definition innerhalb des Pods
      - name: my-app-container # Name des Containers
        image: my-app-image:latest # Image des Containers
        ports:
        - containerPort: 8080 # Port des Containers

#2. Service

apiVersion: v1
kind: Service # Service macht den Pod über eine Netzwerkverbindung zugänglich
metadata:
  name: my-app-service # Name des Services
  namespace: my-app-namespace # Namespace des Services, muss mit dem Deployment übereinstimmen
spec: # Service-Definition
  selector: # Selektiert den Ziel-Pod
    app: my-app-pod # Label des Ziel-Pods
  ports: # Port-Definition
  - protocol: TCP # Protokoll des Services, z.B. TCP oder UDP
    port: 80 # Port des Services nach außen
    targetPort: 8080 # Port des Ziel-Containers
  type: LoadBalancer # Typ des Services, um externe Zugriffe zu ermöglichen
</pre>
