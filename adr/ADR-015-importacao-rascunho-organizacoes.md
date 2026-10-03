# ADR-015: Modo de importação em rascunho de organizações (`importDraft`)

**Status:** Proposto
**Data:** 2026-09-23
**Autor:** Tech Lead (Flag Platform)
**Contexto relacionado:** `america_platform_agent/docs/02-gap-analysis-organizacoes.md`

---

## Contexto

O agente de coleta (`america_platform_agent`) descobre, coleta e extrai dados de
organizações de flag football a partir de **fontes públicas** (sites oficiais,
Instagram, Facebook) para acelerar o cadastro inicial de federações, ligas e associações.

Ao tentar persistir via `POST /api/v1/organizations`, o agente esbarra em um bloqueio:

- `OrganizationService.create()` chama `validatePresident(presidentCpf)`, que **exige**
  um CPF de presidente **válido** (dígitos verificadores conferidos por `DocumentValidator`).
- `presidentCpf` **não é coletável** de fontes públicas com confiabilidade; quando
  aparece, é **dado pessoal** cuja coleta automatizada para cadastro é questionável sob a LGPD.

Resultado: nenhuma organização pode ser semeada automaticamente, mesmo quando todos os
demais campos (`tradeName`, `legalName`, `organizationType`, contatos, endereço, logo)
estão disponíveis e corretos. Isso anula o principal ganho do agente.

---

## Decisão

Adicionar um **modo de importação em rascunho** à criação de organizações, opt-in e
restrito, que relaxa a exigência do CPF e cria a organização como **`INACTIVE`** para
posterior complemento humano na Admin Web.

### Concretização

1. Novo parâmetro **`importDraft`** (boolean, default `false`) no
   `POST /api/v1/organizations` — como `@RequestParam` (não polui o corpo do
   `CreateOrganizationRequest`, mantendo o contrato atual).
2. Quando `importDraft=true`:
   - **Pula** `validatePresident(...)`.
   - `presidentCpf` inválido/ausente é **descartado** (não persiste dado sujo).
   - `status` é forçado para **`INACTIVE`** (organização não aparece em listagens públicas).
   - Mantém as demais validações (`tradeName` único, `organizationType`, `legalName`,
     `country`, `timezone`, `locale`).
3. Quando `importDraft=false` (default): comportamento **inalterado**.
4. Autorização: mesma do create (`ADMIN_ORGANIZER`).
5. Documentar no Swagger/OpenAPI, deixando explícito que rascunhos devem ser
   **revisados e ativados** por um operador (via `PUT` preenchendo `presidentCpf` +
   `POST /{id}/reactivate`).

### Alternativas consideradas

| Alternativa | Por que não |
|---|---|
| Nova tabela/endpoint `/imports` | Duplica o modelo e as validações; mais superfície para manter |
| Novo status `DRAFT`/`IMPORTED` no enum | Aumenta o enum e exige tratamento em todas as listagens; `INACTIVE` já expressa "não publicada" |
| Tornar `presidentCpf` opcional globalmente | Enfraquece a regra para todos os fluxos, inclusive cadastro manual legítimo |
| Seed via SQL/Flyway direto no Postgres | Fura as regras de negócio e a auditoria (`createdBy`); não escala para o agente |

---

## Consequências

### Positivas
- Desbloqueia a semeadura assistida de organizações a partir de dados públicos.
- Mantém intactas as garantias do cadastro manual (não vaza CPF obrigatório).
- Rascunhos ficam invisíveis ao público até revisão humana.

### Negativas / cuidados
- Introduz um caminho em que `presidentCpf` fica nulo — relatórios/consultas devem
  tolerar isso (hoje o campo já é anulável em `OrganizationResponse`).
- Requer disciplina de operação para ativar ou descartar rascunhos (evitar acúmulo de
  `INACTIVE` órfãos). Mitigação: relatório `data/processed/gaps.md` do agente + filtro
  `includeDisabled=true` (ADMIN).
- **Não resolve** a questão de proveniência: idealmente a organização deve registrar
  `source_url`/origem da coleta. Fica como evolução futura (campo opcional de origem).

---

## Impacto por repositório

- **flag_backend:** `OrganizationApi` (novo `@RequestParam`), `OrganizationService.create`
  (ramo `importDraft`), testes de contrato do endpoint e `@Operation` no Swagger.
- **flag_admin_web:** nenhuma mudança obrigatória; opcionalmente um indicador de
  "rascunho/INACTIVE a revisar" na listagem.
- **america_platform_agent:** `FlagApiClient.create_organization` passa a enviar
  `?importDraft=true` quando o perfil for "rascunho" (evolução; hoje o agente opera
  em `--export-only` + relatório de lacunas até este ADR ser implementado).

---

## Referências

- `flag_backend/src/main/java/br/com/flagplatform/organization/service/OrganizationService.java`
- `flag_backend/src/main/java/br/com/flagplatform/organization/dto/request/CreateOrganizationRequest.java`
- `america_platform_agent/docs/02-gap-analysis-organizacoes.md`
