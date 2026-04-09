# Kubernetes Hello World — Komplett Guide

En steg-for-steg guide for å sette opp en lokal Kubernetes-cluster med kind, deploye en Pod som serverer en Hello World-side, og eksponere den til nettleseren.

---

## Forutsetninger

- Windows 11
- PowerShell
- Internett-tilgang
- ~2 GB ledig RAM
- ~5 GB ledig diskplass

---

## Konsepter forklart på norsk

### Kubernetes
"Dirigent for containere" — et system som styrer hvordan containere startes, stoppes, restart-es, og kommuniserer. Designet for å kjøre applikasjoner i stor skala.

### Container
En "frossen app" — alle filer, avhengigheter og konfigurasjon pakket sammen i et lag som kan kjøres hvor som helst Docker er installert.

### Docker
Programvaren som faktisk kjører containere på maskinen din. Alle andre verktøy (kind, kubectl) bygger på Docker.

### kind (Kubernetes IN Docker)
Et verktøy som lager en mini-Kubernetes-cluster inne i Docker-containere. Perfekt for lokal utvikling og læring uten å måtte betale for en skyhostet Kubernetes.

### kubectl
Kommandolinjeverktøyet for å snakke med Kubernetes. "kube control". Brukes til å deploye, liste, slette og endre Kubernetes-ressurser.

### Pod
Minste enhet i Kubernetes. En wrapper rundt én eller flere containere som hører sammen. Får sin egen IP-adresse og kan administreres av Kubernetes.

### YAML manifest
En fil som beskriver hva du vil ha i Kubernetes. Deklarativ syntaks — du sier HVA du vil ha, ikke HVORDAN det skal lages. Samme prinsipp som Terraform.

### Port forward
En midlertidig tunnel fra din lokale maskin til en Pod. Brukes til testing og debugging. I produksjon bruker man Services og Ingress i stedet.

---

## Installasjon

### Steg 1: Docker Desktop

1. Gå til https://www.docker.com/products/docker-desktop/
2. Last ned for Windows AMD64
3. Kjør installer
4. Restart maskinen
5. Start Docker Desktop fra Start-menyen
6. Vent til den sier "Docker Desktop is running" nederst

Test:
```powershell
docker --version
```

### Steg 2: Lag verktøy-mappe og last ned kind

```powershell
New-Item -ItemType Directory -Force -Path "C:\tools"
curl.exe -Lo C:\tools\kind.exe https://kind.sigs.k8s.io/dl/v0.20.0/kind-windows-amd64
```

### Steg 3: Last ned kubectl

```powershell
curl.exe -Lo C:\tools\kubectl.exe https://dl.k8s.io/release/v1.29.0/bin/windows/amd64/kubectl.exe
```

### Steg 4: Legg C:\tools til PATH permanent

1. Søk "environment variables" i Windows Start
2. Klikk "Edit the system environment variables"
3. Klikk "Environment Variables..."
4. Under "User variables", finn **Path**, klikk **Edit**
5. Klikk **New** → skriv `C:\tools`
6. OK → OK → OK
7. Lukk og åpne PowerShell på nytt

Test:
```powershell
kind --version
kubectl version --client
```

---

## Sette opp Kubernetes-cluster

### Opprett cluster
```powershell
kind create cluster --name hello-world
```

Dette tar 1-3 minutter første gang. kind laster ned et Kubernetes node-image og spinner opp en Docker-container som kjører hele Kubernetes-kontrollplanen.

### Verifiser at clusteren kjører
```powershell
kubectl cluster-info --context kind-hello-world
kubectl get nodes
```

Du skal se én node med status `Ready`.

---

## Deploye Hello World-Pod

### Opprett prosjektmappe
```powershell
New-Item -ItemType Directory -Force -Path "C:\projects\kubernetes-hello"
cd C:\projects\kubernetes-hello
```

### Lag YAML-manifest

Opprett filen `hello-world-pod.yaml` med dette innholdet:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-world-pod
  labels:
    app: hello-world
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

**Forklaring av feltene:**

| Felt | Betydning |
|------|-----------|
| `apiVersion: v1` | Bruker Kubernetes core API (grunnleggende objekter) |
| `kind: Pod` | Vi lager en Pod-ressurs |
| `metadata.name` | Navnet på Pod-en |
| `metadata.labels` | Etiketter for å finne/gruppere Pod-en senere |
| `spec.containers` | Liste over containere i Pod-en |
| `image: nginx:latest` | Docker-image som skal kjøres (fra Docker Hub) |
| `containerPort: 80` | Port nginx lytter på inne i containeren |

### Deploy Pod-en
```powershell
kubectl apply -f hello-world-pod.yaml
```

### Sjekk status
```powershell
kubectl get pods
```

Vent til STATUS viser `Running` (kan ta 15-30 sekunder første gang mens nginx-image lastes ned).

```powershell
kubectl describe pod hello-world-pod
```

Denne gir detaljert info hvis noe feiler.

---

## Eksponere Pod-en til nettleseren

### Port forward
```powershell
kubectl port-forward pod/hello-world-pod 8080:80
```

Dette lager en tunnel:
- Trafikk til `localhost:8080` på din maskin
- Videresendes til port 80 inne i Pod-en
- Hvor nginx lytter

**VIKTIG:** La dette PowerShell-vinduet være åpent. Trykk Ctrl+C for å stoppe.

### Åpne nettleseren
Naviger til: http://localhost:8080

Du vil se "Welcome to nginx!"-siden.

---

## Rydde opp

### Stopp port-forward
Trykk `Ctrl+C` i PowerShell-vinduet som kjører port-forward.

### Slett Pod
```powershell
kubectl delete -f hello-world-pod.yaml
```

### Slett hele clusteren
```powershell
kind delete cluster --name hello-world
```

Dette fjerner alt kind laget. Docker rydder opp containerne automatisk.

---

## Kommando-referanse

### Docker
| Kommando | Hva den gjør |
|----------|-------------|
| `docker --version` | Viser Docker-versjon |
| `docker ps` | Lister kjørende containere |
| `docker images` | Lister lokale images |

### kind
| Kommando | Hva den gjør |
|----------|-------------|
| `kind create cluster --name <navn>` | Lager ny cluster |
| `kind get clusters` | Lister alle kind-clustere |
| `kind delete cluster --name <navn>` | Sletter cluster |

### kubectl
| Kommando | Hva den gjør |
|----------|-------------|
| `kubectl cluster-info` | Viser info om aktuell cluster |
| `kubectl get nodes` | Lister noder |
| `kubectl get pods` | Lister pods i default namespace |
| `kubectl get pods -A` | Lister pods i alle namespaces |
| `kubectl apply -f <fil>` | Oppretter/oppdaterer ressurser fra YAML |
| `kubectl delete -f <fil>` | Sletter ressurser definert i YAML |
| `kubectl describe pod <navn>` | Detaljert info om Pod |
| `kubectl logs <pod-navn>` | Viser logs fra containeren |
| `kubectl exec -it <pod> -- /bin/bash` | Åpner shell i containeren |
| `kubectl port-forward <pod> <lokal>:<container>` | Lager tunnel |

---

## Feilsøking

### Pod stuck i ContainerCreating
Første gang tar det tid å laste ned nginx-image. Vent 1-2 minutter.

```powershell
kubectl describe pod hello-world-pod
```

Se i `Events`-seksjonen nederst for hva som skjer.

### Connection refused på localhost:8080
Port-forward er ikke aktiv. Sjekk at PowerShell-vinduet kjører og viser "Forwarding from...". Hvis ikke, start port-forward på nytt.

### Port 8080 er opptatt
Bruk en annen port:
```powershell
kubectl port-forward pod/hello-world-pod 9090:80
```

Og åpne `http://localhost:9090` i stedet.

### kind kommandoer feiler
Sjekk at Docker Desktop kjører. Se etter hvalen i systemtrayen.

### PATH-problemer
Etter å ha lagt til C:\tools i PATH, må du åpne et NYTT PowerShell-vindu. Gamle vinduer bruker gammel PATH.

---

## Hva du har lært

- **Kubernetes-konsepter**: Pods, containers, manifests, cluster, kubectl
- **kind**: Lokal Kubernetes for læring og testing
- **YAML**: Deklarativ ressursdefinisjon
- **Port-forward**: Eksponere Pods for lokal testing
- **Container-images**: Hente og kjøre ferdige images fra Docker Hub
- **Kommandolinjeverktøy**: docker, kubectl, kind

---

## Til intervju — hva du kan si

*"Jeg har satt opp en lokal Kubernetes-cluster med kind, som kjører en mini-Kubernetes inne i Docker-containere. Jeg skrev en Pod-manifest i YAML som starter en nginx-container, og eksponerte den til min vertsmaskin med kubectl port-forward. Det er en enkel måte å lære og teste Kubernetes-konsepter lokalt uten å betale for en skyhostet cluster. I produksjon ville jeg brukt Services og Ingress i stedet for port-forward, og kjørt det på AKS i Azure."*

---

## Videre læring

Neste steg hvis du vil gå dypere:

1. **Services** — abstraksjon over Pods som gir stabil DNS og load balancing
2. **Deployments** — kjør flere kopier av samme Pod for høy tilgjengelighet
3. **Ingress** — eksponere tjenester med HTTP-routing og TLS
4. **ConfigMaps og Secrets** — injiser konfigurasjon og hemmeligheter
5. **AKS** — Azure Kubernetes Service for produksjonsbruk
