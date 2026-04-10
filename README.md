# 🚀 RH-Demo GitOps Project Guide
## (Przewodnik po Projekcie RH-Demo GitOps)

Welcome to the **RH-Demo** project. This repository is a centralized GitOps management hub for infrastructure and application delivery using **OpenShift**, **ArgoCD**, **Kustomize**, **3Scale**, **Keycloak (RHBK)**, and **Kafka**.

Witamy w projekcie **RH-Demo**. To repozytorium jest scentralizowanym centrum zarządzania GitOps dla infrastruktury i dostarczania aplikacji przy użyciu **OpenShift**, **ArgoCD**, **Kustomize**, **3Scale**, **Keycloak (RHBK)** oraz **Kafka**.

---

### 🗺️ Project Architecture (Architektura Projektu)

The project follows the **"App-of-Apps"** pattern. ArgoCD manages root applications that in turn manage the state of your infrastructure and business applications across multiple environments (`dev`, `stage`, `prod`).

Projekt opiera się na wzorcu **"App-of-Apps"**. ArgoCD zarządza głównymi aplikacjami, które z kolei zarządzają stanem infrastruktury i aplikacji biznesowych w wielu środowiskach (`dev`, `stage`, `prod`).

#### 📁 Directory Structure (Struktura Katalogów)

| Directory / Katalog | Description / Opis | Documentation / Dokumentacja |
| :--- | :--- | :--- |
| **`bootstrap/`** | Entry point for ArgoCD. Root applications for environments. | [Read more (PL/EN)](bootstrap/README.md) |
| **`clusters/`** | Cluster-level configuration (Infra & Apps definitions). | [Read more (PL/EN)](clusters/README.md) |
| **`components/`** | Infrastructure components (Keycloak, Kafka, 3Scale). | [Read more (PL/EN)](components/README.md) |
| **`apps/`** | Business applications (Quarkus-Camel, 3Scale CRs). | [Read more (PL/EN)](apps/README.md) |

---

### 🔄 GitOps Workflow (Przepływ GitOps)

**English:**
1.  **Bootstrap**: Apply the root YAML from `bootstrap/` to OpenShift.
2.  **Clusters**: The bootstrap app points to `clusters/<env>/`, which defines what infrastructure and apps should exist.
3.  **Components/Apps**: The cluster definitions point to specific overlays in `components/` or `apps/`, applying environment-specific patches.

**Polski:**
1.  **Bootstrap**: Zastosuj główny plik YAML z `bootstrap/` na OpenShift.
2.  **Clusters**: Aplikacja bootstrap wskazuje na `clusters/<env>/`, który definiuje, jakie komponenty infrastruktury i aplikacje powinny istnieć.
3.  **Components/Apps**: Definicje klastra wskazują na konkretne nakładki (overlays) w `components/` lub `apps/`, nakładając poprawki specyficzne dla środowiska.

---

### 🛠️ Quick Start (Szybki Start)

**EN**: To initialize the development environment:
**PL**: Aby zainicjować środowisko deweloperskie:

```bash
# 1. Login to OpenShift / Zaloguj się do OpenShift
oc login --token=... --server=...

# 2. Bootstrap Infrastructure / Zainicjuj Infrastrukturę
oc apply -f bootstrap/platform-dev.yaml

# 3. Bootstrap Applications / Zainicjuj Aplikacje
oc apply -f bootstrap/apps-dev.yaml
```

---

### 📜 Documentation Index (Indeks Dokumentacji)

-   **Infrastructure (Infrastruktura)**: Learn how to manage Keycloak, Kafka, and 3Scale in [components/README.md](components/README.md).
-   **Applications (Aplikacje)**: Learn how to onboard Quarkus-Camel apps and 3Scale APIs in [apps/README.md](apps/README.md).
-   **Cluster Management (Zarządzanie Klastrem)**: Learn how to add new environments or apps to clusters in [clusters/README.md](clusters/README.md).
-   **Root Management (Zarządzanie Główne)**: Learn about the entry points in [bootstrap/README.md](bootstrap/README.md).

---

### ⚠️ Best Practices (Dobre Praktyki)

-   **No Namespaces in Overlays**: Always set namespaces in the specific component/app overlay, never in the global environment `kustomization.yaml`.
-   **Dry Run**: Always test your Kustomize changes locally before pushing:
    `oc apply -k <path> --dry-run=client`
-   **Sync Waves**: Use sync-wave annotations to ensure databases start before applications.
