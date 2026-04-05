# Formation eBPF

 formation technique sur eBPF (extended Berkeley Packet Filter) couvrant l'observabilité système, la sécurité runtime et le développement bas niveau sous Linux.

---

## Vue d'ensemble

| Élément | Détail |
|---|---|
| **Public** | Ingénieurs système, DevOps, équipes sécurité/SOC, développeurs kernel |
| **Niveau** | Intermédiaire à avancé (bases Linux requises) |
| **Durée totale** | 6 jours (Module 0 : 0,5j + 4 Parcours : 2j chacun) |
| **Format** | Présentiel ou distanciel — labs pratiques sur VM Linux |
| **Prérequis kernel** | Linux ≥ 5.8, BTF disponible (`/sys/kernel/btf/vmlinux`) |

---

## Structure du dépôt

```
formation-ebpf/
│
├── module-0/                     Socle eBPF (prérequis à tous les parcours)
│   ├── module0_formateur.docx    Slides + notes pédagogiques
│   ├── module0_etudiant.docx     Support stagiaire avec exercices
│   └── module0_ebpf.pptx         Présentation (24 slides)
│
├── parcours-a/                   Observabilité & Performance
│   ├── formateur_parcours_a.docx Contenu de cours détaillé
│   ├── lab-ebpf-dashboard/       Lab Docker opérationnel
│   │   ├── docker-compose.yml
│   │   ├── configs/              Configurations ebpf_exporter
│   │   ├── grafana/              Dashboard pré-provisionné
│   │   └── scripts/              start.sh · stop.sh · load.sh
│   └── parcours_a_observabilite.pptx  Présentation (13 slides)
│
├── parcours-b/                   Sécurité & Détection comportementale
│   ├── formateur_parcours_b.docx Slides + notes pédagogiques
│   ├── etudiant_parcours_b.docx  Support stagiaire avec exercices
│   ├── lab-ebpf-security/        Lab Docker opérationnel
│   │   ├── docker-compose.yml
│   │   ├── falco/                Configuration Falco + règles YAML
│   │   ├── grafana/              Dashboard sécurité pré-provisionné
│   │   └── scripts/              start.sh · stop.sh · attack-scenarios.sh
│   └── parcours_b_securite.pptx  Présentation (21 slides)
│
└── README.md
```

---

## Module 0 — Socle eBPF `0,5 jour`

**Prérequis obligatoire** à tous les parcours. Couvre les fondations conceptuelles et pratiques.

| Bloc | Contenu | Durée |
|---|---|---|
| 1 | Pourquoi eBPF : historique, alternatives, écosystème | 30 min |
| 2 | Architecture interne : cycle de vie, maps, types de programmes, BTF/CO-RE | 45 min |
| 3 | bpftrace : syntaxe, probes, agrégations — 4 exercices pratiques | 60 min |
| 4 | Le vérificateur : rôle, erreurs courantes, corrections | 30 min |
| 5 | Environnement de développement : libbpf, Makefile, frameworks | 25 min |

**Livrables stagiaire :** support de cours avec exercices, cheatsheet bpftrace, référence helpers, table d'erreurs vérificateur.

---

## Parcours A — Observabilité & Performance `2 jours`

Construction d'une chaîne d'observabilité kernel complète avec eBPF, VictoriaMetrics et Grafana.

### Jour 1 — Traçage système

| Bloc | Contenu |
|---|---|
| Profilage CPU & Flamegraphs | Sampling `perf_event` à 99 Hz, flamegraphs on-CPU / off-CPU |
| Latence I/O disque | `biolatency`, tracepoints `block`, seuils de diagnostic |
| Lock contention & Runtimes | Contention futex/mutex, GC profiling Go/Java, async Rust (Tokio) |

### Jour 2 — Réseau & Intégration

| Bloc | Contenu |
|---|---|
| TCP tracing | `tcplife`, `tcpretrans`, états TCP, détection retransmissions |
| Corrélation eBPF + OpenTelemetry | Propagation span_id, latence kernel vs applicative |
| Détection régressions CI/CD | Gate automatique sur latence p99 dans la pipeline |
| **Lab final** | Dashboard temps réel CPU/Mémoire/I-O avec ebpf_exporter + VictoriaMetrics + Grafana |

### Lab Docker — Dashboard observabilité

```bash
# Prérequis
uname -r                        # ≥ 5.8
ls /sys/kernel/btf/vmlinux      # BTF requis

# Démarrage
sudo ./scripts/start.sh

# Interfaces
# Grafana         http://localhost:3000   (admin / ebpf-lab)
# VictoriaMetrics http://localhost:8428/vmui
# ebpf_exporter   http://localhost:9435/metrics

# Générer de la charge
./scripts/load.sh
```

**Métriques collectées :**

| Famille | Métriques |
|---|---|
| CPU | `cpu_time_seconds` (sampling 99 Hz), `run_queue_latency_seconds` |
| Mémoire | `page_faults_total`, `oom_kills_total`, `kernel_page_alloc_total` |
| I/O disque | `bio_latency_seconds` (histogramme), `bio_requests_total` (IOPS) |

---

## Parcours B — Sécurité & Détection comportementale `2 jours`

Construction d'une chaîne EDR minimale avec Falco, Falcosidekick, VictoriaMetrics et Grafana.

### Contenu

| Bloc | Techniques surveillées | Tags MITRE |
|---|---|---|
| Escalade de privilèges | `setuid`, `setreuid`, `capset`, binaires SUID | T1548.001, T1548.003 |
| Fichiers sensibles | `/etc/shadow`, clés SSH, credentials cloud, crontab, `ld.so.preload` | T1003.008, T1552.001, T1574.006 |
| Exfiltration réseau | Reverse shell, DNS tunneling, connexions suspectes, netcat | T1041, T1071.004, T1059.004 |
| Lab EDR maison | Falco + Falcosidekick + VictoriaMetrics + Grafana | — |

### Lab Docker — Chaîne de détection

```bash
# Démarrage
sudo ./scripts/start.sh

# Interfaces
# Grafana         http://localhost:3000   (admin / ebpf-lab)
# VictoriaMetrics http://localhost:8428/vmui
# Falcosidekick   http://localhost:2801/metrics

# Scénarios d'attaque
sudo ./scripts/attack-scenarios.sh           # menu interactif
sudo ./scripts/attack-scenarios.sh privesc   # escalade de privilèges
sudo ./scripts/attack-scenarios.sh files     # accès fichiers sensibles
sudo ./scripts/attack-scenarios.sh network   # exfiltration réseau
sudo ./scripts/attack-scenarios.sh edr       # chaîne complète post-exploitation
```

**Règles Falco incluses :** 17 règles réparties sur 3 fichiers YAML, toutes mappées MITRE ATT&CK.

| Fichier | Règles |
|---|---|
| `privilege_escalation.yaml` | Privilege Escalation via setuid · setreuid · Capabilities Modification · SUID Binary Execution · Container Escape via nsenter |
| `file_monitoring.yaml` | Shadow File Read · Critical Config Write · SSH Private Key Access · Cloud Credentials · Kubernetes Token · Persistence via Cron · LD_PRELOAD |
| `network_exfiltration.yaml` | Reverse Shell · Shell from Web Service · Unexpected Outbound · High Frequency DNS · Netcat Execution · Data Staging |

---

## Parcours C — Réseau & Kernel bas niveau `2 jours` *(à venir)*

XDP, TC (Traffic Control), pare-feu eBPF, load balancer, DDoS mitigation, QoS.

## Parcours D — Debugging & Runtimes `2 jours` *(à venir)*

uprobes sur binaires fermés, black-box debugging, chaos engineering, USDT, async tracing.

---

## Prérequis techniques

### Kernel et système

```bash
# Vérifier la version kernel
uname -r                                    # ≥ 5.8 requis, 6.x recommandé

# Vérifier la disponibilité de BTF (Base du CO-RE)
ls /sys/kernel/btf/vmlinux

# Vérifier les outils
bpftrace --version                          # ≥ 0.18
bpftool version
clang --version | head -1                   # ≥ 12
```

### Installation des outils (Ubuntu 22.04 / 24.04)

```bash
sudo apt update && sudo apt install -y \
    clang llvm \
    libbpf-dev \
    linux-headers-$(uname -r) \
    linux-tools-$(uname -r) \
    bpftrace \
    pkg-config
```

### Docker (labs)

```bash
# Docker Engine avec le plugin Compose
curl -fsSL https://get.docker.com | sh
docker compose version                      # ≥ 2.x requis
```

---

## Carte des parcours par profil

| Profil | Parcours recommandés |
|---|---|
| Ingénieur système / DevOps | Module 0 → Parcours A |
| Équipe sécurité / SOC | Module 0 → Parcours B |
| Développeur kernel / bas niveau | Module 0 → Parcours C |
| SRE / Développeur | Module 0 → Parcours D |
| Profil complet | Module 0 → A → B → C → D |

---

## Formats des livrables

Chaque module produit :

| Format | Description |
|---|---|
| `.docx` formateur | Slides + notes pédagogiques détaillées (contexte, analogies, points d'attention) |
| `.docx` étudiant | Théorie, code commenté, exercices numérotés, tableaux de validation, annexes |
| `.pptx` | Présentation visuelle prête à projeter (fond sombre titres, clair contenu) |
| `docker-compose.yml` | Lab opérationnel : `sudo ./scripts/start.sh` suffit |
| Scripts `.sh` | `start.sh`, `stop.sh`, scénarios de charge ou d'attaque |
| Règles `.yaml` | Falco : fichiers versionnables dans git, mappés MITRE ATT&CK |
| Configurations `.yaml` | ebpf_exporter : programmes eBPF + mapping Prometheus |

---

## Stack technique

| Composant | Rôle | Version testée |
|---|---|---|
| Linux kernel | Hôte des programmes eBPF | 6.x (BTF, CO-RE, modern_ebpf) |
| bpftrace | Traçage ad hoc, prototypage | 0.20+ |
| libbpf | Framework C production | 0.8+ |
| Falco | Détection runtime (driver modern_ebpf) | 0.38+ |
| Falcosidekick | Routage alertes + métriques Prometheus | 2.28+ |
| ebpf_exporter | Export métriques eBPF → Prometheus | 2.x (Cloudflare) |
| VictoriaMetrics | Stockage métriques (compatible Prometheus) | stable |
| Grafana | Visualisation, dashboards pré-provisionnés | 10+ |

---

## Références

- [eBPF.io](https://ebpf.io) — Documentation officielle
- [Kernel BPF docs](https://docs.kernel.org/bpf/index.html) — Référence kernel
- [bpftrace Reference](https://github.com/bpftrace/bpftrace/blob/master/docs/reference_guide.md)
- [Falco documentation](https://falco.org/docs/)
- [MITRE ATT&CK Enterprise Linux](https://attack.mitre.org/matrices/enterprise/linux/)
- [Brendan Gregg — BPF Performance Tools](https://www.brendangregg.com/bpf-performance-tools-book.html)
- [Cloudflare ebpf_exporter](https://github.com/cloudflare/ebpf_exporter)
- [VictoriaMetrics documentation](https://docs.victoriametrics.com)

