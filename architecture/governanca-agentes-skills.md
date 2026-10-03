# Governança de Agentes, Skills e Base de Conhecimento

> Fonte de verdade sobre **onde vive cada artefato** de configuração e documentação da Flag Platform.
> Regra-mãe: **uma skill/agente/documento tem uma única casa; nunca duplicar.**

---

## 1. Regra de casas (anti-duplicação)

| Artefato | Casa | Observação |
|----------|------|-----------|
| **Skill da plataforma** (vale para todos os repos) | `~/.config/opencode/skills/<nome>/` | Global. Não copiar para repos. |
| **Skill externa** (de terceiros) | `~/.agents/skills/<nome>/` | Auto-carregada. Não editar. |
| **Skill específica de um repo** | `<repo>/.opencode/skills/<nome>/` | Só o que é exclusivo daquele repo. |
| **Agente (subagent/primary)** | `~/.config/opencode/agent/<nome>.md` | Global. Não copiar para repos. |
| **Comando** | `~/.config/opencode/command/<nome>.md` | Global. |
| **Documentação transversal** | `flag_platform_docs/{adr,architecture,design,product,research,_planning,apps}/` | Hub. |
| **Documentação de implementação** | `<repo>/docs/` | Específica do repo. |
| **Tracker (tarefas/planos)** | `flag_platform_docs/_planning/<esforço>/` (transversal) ou `<repo>/docs/` (específico) | **Local (markdown)**; não GitHub Issues por padrão. |

**Divergência é o pior caso:** duas cópias com conteúdos diferentes fazem a versão errada ganhar silenciosamente. Ao encontrar duplicata, **mescle e arquive** (não apague direto).

---

## 2. Inventário atual

### 2.1 Agentes (globais `~/.config/opencode/agent/`)
| Agente | mode | Escopo |
|--------|------|--------|
| `tech-lead` | primary | Orquestrador do fluxo |
| `po-pm` | primary | Formaliza o pedido em artefato |
| `backend` | subagent | APIs, banco, regras, auth (Spring Boot) |
| `dba` | subagent | Migrations **Liquibase**, **Oracle**, schema, constraints, índices, performance |
| `frontend` | subagent | UI web |
| `app` | subagent | Mobile/desktop (Flutter, RN, Electron) |
| `tester` | subagent | Testes (quando solicitado) |
| `devops` | subagent | CI/CD, Docker, infra, deploy |
| `ux-designer` | subagent | UX, wireframes, design system |
| `explore` / `general` | built-in | Exploração / pesquisa |

### 2.2 Skills
- **Globais** (`~/.config/opencode/skills/`): `flag-platform-project`, `design-an-interface`, `edit-article`, `git-guardrails-claude-code`, `github-triage`, `grill-me`, `improve-codebase-architecture`, `migrate-to-shoehorn`, `obsidian-vault`, `prd-to-issues`, `prd-to-plan`, `qa`, `request-refactor-plan`, `scaffold-exercises`, `setup-pre-commit`, `triage-issue`, `ubiquitous-language`, `ux-review`, `write-a-prd`, `write-a-skill`.
- **Externas** (`~/.agents/skills/`): `create-github-action-workflow-specification`, `devops-engineer`, `find-skills`, `flutter-architecture`, `flutter-dart-code-review`, `java-springboot`, `postgresql-database-engineering`, `ui-ux-pro-max`, família `oci` (enterprise-ai, functions, iot-platform, oke).
- **Do workspace** (`C:\Projetos\America\.agents\skills\`): `codebase-design`, `diagnosing-bugs`, `domain-modeling`, `setup-matt-pocock-skills`, `tdd`, `wayfinder`.
- **Específicas de repo**: `america_platform_agent/.opencode/skills/coleta-organizacoes`.

### 2.3 Comandos
`~/.config/opencode/command/`: `deploy`, `implement`, `issue`. Repo: `america_platform_agent/.opencode/command/coletar.md`.

### 2.4 Base de conhecimento
| Repo | Contexto |
|------|----------|
| raiz `C:\Projetos\America` | `AGENTS.md` (diretrizes mestras) |
| `flag_platform_docs` (hub) | `adr/`, `architecture/`, `design/`, `product/`, `research/`, `_planning/`, `apps/` |
| `flag_backend` | `AGENTS.md` + `docs/` |
| `flag_admin_web` | `AGENTS.md` + `docs/` (adr, agents) |
| `flag_referee_app` | `AGENTS.md` + `docs/` |
| `flag_public_app` | `README.md` + `docs/` |
| `flag_platform_infra` | `AGENTS.md` + `docs/kb` |
| `flag_tester_e2e` | `README.md` |
| `america_platform_agent` | `README.md` + `docs/` |

---

## 3. Duplicatas resolvidas (arquivadas em `~/.config/opencode/_archive/`)

| Artefato | Ação |
|----------|------|
| `flag-dev-workflow` (global + `flag_platform/.opencode`) | Removida — substituída por `flag-platform-project` |
| `flag-platform-project` (cópia em `flag_public_app/.agents/skills`) | Removida — canônica é a global |
| `tdd` (global divergente) | Versão nova levada para o workspace; global arquivada |
| `setup-matt-pocock-skills` (global duplicada) | Removida — mantida no workspace |
| `flutter-apply-architecture-best-practices` (`flag_admin_web`) | Removida — usar a external `flutter-architecture` + ADR-011 |
| `flag_platform/.opencode/{skill,agent}` | Removido — repo descontinuado |

---

## 4. Como evoluir (processo)

1. **Nova skill/agente** → decida a casa pela tabela do §1; crie em **uma** só.
2. **Atualizar** → edite a cópia canônica; nunca crie uma segunda.
3. **Duplicata encontrada** → mescle o conteúdo único, arquive a sobra em `~/.config/opencode/_archive/`, registre aqui.
4. **Repo descontinuado** → remova o `.opencode/`/`.agents/` ativo e marque como deprecated.
5. Após qualquer mudança em config do opencode (skills/agentes/comandos), **reinicie o opencode** — a config não recarrega a quente.
