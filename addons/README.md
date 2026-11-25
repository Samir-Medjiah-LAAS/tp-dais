Add-ons d'observabilité d'Istio 1.31 (Prometheus et Kiali), copiés depuis
https://github.com/istio/istio/tree/release-1.31/samples/addons

Ce sont des configurations de démonstration : pas de persistance, pas de haute disponibilité.
Grafana n'est pas fourni : aucun jalon ne l'utilise et la mémoire de la VM est comptée.
Installation (TP3, 6.3) : kubectl apply -f addons/
