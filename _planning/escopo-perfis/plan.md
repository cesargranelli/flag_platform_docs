# Plano — Escopo por Perfil (escopo × papel)

> **Status:** aberto. Decisões de produto fechadas em 2026-10-15 (ver `map.md` §Decisões).
> **Repos:** flag_backend (F1–F2) · flag_admin_web (F3–F5)

---

## 1. Objetivo

Tornar a plataforma coerente com o discurso de gestão: **cada usuário enxerga o que lhe
pertence**. O modelo fundido `UserRole` (6 valores decidindo "quem sou" e "quanto vejo") é
substituído por dois eixos independentes:

```
ESCOPO (de onde venho — vem do vínculo, resolvido pelo backend)
  PLATFORM        → sem dono; vê a plataforma inteira
  ORGANIZATION    → dono = organizationId; vê minha org, agremiações filiadas,
                    competições e sedes da minha org
  INSTITUTION     → dono = institutionId; vê minha agremiação e minhas equipes

PAPEL (o que posso fazer dentro do escopo — deriva do UserRole atual)
  ADMIN | EDITOR | VIEWER
```

### 1.1 Mapeamento do UserRole atual

| `UserRole` (hoje) | Escopo | Papel |
|---|---|---|
| `ADMIN` | `PLATFORM` | `ADMIN` |
| `ORGANIZER` | `ORGANIZATION` | `EDITOR` |
| `MANAGER` (Presidente) | `INSTITUTION` | `EDITOR` |
| `COMMISSIONER` (Delegado) | `INSTITUTION` | `VIEWER` |
| `REFEREE` (Árbitro) | `INSTITUTION` | `VIEWER` |
| `FAN` (Torcedor) | fora do admin_web | — |

> O escopo passa a ser **derivado do vínculo** (`organizationId`/`institutionId`), não do enum.
> O enum sobrevive como `papel`; o backend decide o escopo a partir de quem é o dono do
> vínculo. `ORGANIZER`/`MANAGER` deixam de significar "quanto vejo" e só ajudam a derivar o papel.

---

## 2. Fluxo de dados (contrato)

```
Login ──► Backend autentica ──► gera token com claims:
           { sub, papel, scope: PLATFORM|ORGANIZATION|INSTITUTION,
             organizationId?, institutionId? }
                │
                ▼
GET /api/v1/auth/me ──► { ..., organizationId?, institutionId?, scope }
                │        (fonte única do vínculo; app NÃO infere)
                ▼
Listagens (backend aplica o escopo ANTES de paginar):
  scope=PLATFORM      → tudo
  scope=ORGANIZATION  → organizations: SÓ a minha
                        institutions: organizationId=orgId (server-side)
                        competitions: organizationId=orgId
                        venues: organizationId=orgId
  scope=INSTITUTION   → organizations/venues: 403
                        institutions: institutionId=instId
                        teams: institutionId=instId
                │
                ▼
App Flutter: home/guards/listas leem AppScopeProvider
             (o app consulta; nunca decide escopo sozinho)
```

**Princípio:** esconder card é cortesia; **o gate é o backend**. Se alguém chamar
`GET /organizations` com token `INSTITUTION`, o servidor responde `403`.

### 2.1 Detalhes do backend (F1–F2)

- **ESC-01 — identidade**
  - Popula `organizationId`/`institutionId` do usuário no `/auth/me` a partir do cadastro/
    aprovação (hoje nunca são preenchidos — `auth_repository.dart` monta `AuthUser.fromUser`).
  - Adiciona claims de escopo ao JWT emitido no login.
  - Regra de derivação: `ADMIN` → `PLATFORM`; usuário com `organizationId` → `ORGANIZATION`;
    com `institutionId` → `INSTITUTION`; sem vínculo e não-admin → rejeição/clareza (definir
    comportamento: pendente de vínculo).
- **ESC-02 — filtros**
  - Toda listagem lê o escopo do **usuário autenticado** (claim/princípio de autoridade do
    servidor), não de query param enviado pelo cliente.
  - Paginação server-side (`page`/`size` + `X-Total-Count`) já existe (esforço
    [`padronizacao-listagens`](../padronizacao-listagens/plan.md)); o filtro de escopo entra na
    mesma camada (`*Specification`), sem SQL nativo (Liquibase/DDL intocados).
  - `403` com corpo claro em endpoints fora do escopo.
  - Escopo `PLATFORM` permanece byte-compatível (nenhuma regressão para admin).

### 2.2 Detalhes do app (F3–F5)

- **ESC-03 — fundação**
  - Novo modelo `AppScope` (`scope`, `organizationId?`, `institutionId?`) + `papel`.
  - `AuthUser` passa a carregar o vínculo vindo do `/auth/me`; `SessionManager` persiste.
  - `AppScopeProvider` (Riverpod) como fonte única lida por UI/guards.
- **ESC-04 — home/cabeçalho** — matriz §3.
- **ESC-05 — guards** — `app_router.dart` ganha `redirect` por escopo + tela 403 amigável
  (hoje o único gate é autenticação/verificação de e-mail, linhas 22–72).
- **ESC-06 — listas** — ViewModels param de mandar listagens globais; telas "Minha
  Organização" (detalhe) e "Minha Agremiação" (detalhe); ações de escrita passam a olhar
  `papel` (`ADMIN|EDITOR` escreve, `VIEWER` só lê) unificando os 3 padrões ad-hoc de hoje
  (`isAdmin`, `canWrite = admin|organizer|manager`, `canWrite = user != null`).

---

## 3. Matriz de visibilidade (UI)

| Recurso | ADMIN (PLATFORM) | ORG | INST |
|---|---|---|---|
| Organizações (lista) | ✔ | ✗ | ✗ |
| Minha Organização (detalhe) | — | ✔ | ✗ |
| Agremiações (lista) | ✔ | ✔ (só as da minha org) | ✗ |
| Minha Agremiação (detalhe) | — | — | ✔ |
| Competições | ✔ | ✔ (só da minha org) | ✔ (inscritas + janelas abertas — card único) |
| Sedes | ✔ | ✔ (só as da minha org) | ✗ |
| Equipes | ✔ | ✔ (só das agremiações da org) | ✔ (só as minhas) |
| Pessoas/Elenco | ✔ | escopo da org (ver ESC-06) | por equipe própria |
| Aprovações | ✔ | ✗ | ✗ |
| Usuários | ✔ | ✗ | ✗ |

**Wireframes aprovados:** home de ORG sem card "Organizações" (card "Minha Organização" →
detalhe); tarja de escopo no cabeçalho abaixo do greeting ("▸ Federação de Basquete de Goiás");
home de INST com cards "Minha Agremiação", "Equipes" e "Competições" (único); subtítulo do
card de lista mostra "18/48" (meus/total) para tornar o filtro visível.

---

## 4. Fases e ordem de execução

| Fase | Issue(s) | Repo | Descrição |
|---|---|---|---|
| F1 | ESC-01 | flag_backend | Vínculo no `/auth/me` + claims no token |
| F2 | ESC-02 | flag_backend | Filtro de escopo por claim + 403 em todas as listagens |
| F3 | ESC-03 | flag_admin_web | Modelo `AppScope` + transporte do vínculo |
| F4 | ESC-04, ESC-05, ESC-06 | flag_admin_web | Home por escopo, guards, listas |
| F5 | ESC-07 | docs | `AGENTS.md` + registro das decisões |

**Tracer bullet sugerido:** F1/F2 com `venues` (menor superfície: FK `organizationId` já é
obrigatório no modelo) → F3/F4 com o card "Sedes" da home de organização end-to-end, antes de
replicar nas demais listagens.

---

## 5. Fora de escopo

- Multi-vínculo (usuário com várias agremiações e troca de escopo) — modelo prevê
  (`scope` claim), UI não entrega agora.
- Mudança de `Person.role` (papel esportivo ≠ papel de acesso — já separados; documentar só).
- Alterações de schema/Liquibase — nenhuma prevista (vínculo já existe no modelo de usuário).
- Apps público/árbitro — só admin_web; o contrato de claims fica pronto para reuso.

---

## 6. Riscos

| Risco | Mitigação |
|---|---|
| Usuário aprovado sem vínculo (nem org nem agremiação) | F1 define estado "pendente de vínculo" + tela clara (padrão já existe: `/pending-approval`) |
| Regressão para ADMIN | Escopo `PLATFORM` mantém comportamento idêntico; smoke checklist na F2 |
| Cache em memória dos repositórios (TTL 30s) vazar dado entre escopos | ESC-06 inclui chavear cache por escopo ou reduzir TTL nas listagens afetadas |
| Backend de outro repo em versionamento distinto | F1/F2 são issues de `flag_backend`; coordenar release antes de F4 |
