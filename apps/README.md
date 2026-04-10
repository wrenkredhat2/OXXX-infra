### 🚀 RH-Demo Applications & 3Scale Onboarding Guide
#### (Przewodnik po Wdrażaniu Aplikacji i Integracji z 3Scale w RH-Demo)

This guide describes how to onboard new Quarkus-Camel applications and manage their integration with 3Scale API Management using Kustomize and the 3Scale Operator.

---

### 📂 Structure Overview (Przegląd Struktury)

The application configuration follows a **Base & Overlay** pattern:

-   **`apps/base/`**: Contains the "source of truth." These are generic Kubernetes manifests and 3Scale Custom Resources (CRs) that serve as templates for all apps.
-   **`apps/team-a/overlays/`**: Contains environment-specific configurations (`dev`, `stage`, `prod`) for a specific team. This is where you apply patches to customize names, images, namespaces, and 3Scale settings.

---

### 🟢 1. Onboarding a New Quarkus-Camel App
#### (Wdrażanie nowej aplikacji Quarkus-Camel)

**English:**
1.  **Define Base**: Ensure `apps/base/deployment.yaml` and `apps/base/service.yaml` contain the standard configuration for your Quarkus-Camel apps.
2.  **Create Overlay**: Create a new folder for your team (e.g., `apps/team-b/overlays/dev/`).
3.  **Customize Deployment**: In your `kustomization.yaml`, reference the base and use `patches` to set the unique name and container image for your app.

**Polski:**
1.  **Zdefiniuj bazę**: Upewnij się, że pliki `apps/base/deployment.yaml` i `apps/base/service.yaml` zawierają standardową konfigurację dla aplikacji Quarkus-Camel.
2.  **Stwórz nakładkę (Overlay)**: Utwórz nowy folder dla swojego zespołu (np. `apps/team-b/overlays/dev/`).
3.  **Dostosuj wdrożenie (Deployment)**: W pliku `kustomization.yaml` odnieś się do bazy i użyj sekcji `patches`, aby ustawić unikalną nazwę oraz obraz kontenera dla swojej aplikacji.

**Example Patch / Przykład Poprawki:**
```yaml
# apps/team-a/overlays/dev/kustomization.yaml
patches:
  - target:
      kind: Deployment
      name: app-camel
    patch: |-
      - op: replace
        path: /metadata/name
        value: my-new-app
      - op: replace
        path: /spec/template/spec/containers/0/image
        value: quay.io/my-org/my-app:v1.0
```

---

### 🌐 2. 3Scale API Management Onboarding
#### (Integracja z Zarządzaniem API 3Scale)

**English:**
3Scale onboarding is managed via Custom Resources (CRs) that the 3Scale Operator synchronizes to the API Manager.

1.  **Backend (`Backend` CR)**: Defines your internal service URL.
    - Path: `apps/base/backend-orders.yaml`
2.  **Product (`Product` CR)**: Defines the public API, including authentication (OIDC/Keycloak) and mapping rules.
    - Path: `apps/base/product-orders.yaml`
3.  **Application (`Application` CR)**: Defines the subscription and plan for a specific consumer.
    - Path: `apps/base/application-orders-ui.yaml`

**Polski:**
Integracja z 3Scale jest zarządzana przez zasoby Custom Resources (CR), które operator 3Scale synchronizuje z API Managerem.

1.  **Backend (CR `Backend`)**: Definiuje wewnętrzny URL Twojej usługi.
    - Ścieżka: `apps/base/backend-orders.yaml`
2.  **Product (CR `Product`)**: Definiuje publiczne API, w tym uwierzytelnianie (OIDC/Keycloak) i reguły mapowania.
    - Ścieżka: `apps/base/product-orders.yaml`
3.  **Application (CR `Application`)**: Definiuje subskrypcję i plan dla konkretnego konsumenta.
    - Ścieżka: `apps/base/application-orders-ui.yaml`

---

### 🔵 3. Customizing 3Scale for Environments
#### (Dostosowanie 3Scale dla różnych środowisk)

**English:**
Since different environments have different internal URLs and Keycloak endpoints, you must patch the 3Scale CRs in your overlay.

1.  **Patch Private Base URL**: Update the `Backend` CR to point to the service in the correct namespace.
2.  **Patch Public Base URL**: Update the `Product` CR with the environment-specific public route.
3.  **Patch Keycloak Endpoint**: Update the `keycloak-issuer` secret with the correct realm URL.

**Polski:**
Ponieważ różne środowiska mają różne wewnętrzne adresy URL i punkty końcowe Keycloak, musisz zmodyfikować zasoby 3Scale w swojej nakładce (overlay).

1.  **Patch Private Base URL**: Zaktualizuj CR `Backend`, aby wskazywał na usługę w odpowiednim namespace.
2.  **Patch Public Base URL**: Zaktualizuj CR `Product` o publiczną trasę specyficzną dla środowiska.
3.  **Patch Keycloak Endpoint**: Zaktualizuj sekret `keycloak-issuer` o poprawny URL dla danego realmu.

**Example Patch / Przykład Poprawki:**
```yaml
# apps/team-a/overlays/dev/kustomization.yaml
patches:
  - target:
      kind: Backend
      name: orders-backend
    patch: |-
      - op: replace
        path: /spec/privateBaseURL
        value: http://orders-backend.team-a-dev.svc.cluster.local
```

---

### ⚠️ Important Rules (Ważne Zasady)

1.  **Sync Waves**:
    -   **EN**: 3Scale resources use `argocd.argoproj.io/sync-wave` to ensure correct order: `Backend` (1) -> `Product` (2) -> `Application` (3).
    -   **PL**: Zasoby 3Scale używają `sync-wave`, aby zapewnić poprawną kolejność: `Backend` (1) -> `Product` (2) -> `Application` (3).
2.  **Naming Consistency**:
    -   **EN**: Ensure the `systemName` in `Product` matches the `backendUsage` reference to the `Backend` system name.
    -   **PL**: Upewnij się, że `systemName` w zasobie `Product` zgadza się z referencją w `backendUsage` do nazwy systemowej `Backend`.

---

### 🚀 How to deploy (Jak wdrożyć)
**EN**: Verify the generated manifests for your team:
**PL**: Sprawdź wygenerowane manifesty dla swojego zespołu:

```bash
oc apply -k apps/team-a/overlays/dev/ --dry-run=client
```
