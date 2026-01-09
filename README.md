# Bug Bounty Scope Fetcher

Script Bash pour automatiser la récupération des scopes de programmes de bug bounty depuis HackerOne et YesWeHack, traiter les wildcards, et générer une liste de hosts à tester.

## Fonctionnalités

- Récupération automatique des scopes via `bbscope` (HackerOne & YesWeHack)
- Extraction des domaines et wildcards
- Énumération des sous-domaines avec `assetfinder`
- Vérification des hosts vivants avec `httpx`
- Exclusion des hosts déjà testés (basé sur les logs)
- Génération d'une liste finale prête à l'emploi

## Prérequis

### Outils requis

| Outil | Description | Installation |
|-------|-------------|--------------|
| [bbscope](https://github.com/sw33tLie/bbscope) | Récupération des scopes | `go install github.com/sw33tLie/bbscope@latest` |
| [assetfinder](https://github.com/tomnomnom/assetfinder) | Énumération de sous-domaines | `go install github.com/tomnomnom/assetfinder@latest` |
| [httpx](https://github.com/projectdiscovery/httpx) | Probe HTTP | `go install github.com/projectdiscovery/httpx/cmd/httpx@latest` |

### Configuration

Créez un fichier `.env` à la racine du projet :

```bash
HACKERONE_API_KEY="votre_api_key_h1"
HACKERONE_USERNAME="votre_username_h1"
YWH_API_TOKEN="votre_token_ywh"
```

## Utilisation

```bash
chmod +x scope_fetcher.sh
./scope_fetcher.sh
```

## Structure des fichiers générés

```
lists/
├── bbscope_h1_list_YYYY-MM-DD.txt          # Scope brute HackerOne
├── bbscope_ywh_list_YYYY-MM-DD.txt         # Scope brute YesWeHack
├── domain_h1_list_YYYY-MM-DD.txt           # Domaines extraits (H1)
├── domain_ywh_list_YYYY-MM-DD.txt          # Domaines extraits (YWH)
├── wildcard_h1_list_YYYY-MM-DD.txt         # Wildcards (H1)
├── wildcard_ywh_list_YYYY-MM-DD.txt        # Wildcards (YWH)
├── list_from_wildcard_h1_YYYY-MM-DD.txt    # Sous-domaines énumérés (H1)
├── list_from_wildcard_ywh_YYYY-MM-DD.txt   # Sous-domaines énumérés (YWH)
├── alive_from_wildcard_h1_YYYY-MM-DD.txt   # Hosts vivants (H1)
├── alive_from_wildcard_ywh_YYYY-MM-DD.txt  # Hosts vivants (YWH)
├── already_tested_hosts_YYYY-MM-DD.txt     # Hosts déjà testés
├── hosts_to_test_h1_YYYY-MM-DD.txt         # Liste finale (H1)
└── hosts_to_test_ywh_YYYY-MM-DD.txt        # Liste finale (YWH)
```

## Workflow

```
┌─────────────────┐     ┌─────────────────┐
│   HackerOne     │     │   YesWeHack     │
└────────┬────────┘     └────────┬────────┘
         │                       │
         └───────────┬───────────┘
                     ▼
           ┌─────────────────┐
           │    bbscope      │
           └────────┬────────┘
                    ▼
    ┌───────────────┴───────────────┐
    ▼                               ▼
┌────────┐                    ┌──────────┐
│Domaines│                    │Wildcards │
└────┬───┘                    └─────┬────┘
     │                              ▼
     │                      ┌─────────────┐
     │                      │ assetfinder │
     │                      └──────┬──────┘
     │                             ▼
     │                      ┌─────────────┐
     │                      │    httpx    │
     │                      └──────┬──────┘
     │                             │
     └──────────────┬──────────────┘
                    ▼
          ┌─────────────────┐
          │  Déduplication  │
          │  - logs testés  │
          └────────┬────────┘
                   ▼
          ┌─────────────────┐
          │ hosts_to_test   │
          └─────────────────┘
```

## Gestion des logs

Le script exclut automatiquement les hosts déjà testés. Pour cela, placez vos fichiers de logs dans le dossier `logs/`. Le script recherche les lignes contenant `host:` et extrait le dernier champ.

Format attendu dans les logs :
```
[INFO] host: example.com
```

