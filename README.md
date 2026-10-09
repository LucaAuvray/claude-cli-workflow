# Mon workflow Claude Code (CLI) : réseau, système et dev

Ma configuration de [Claude Code](https://claude.com/claude-code) sous Windows 11, pensée pour deux usages : **réseau / système** (Cisco, homelab, serveurs en SSH) et **développement** (React, Rust, Node). Tout est ici, sans secret : chemins génériques, aucune clé API.

> Principe directeur : peu de briques, choisies pour ne pas se marcher dessus. Chaque skill ou MCP ajouté coûte du contexte à chaque session, donc je n'ajoute que ce qui comble un vrai manque.

## Vue d'ensemble

| Couche | Ce que j'utilise | Rôle |
|---|---|---|
| Règles globales | `~/.claude/CLAUDE.md` | Garde-fous serveurs, choix des outils de doc |
| Plugins | Ponytail, Superpowers, claude-hud, rust-analyzer-lsp, 21st | Style de code, méthode, statusline, LSP Rust, UI |
| MCP | Context7, GitMCP, Packet Tracer, Claude in Chrome, 21st | Doc de libs, simulation réseau, navigateur, composants |
| Skills / agents | Sélection du dépôt [ECC](https://github.com/affaan-m/ECC) (MIT) | Revue réseau, sécurité, Rust, React, Docker |
| Hooks | Son + notification Windows | Savoir quand Claude a fini ou m'attend |

## 1. CLAUDE.md global

Lu à chaque session. Court exprès.

```markdown
# Environnement
- Réponds en français.
- Windows 11 : PowerShell + Git Bash. Pas de sshpass ni d'apt en local.
- Serveurs sur Tailscale : passer par les alias de `~/.ssh/config` (`ssh <alias> '<cmd>'`).
  plink uniquement pour un hôte dont la clé n'est pas encore autorisée.
- Superpowers n'est activé que dans les repos de code. Dossier avec `.git` où il est absent : proposer
  `claude plugin enable superpowers@superpowers-dev --scope local`.
- Préférer les CLI (ssh, gh, docker, kubectl, pvesh/qm/pct, jq) à un MCP.

# Documentation de bibliothèques
- API d'une lib connue : Context7 d'abord (resolve-library-id puis query-docs).
- GitMCP si Context7 ne trouve rien, si la lib est petite ou très récente, ou pour lire le vrai code source.
- Sinon `gh` ou WebFetch sur la doc officielle. Ne pas répondre de mémoire sur une API qui a pu changer.

# Sur un serveur
- Lire et diagnostiquer avant de modifier.
- Copier un fichier en `.bak-<date>` avant de l'éditer (les checkpoints de Claude ne couvrent pas les modifs faites via ssh).
- Vérifier après chaque changement (systemctl status, journalctl, curl) et montrer la sortie.
```

Pourquoi ces règles : les modifications faites par SSH échappent aux checkpoints de Claude, donc la sauvegarde `.bak` et la vérification systématique sont à imposer soi-même.

## 2. Plugins

| Plugin | Source | Pourquoi |
|---|---|---|
| Ponytail | `DietrichGebert/ponytail` | Force le plus petit changement qui résout vraiment le problème |
| Superpowers | `obra/superpowers` | TDD, planification, revue. **Activé seulement dans les repos de code** (`--scope local`), désactivé globalement |
| claude-hud | `jarrodwatts/claude-hud` | Statusline : contexte, usage, outils en cours |
| rust-analyzer-lsp | marketplace officielle | Diagnostics et navigation Rust |
| 21st | `21st-dev/claude-code-plugin` | Composants UI, thèmes et variantes (React / shadcn / Tailwind) |

## 3. Serveurs MCP

| MCP | Usage |
|---|---|
| **Context7** | Doc à jour des bibliothèques connues (première source) |
| **GitMCP** | Doc et code de n'importe quel dépôt GitHub (secours, libs récentes, lecture du source) |
| **Packet Tracer** | Pilote Cisco Packet Tracer (topologies, VLAN, ACL, NAT) depuis une description en langage naturel |
| **Claude in Chrome** | Navigation et tests dans le vrai navigateur |
| **21st** (via le plugin) | Recherche et installation de composants UI |

Ce que je n'ai **pas** ajouté, volontairement : un MCP GitHub (le CLI `gh` suffit), les MCP Cloudflare (pas de Workers/Pages, seulement un tunnel géré par `cloudflared`), les MCP de mémoire (redondants avec les fichiers de mémoire de Claude Code).

Claude voit le nom et la description de chaque outil et choisit selon le contexte, mais ce n'est pas garanti. Pour les cas ambigus (Context7 vs GitMCP), la règle est écrite dans le `CLAUDE.md` ; pour un cas ponctuel, je nomme l'outil dans le prompt.

## 4. Skills et agents (sélection ECC)

Copiés un par un dans `~/.claude/skills/` (et `~/.claude/agents/`) depuis ECC, sans installer le plugin complet (68 agents, 293 skills, des hooks monolithiques : trop de contexte et de recoupements avec Superpowers et Ponytail). Seuls le nom et la description de chaque skill sont chargés ; le contenu complet ne l'est que s'il est utilisé.

### Réseau / système

| Skill | À quoi il sert | Utile quand |
|---|---|---|
| `network-config-validation` | Checklist avant déploiement : commandes dangereuses, IP en double, chevauchements de sous-réseaux, ACL non définies | Avant d'appliquer une config sur un routeur ou un switch |
| `cisco-ios-patterns` | Revue IOS/IOS-XE : wildcard masks, placement des ACL, fenêtre de changement | Écriture ou relecture de config Cisco |
| `network-interface-health` | Erreurs, CRC, drops, duplex mismatch, flapping (routeurs, switches, Linux) | Un lien est lent ou instable |
| `homelab-wireguard-vpn` | WireGuard : clés, peers, split ou full tunnel | Accès distant à un homelab |
| `homelab-vlan-segmentation` | VLAN IoT / invités / serveurs (UniFi, pfSense/OPNsense, MikroTik) | Séparer un réseau domestique ou de lab |
| `homelab-pihole-dns` | Pi-hole : blocklists, DoH, DHCP, DNS locaux | Seulement si Pi-hole est en place |

Agents associés : `network-config-reviewer` (lecture seule), `network-troubleshooter` (diagnostic couche par couche en lecture seule).

### Dev

| Skill | À quoi il sert | Utile quand |
|---|---|---|
| `security-review` | Checklist sécurité : auth, entrées, secrets, API | Code qui touche à l'authentification ou aux secrets |
| `rust-patterns` / `rust-testing` | Rust idiomatique, stratégie de tests | Écrire ou relire du Rust |
| `react-patterns` / `react-testing` | Hooks, Suspense, Testing Library, Vitest, axe | Composants React |
| `frontend-a11y` | ARIA, clavier, focus | Modales, menus, formulaires |
| `docker-patterns` / `deployment-patterns` | Dockerfile, Compose, CI/CD, rollback | Conteneuriser ou livrer |
| `context-budget` | Mesure la consommation de contexte des skills, agents et MCP | Les sessions se remplissent trop vite |

Agent associé : `silent-failure-hunter` (erreurs avalées, mauvais fallbacks).

Déclenchement : les skills s'activent selon leur description quand la demande correspond ; les agents, moins systématiquement, donc je les nomme (« passe `silent-failure-hunter` sur ce diff »).

## 5. Hook : son et notification quand Claude finit ou m'attend

Deux événements : `Stop` (réponse terminée) et `Notification` (permission ou réponse attendue). Dans `~/.claude/settings.json` :

```json
{
  "hooks": {
    "Stop": [
      { "hooks": [ { "type": "command",
        "command": "powershell.exe -NoProfile -ExecutionPolicy Bypass -File C:/Users/<moi>/.claude/hooks/notify.ps1 -Kind stop" } ] }
    ],
    "Notification": [
      { "hooks": [ { "type": "command",
        "command": "powershell.exe -NoProfile -ExecutionPolicy Bypass -File C:/Users/<moi>/.claude/hooks/notify.ps1 -Kind attention" } ] }
    ]
  }
}
```

`~/.claude/hooks/notify.ps1` (toast Windows et son ; le nom du dossier du projet affiché entre crochets vient du champ `cwd` que Claude Code envoie au hook) :

```powershell
param([ValidateSet('stop','attention')][string]$Kind = 'stop')
# Son + toast Windows. Appelé par les hooks Stop / Notification de settings.json.
$ErrorActionPreference = 'SilentlyContinue'

$msg = if ($Kind -eq 'stop') { 'Claude a terminé.' } else { 'Claude a besoin de toi.' }
try {
    $raw = [Console]::In.ReadToEnd()
    if ($raw) {
        $j = $raw | ConvertFrom-Json
        if ($Kind -eq 'attention' -and $j.message) { $msg = $j.message }
        if ($j.cwd) { $msg += " [" + (Split-Path $j.cwd -Leaf) + "]" }
    }
} catch {}

$title = if ($Kind -eq 'stop') { 'Claude Code : terminé' } else { 'Claude Code : action requise' }

[void][Windows.UI.Notifications.ToastNotificationManager, Windows.UI.Notifications, ContentType = WindowsRuntime]
[void][Windows.Data.Xml.Dom.XmlDocument, Windows.Data.Xml.Dom.XmlDocument, ContentType = WindowsRuntime]
$xml = New-Object Windows.Data.Xml.Dom.XmlDocument
$xml.LoadXml("<toast><visual><binding template='ToastGeneric'><text>$([Security.SecurityElement]::Escape($title))</text><text>$([Security.SecurityElement]::Escape($msg))</text></binding></visual><audio silent='true'/></toast>")
$appId = '{1AC14E77-02E7-4E5D-B744-2EB1AE5198B7}\WindowsPowerShell\v1.0\powershell.exe'
[Windows.UI.Notifications.ToastNotificationManager]::CreateToastNotifier($appId).Show([Windows.UI.Notifications.ToastNotification]::new($xml))

# Sons Windows plus audibles que les SystemSounds (qui n'ont pas de PlaySync) ; repli sur Exclamation si le .wav manque.
$wav = "$env:WINDIR\Media\" + $(if ($Kind -eq 'stop') { 'tada.wav' } else { 'Alarm02.wav' })
if (Test-Path $wav) { (New-Object System.Media.SoundPlayer $wav).PlaySync() } else { [System.Media.SystemSounds]::Exclamation.Play(); Start-Sleep -Milliseconds 1200 }
```

Deux pièges rencontrés :
- `[System.Media.SystemSounds]::X` n'a **pas** de `PlaySync()` (seulement `Play()`, asynchrone) ; avec `SilentlyContinue`, l'erreur passait inaperçue. Solution : `System.Media.SoundPlayer` sur un `.wav` de `C:\Windows\Media`.
- PowerShell 5.1 lit un script UTF-8 **sans BOM** comme de l'ANSI : « terminé » devenait « terminÃ© ». Enregistrer le script **avec BOM**.

## 6. Ce qui n'est pas publié

Clés API, jetons, variables d'environnement, alias SSH, adresses de machines, connecteurs personnels (mail, agenda, drive) et skills perso.

---

*Config évolutive, dernière mise à jour : octobre 2026. Les skills et agents viennent d'[ECC](https://github.com/affaan-m/ECC) (licence MIT), tous les crédits à ses auteurs.*
