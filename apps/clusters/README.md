### ☸️ RH-Demo Cluster & GitOps Management Guide
#### (Przewodnik po Zarządzaniu Klastrami i GitOps w RH-Demo)

This guide explains the project structure within the `clusters/` and `bootstrap/` directories and provides instructions on how to manage environment-level configurations using the **App-of-Apps** pattern in ArgoCD.

---

### 📂 Structure Overview (Przegląd Struktury)

The project uses a hierarchical ArgoCD structure to manage infrastructure and applications:

1.  **`bootstrap/`**: Contains the "Root" Applications. These are the entry points that tell ArgoCD where to find the cluster-level configurations (e.g., `platform-dev.yaml` points to `clusters/dev/infra`).
2.  **`clusters/<env>/`**: Contains environment-specific ArgoCD manifests.
    -   **`infra/`**: Defines which infrastructure components (from `components/overlays/<env>`) should be deployed to this cluster.
    -   **`apps/`**: Defines which application groups (from `apps/<team>/overlays/<env>`) should be deployed to this cluster.

---

### 🟢 1. Adding a New Application to a Cluster
#### (Dodawanie nowej aplikacji do klastra)

**English:**
To add a new application (e.g., for a new team or service) to an existing cluster:
1.  **Create Manifest**: Create a new ArgoCD `Application` YAML in `clusters/<env>/apps/` (e.g., `my-service.yaml`).
2.  **Point to Overlay**: Set `spec.source.path` to the corresponding overlay directory in `apps/`.
3.  **Register in Kustomization**: Add the new YAML file to `clusters/<env>/apps/kustomization.yaml`.

**Polski:**
Aby dodać nową aplikację (np. dla nowego zespołu lub usługi) do istniejącego klastra:
1.  **Stwórz manifest**: Utwórz nowy plik YAML ArgoCD `Application` w `clusters/<env>/apps/` (np. `my-service.yaml`).
2.  **Wskaż Overlay**: Ustaw `spec.source.path` na odpowiedni katalog overlay w `apps/`.
3.  **Zarejestruj w Kustomization**: Dodaj nowy plik YAML do `clusters/<env>/apps/kustomization.yaml`.

**Example / Przykład (`clusters/dev/apps/new-app.yaml`):**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: new-app
  namespace: openshift-gitops
spec:
  project: default
  source:
    repoURL: https://git.svc.arencloud.com/egevorky/rhdemo.git
    targetRevision: HEAD
    path: apps/team-a/overlays/dev
  destination:
    server: https://kubernetes.default.svc
```

---

### 🔵 2. Managing Infrastructure Components
#### (Zarządzanie komponentami infrastruktury)

**English:**
Infrastructure components (like Keycloak, Kafka, 3Scale) are managed as a single logical group per environment.
1.  The `infra/` folder in each cluster directory contains an `Application` that points to `components/overlays/<env>`.
2.  To add a new component to the cluster, ensure it is registered in `components/overlays/<env>/kustomization.yaml`. ArgoCD will automatically detect and sync it.

**Polski:**
Komponenty infrastruktury (takie jak Keycloak, Kafka, 3Scale) są zarządzane jako jedna logiczna grupa na środowisko.
1.  Folder `infra/` w każdym katalogu klastra zawiera zasób `Application`, który wskazuje na `components/overlays/<env>`.
2.  Aby dodać nowy komponent do klastra, upewnij się, że jest on zarejestrowany w `components/overlays/<env>/kustomization.yaml`. ArgoCD automatycznie go wykryje i zsynchronizuje.

---

### 🟡 3. Bootstrapping a New Environment
#### (Uruchamianie nowego środowiska)

**English:**
1.  **Create Cluster Folders**: Create `clusters/<new-env>/infra` and `clusters/<new-env>/apps`.
2.  **Define Applications**: Create the `Application` manifests in these folders pointing to the correct overlays.
3.  **Create Bootstrap Manifest**: Create a new file in `bootstrap/` (e.g., `platform-<new-env>.yaml`) that points to `clusters/<new-env>/infra`.
4.  **Apply Bootstrap**: Run `oc apply -f bootstrap/platform-<new-env>.yaml`.

**Polski:**
1.  **Stwórz foldery klastra**: Utwórz `clusters/<new-env>/infra` oraz `clusters/<new-env>/apps`.
2.  **Zdefiniuj aplikacje**: Utwórz manifesty `Application` w tych folderach, wskazując na odpowiednie nakładki (overlays).
3.  **Stwórz manifest Bootstrap**: Utwórz nowy plik w `bootstrap/` (np. `platform-<new-env>.yaml`), który wskazuje na `clusters/<new-env>/infra`.
4.  **Uruchom Bootstrap**: Uruchom komendę `oc apply -f bootstrap/platform-<new-env>.yaml`.

---

### ⚠️ Important Rules (Ważne Zasady)

1.  **Namespace**: Most ArgoCD `Application` resources should be deployed into the `openshift-gitops` namespace so the operator can manage them.
2.  **Self-Healing**: Automated sync with `prune: true` and `selfHeal: true` is recommended for cluster-level applications to maintain environment consistency.
3.  **Path Accuracy**: Always double-check that the `path` in your `Application` manifest correctly points to an existing Kustomize overlay.

---

### 🚀 How to verify (Jak sprawdzić)
**EN**: Check if your cluster manifests are valid:
**PL**: Sprawdź, czy manifesty klastra są poprawne:

```bash
oc apply -k clusters/dev/apps/ --dry-run=client
oc apply -k clusters/dev/infra/ --dry-run=client
```
