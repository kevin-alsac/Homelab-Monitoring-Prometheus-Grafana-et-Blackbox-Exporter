# Notes de sécurité

Ce projet est prévu pour un environnement de test ou de homelab local.

## Recommandations

- Ne pas exposer Prometheus sur Internet.
- Ne pas exposer Blackbox Exporter sur Internet.
- Restreindre Grafana au réseau local ou à un VPN.
- Changer le mot de passe admin par défaut.
- Créer un compte en lecture seule pour la consultation.
- Ne pas stocker de secrets dans les labels Prometheus.
- Ne pas publier de captures contenant des IP publiques, tokens ou noms internes sensibles.
- Éviter d'ouvrir des ports sur la box Internet.

## Pourquoi Prometheus peut être sensible

Prometheus peut révéler :

- noms de machines
- adresses IP locales
- noms de services
- ports ouverts
- chemins de montage
- disponibilité des équipements
- habitudes d'utilisation indirectes

Ces informations ne sont pas forcément critiques seules, mais elles donnent une cartographie utile de l'infrastructure.
