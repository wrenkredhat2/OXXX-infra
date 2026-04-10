### 🏗️ RH-Demo Infrastructure Management Guide
#### (Przewodnik po Zarządzaniu Infrastrukturą RH-Demo)

This guide explains the project structure within the `components/` directory and provides step-by-step instructions on how to add and customize components across environments.

---

### 📂 Structure Overview (Przegląd Struktury)

The project follows a **Base & Overlay** pattern using Kustomize:

-   **`components/base/`**: Contains the "source of truth." These are generic Kubernetes manifests (Deployments, Secrets, Services) that should be reusable across all environments.
-   **`components/overlays/`**: Contains environment-specific configurations (`dev`, `stage`, `prod`). This is where we apply patches to modify names, hosts, or replicas.

---

### 🟢 1. Adding a New Component Base
#### (Dodawanie nowej bazy komponentu)

**English:**
1.  **Create Directory**: Create a new folder under `components/base/<component-name>`.
2.  **Add Manifests**: Place your YAML files (e.g., `deployment.yaml`, `service.yaml`) in that folder.
3.  **Create Kustomization**: Create a `kustomization.yaml` inside your new folder.
4.  **Register Base**: Add your new directory to the parent `components/base/kustomization.yaml`.

**Polski:**
1.  **Stwórz folder**: Utwórz nowy folder w `components/base/<nazwa-komponentu>`.
2.  **Dodaj manifesty**: Umieść pliki YAML (np. `deployment.yaml`) w tym folderze.
3.  **Stwórz Kustomization**: Utwórz plik `kustomization.yaml` w nowym folderze.
4.  **Zarejestruj bazę**: Dodaj ścieżkę do nowego folderu w głównym pliku `components/base/kustomization.yaml`.

**Example / Przykład:**
```yaml
# components/base/keycloak/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: keycloak
resources:
  - postgres-direct
  - instance
```

---

### 🔵 2. Customizing for Each Environment
#### (Dostosowanie dla każdego środowiska)

**English:**
To enable a component for an environment (e.g., `dev`), you must create an overlay that references the base and applies necessary patches.

1.  **Create Overlay Folder**: Go to `components/overlays/<env>/<component-name>`.
2.  **Reference Base**: In your `kustomization.yaml`, point to the base folder using relative paths.
3.  **Apply Patches**: Use `patches` to change environment-specific values like URLs, CPU/Memory, or Secret values.

**Polski:**
Aby aktywować komponent w danym środowisku (np. `dev`), musisz stworzyć nakładkę (overlay), która odwołuje się do bazy i nakłada poprawki.

1.  **Stwórz folder nakładki**: Przejdź do `components/overlays/<env>/<nazwa-komponentu>`.
2.  **Odnieś się do bazy**: W pliku `kustomization.yaml` wskaż folder bazy używając ścieżki względnej.
3.  **Zastosuj poprawki (Patches)**: Użyj sekcji `patches`, aby zmienić wartości specyficzne dla środowiska (URL, zasoby, dane w Secretach).

**Example Patch / Przykład Poprawki (`components/overlays/dev/rhbk/kustomization.yaml`):**
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: keycloak
resources:
  - ../../../base/keycloak
patches:
  - target:
      kind: Keycloak
      name: keycloak-instance
    patch: |-
      - op: replace
        path: /spec/hostname/hostname
        value: "keycloak-dev.example.com"
```

---

### ⚠️ Important Rules (Ważne Zasady)

1.  **Namespace Management**: 
    -   **EN**: Avoid setting a `namespace` in the top-level `components/overlays/<env>/kustomization.yaml`. This prevents naming conflicts between components (like `3scale` and `rhbk`) that might share resource names like `postgres-secret`. Set the namespace within the component's own overlay.
    -   **PL**: Unikaj ustawiania `namespace` w głównym pliku `components/overlays/<env>/kustomization.yaml`. Zapobiega to konfliktom nazw między komponentami (np. `3scale` i `rhbk`), które mogą używać tych samych nazw zasobów, np. `postgres-secret`. Ustawiaj namespace bezpośrednio w nakładce konkretnego komponentu.

2.  **Sync Waves**:
    -   **EN**: Use `argocd.argoproj.io/sync-wave` annotations for resources that must start in a specific order (e.g., Databases before Apps).
    -   **PL**: Używaj adnotacji `sync-wave` dla zasobów, które muszą być uruchamiane w konkretnej kolejności (np. Bazy danych przed Aplikacjami).

3.  **Keycloak Realm Import**:
    -   **EN**: When using Keycloak (RHBK), the `realm-import.yaml` (e.g., `platform-dev-realm-import.yaml`) contains the initial realm configuration. This file **must** be adjusted for each environment (URLs, client secrets, etc.) or you should provide your own configuration file.
    -   **PL**: Przy użyciu Keycloak (RHBK), plik `realm-import.yaml` (np. `platform-dev-realm-import.yaml`) zawiera początkową konfigurację realmu. Ten plik **musi** zostać dostosowany do każdego środowiska (adresy URL, client secrets itp.) lub należy dostarczyć własny plik konfiguracyjny.

---

### 🚀 How to deploy (Jak wdrożyć)
**EN**: Always verify your changes locally before pushing:
**PL**: Zawsze sprawdzaj zmiany lokalnie przed wysłaniem:

```bash
oc apply -k components/overlays/dev/ --dry-run=client
```
