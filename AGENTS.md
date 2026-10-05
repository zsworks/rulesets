# AGENTS.md

## Purpose

Personal collection of **mihomo (Clash Meta) proxy configuration YAML files**, published at
`github.com/zsworks/rulesets` and served to a subscription-conversion frontend through `templates.json`.
There is no build system, package manager, or test suite — the deliverable is the YAML itself.

## Layout

- `mihomo/` — all configs
  - Root (`default.yaml`, `ACL4SSR_Online_Full.yaml`): shared general-purpose
    templates; referenced by **both** the `mihomo` and `singbox` categories in `templates.json`.
  - `mihomo/zsworks/` — the actively maintained personal configs. Nearly all edits happen here:
    - `configfull.yaml` — built on the Chunlion/Clash_Rule-Set anti-DNS-leak template.
    - `zworks_full.yaml` — built on ACL4SSR_Online_Full_WithIcon, plus the custom
      强制代理 / 跳过代理 rule-providers served from `rules/zworks/`.
  - `mihomo/zhuqq2020/` — static snapshots copied from upstream repos; keep the upstream
    attribution header at the top of each file.
- `rules/zworks/` — text rule files (`force_proxy.txt`, `bypass_proxy.txt`) pulled by the
  `zworks_full.yaml` custom rule-providers; served from the published repo.
- `templates.json` — registry consumed by the subscription converter; `value` paths are relative to
  `mihomo/`. A newly added config file is not selectable until registered here.

## Conventions

- **LF line endings** (a commit normalized CRLF→LF); files are tracked with mode 100755.
- Comments and proxy-group names are in **Chinese** (`香港手动`, `一键代理`, `全球直连`, …); keep that style.
- `zsworks/configfull.yaml` is built from **YAML anchors** (`Anchor_*`): proxy-groups and rule-providers
  inherit via `<<: *Anchor_X`. Edit the anchor once, not each inheriting entry. Region membership is
  regex filters (`Anchor_HK`, `Anchor_US`, …) matched against node names; line labels (IEPL/IPLC/倍率)
  deliberately do not count as region words.
- Rules are not local: every `rule-providers` entry pulls a remote `.mrs` rule-set (jsDelivr /
  raw.githubusercontent). Routing is expressed as `RULE-SET,<provider>,<group>` entries.
- **Never commit real subscription URLs or airport names.** The placeholders `订阅链接` / `机场1` are
  intentional (the file header warns about leaking them).
- Preserve the `zworks.me` personal-network customizations when restructuring the config: the
  `+.zworks.me` fake-ip-filter entry, the `nameserver-policy` static mapping to `172.26.0.88`, and the
  `DOMAIN-SUFFIX,zworks.me,<direct group>` rule (see commits `c40ff43`, `e1e8714`).

## Validation

No automated tests. Before finishing an edit:

1. YAML must parse: `python3 -c "import yaml; yaml.safe_load(open('mihomo/zsworks/configfull.yaml'))"`.
2. Cross-check every target in `rules:` against a name in `proxy-groups:` (or `DIRECT`/`REJECT`), and
   every `RULE-SET,<name>` against a key in `rule-providers:` — a dangling reference makes mihomo
   reject the whole config.
3. `mihomo -t -f <file>` is the real check if the mihomo binary is available (it is not installed in
   this WSL environment).

## Gotchas

- The `dns:` section of `zsworks/configfull.yaml` is deliberately tuned against DNS leaks (fake-ip with
  blacklist filter, `respect-rules`, split `direct-nameserver` / `nameserver-policy`). Change entries
  there only with that goal in mind.
- Group names referenced by rules are Chinese strings; renaming a group requires updating every rule
  and anchor that references it.
