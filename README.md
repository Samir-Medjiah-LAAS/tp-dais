# TP DAIS – dépôt étudiant

Application de streaming vidéo déployée sur Kubernetes (k3s), puis placée derrière Istio et empaquetée avec Helm.
Le sujet complet est distribué séparément.

## Images

Les images sont publiques (registre : `REGISTRY`, indiqué par les enseignants, par défaut `ghcr.io/samir-medjiah-laas`).

| Image | Étiquette | Ports | Rôle |
| --- | --- | --- | --- |
| `REGISTRY/dais-gateway` | `1.0` | 80 (HTTP), 1935 (RTMP) | nginx, point d'entrée : RTMP vers `server:1935`, `/hls/` vers `server:8080`, `/` vers `web:80` |
| `REGISTRY/dais-server` | `1.0` | 1935 (RTMP), 8080 (HLS) | nginx-rtmp : reçoit le RTMP et produit du HLS |
| `REGISTRY/dais-web` | `v1`, `v2` | 80 | page web et lecteur vidéo (hls.js) |

## Valeurs à connaître

| Élément | Valeur |
| --- | --- |
| Application RTMP | `live` |
| Nom du flux | `test` (au choix) |
| URL d'envoi | depuis l'extérieur : `rtmp://<IP_FLOTTANTE>:5000/live/test` (LoadBalancer : 5000 → 1935) ; Compose : `rtmp://localhost:1936/live/test` |
| Playlist HLS | `http://<adresse>:<port>/hls/test.m3u8` |
| Page web | `http://<adresse>:<port>/` (paramètre facultatif `?stream=<nom>`) |
| Version de la page | dans le titre (`DAIS streaming v1`), le badge et sur `/version` |
| Santé de `server` | `http://server:8080/healthz` |

Noms de Services attendus par `gateway/nginx.conf` : **`web`**, **`server`** (et `gateway`). Créez `web` et `server` avant de démarrer la Gateway.

## Flux de test

Le port 1935 est filtré avant la VM : le RTMP est donc publié à l'extérieur sur **5000** (les conteneurs restent sur 1935). Si 5000 est lui aussi filtré depuis votre réseau (ffmpeg : « Connection timed out »), lancez ffmpeg **sur la VM**, vers son adresse privée (`rtmp://IP_PRIVEE:5000/live/test`) : le flux passe toujours par le LoadBalancer.


```
ffmpeg -re -f lavfi -i testsrc=size=640x360:rate=25 -f lavfi -i sine=frequency=440 \
  -c:v libx264 -pix_fmt yuv420p -preset veryfast -tune zerolatency -g 50 \
  -c:a aac -f flv rtmp://<adresse>:<port>/live/test
```

Vérifier la playlist : `curl -s http://<adresse>:<port>/hls/test.m3u8`

Mesurer la répartition des versions (TP2) :

```
for i in $(seq 1 100); do curl -s http://<adresse>:8080/ | grep -o 'v[12]' | head -1; done | sort | uniq -c
```

## Contenu

| Chemin | Usage |
| --- | --- |
| `compose.yaml` | Échauffement Docker Compose (TP1, 4.0) |
| `gateway/nginx.conf` | Configuration de la Gateway, à mettre dans une ConfigMap (TP1, 4.2) |
| `k8s/` | Vos manifests |
| `istio/istiod-values.yaml` | Valeurs du chart `istio/istiod` : réservations mémoire réduites (TP2, 5.1) |
| `istio/gateway-values.yaml` | Valeurs du chart `istio/gateway` (TP2, 5.3) |
| `addons/` | Prometheus et Kiali pour Istio 1.31 (TP3, 6.3) |

## Versions

VM `insa-medium-4.4.20` (4 vCPU, 4 Go), un seul nœud · k3s `v1.35.9+k3s1` · Istio `1.31.4` · Helm 4 · dépôt Helm d'Istio : `https://blob.istio.io/istio-release/charts`

## kubectl sans sudo

Le `kubectl` de k3s lit `/etc/rancher/k3s/k3s.yaml` (root seulement) tant que `KUBECONFIG` n'est pas défini :

```
echo 'export KUBECONFIG=~/.kube/config' >> ~/.bashrc && source ~/.bashrc
```
