### 🏁 RH-Demo Bootstrap & Root Applications Guide
#### (Przewodnik po Inicjalizacji i Głównych Aplikacjach RH-Demo)

The `bootstrap/` directory contains the "Root" ArgoCD Applications. These manifests serve as the entry points for the entire GitOps infrastructure, implementing the **App-of-Apps** pattern.

---

### 📂 Structure Overview (Przegląd Struktury)

Root applications in `bootstrap/` do not contain service manifests themselves. Instead, they point to other directories (usually in `clusters/`) that contain further ArgoCD Application definitions.

-   **`platform-<env>.yaml`**: The root for infrastructure components (Keycloak, Kafka, 3Scale). Points to `clusters/<env>/infra`.
-   **`apps-<env>.yaml`**: The root for application groups (team-specific microservices). Points to `clusters/<env>/apps`.

---

### 🟢 1. How to Use Bootstrap
#### (Jak używać Bootstrap)

**English:**
To initialize a new environment or restore an existing one, you manually apply the bootstrap manifests to the OpenShift cluster where ArgoCD is running.
1.  **Log in** to your OpenShift cluster via CLI (`oc login`).
2.  **Apply Platform**: `oc apply -f bootstrap/platform-dev.yaml`
3.  **Apply Apps**: `oc apply -f bootstrap/apps-dev.yaml`

Once applied, ArgoCD will take over and automatically synchronize all sub-applications defined in the targeted cluster directories.

**Polski:**
Aby zainicjować nowe środowisko lub przywrócić istniejące, należy ręcznie zastosować manifesty bootstrap do klastra OpenShift, na którym działa ArgoCD.
1.  **Zaloguj się** do klastra OpenShift przez CLI (`oc login`).
2.  **Uruchom Platformę**: `oc apply -f bootstrap/platform-dev.yaml`
3.  **Uruchom Aplikacje**: `oc apply -f bootstrap/apps-dev.yaml`

Po zastosowaniu, ArgoCD przejmie kontrolę i automatycznie zsynchronizuje wszystkie pod-aplikacje zdefiniowane w docelowych katalogach klastra.

---

### 🔵 2. Adding a New Bootstrap Application
#### (Dodawanie nowej aplikacji Bootstrap)

**English:**
If you create a new environment (e.g., `test`) or a new logical group (e.g., `monitoring`), you need a new bootstrap manifest.
1.  **Copy Template**: Copy an existing file like `platform-dev.yaml`.
2.  **Update Metadata**: Change the `name` to reflect the new purpose (e.g., `platform-test`).
3.  **Update Source**: Point `spec.source.path` to the correct cluster configuration directory (e.g., `clusters/test/infra`).
4.  **Target Revision**: Usually set to `HEAD` for dynamic updates or a specific tag/branch for stability.

**Polski:**
Jeśli tworzysz nowe środowisko (np. `test`) lub nową grupę logiczną (np. `monitoring`), potrzebujesz nowego manifestu bootstrap.
1.  **Skopiuj szablon**: Skopiuj istniejący plik, np. `platform-dev.yaml`.
2.  **Zaktualizuj Metadane**: Zmień `name`, aby odzwierciedlał nowy cel (np. `platform-test`).
3.  **Zaktualizuj Źródło**: Ustaw `spec.source.path` na odpowiedni katalog konfiguracji klastra (np. `clusters/test/infra`).
4.  **Target Revision**: Zazwyczaj ustawione na `HEAD` dla dynamicznych aktualizacji lub konkretny tag/gałąź dla stabilności.

**Example / Przykład (`bootstrap/monitoring-dev.yaml`):**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: monitoring-dev
  namespace: openshift-gitops
spec:
  project: default
  source:
    repoURL: https://git.svc.arencloud.com/egevorky/rhdemo.git
    targetRevision: HEAD
    path: clusters/dev/monitoring
  destination:
    server: https://kubernetes.default.svc
    namespace: openshift-gitops
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

### ⚠️ Important Rules (Ważne Zasady)

1.  **Manual Start**: Bootstrap manifests are the ONLY files meant to be applied manually with `oc apply -f`. Everything else should be managed by ArgoCD.
2.  **GitOps Namespace**: Ensure the `metadata.namespace` is always `openshift-gitops` (or your specific ArgoCD instance namespace).
3.  **Self-Healing**: It is highly recommended to keep `automated.selfHeal: true` for bootstrap apps to ensure the "App-of-Apps" hierarchy remains intact even if someone deletes a sub-application manually.

---

### 🚀 Verification (Weryfikacja)
**EN**: After applying, check the ArgoCD UI to confirm the root application is "Healthy" and "Synced."
**PL**: Po zastosowaniu sprawdź interfejs ArgoCD, aby potwierdzić, że główna aplikacja ma status „Healthy” (Zdrowa) i „Synced” (Zsynchronizowana).
