### 🔍 Opis

Ten komponent służy do budowania i publikowania obrazu Dockera do rejestru (`$CI_REGISTRY`). Umożliwia parametryzację takich elementów jak wersja kontenera, obraz Dockera czy ustawienia TLS do `docker:dind`.

Przykład zastosowania w pipeline'ie:

```yaml
include:
  component: registry.gitlab.com/your-group/gitlab-components/docker-publish
```

---
## ⚙️ Parametry wejściowe (`inputs`)

| Nazwa                | Typ    | Domyślna wartość | Opis                                                           |
| -------------------- | ------ | ---------------- | -------------------------------------------------------------- |
| `docker_image`       | string | `docker:24`      | Obraz Dockera używany do wykonania zadania                     |
| `docker_host`        | string | `""`             | Wartość zmiennej `DOCKER_HOST` dla połączenia z `docker:dind`  |
| `docker_tls_certdir` | string | `""`             | Ścieżka TLS cert dir, np. `/certs` (pusta wartość wyłącza TLS) |
| `container_version`  | string | `latest`         | Wersja kontenera, która zostanie nadana tagowi                 |

---
## 🧬 Zmienne środowiskowe

| Nazwa zmiennej           | Wartość                            |
| ------------------------ | ---------------------------------- |
| `CONTAINER_IMAGE_DOCKER` | `$[[ inputs.docker_image ]]`       |
| `DOCKER_HOST`            | `$[[ inputs.docker_host ]]`        |
| `DOCKER_TLS_CERTDIR`     | `$[[ inputs.docker_tls_certdir ]]` |
| `CONTAINER_VERSION`      | `$[[ inputs.container_version ]]`  |

---

## 🧱 Zależności

* Włączenie plików lokalnych (muszą być obecne w repozytorium):

  * `/source/logo.yml` – do wyświetlenia loga komponentu
  * `/source/input_variables_docker.yml` – do wyświetlania inputów środowiskowych
* Wymaga `docker:dind` jako serwisu

---
## 🚀 Job: `🌐 publish docker image`

Ten job:

1. Buduje obraz Dockera z bieżącego katalogu (wymaga obecności pliku `Dockerfile`).
2. Używa tagu zgodnego z podaną wersją `container_version`.
3. Wysyła obraz do `CI_REGISTRY`.
4. Zapisuje pełną nazwę obrazu do pliku `versioning_container.env`, który może być użyty w kolejnych etapach.

### 📜 Skrypt

```bash
docker build --build-arg CONTAINER_VERSION=$CONTAINER_VERSION -t $CI_REGISTRY_IMAGE:$CONTAINER_VERSION .
docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
docker push $CI_REGISTRY_IMAGE:$CONTAINER_VERSION
```

---
### 📤 Artefakty

* `versioning_container.env` jako `dotenv` – zawiera pełną nazwę obrazu:
  `CONTAINER_IMAGE_VERSION=$CI_REGISTRY_IMAGE:$CONTAINER_VERSION`

---
## 🧪 Przykład użycia z parametrami

```yaml
include:
  component: registry.gitlab.com/your-group/gitlab-components/docker-publish
  inputs:
    docker_image: "docker:26"
    container_version: "v1.2.3"
```