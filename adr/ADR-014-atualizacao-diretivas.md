# ADR-014: Atualização de Diretivas em ADRs — Princípio de Documentos Vivos

## Status
Proposto

## Data
2026-09-11

## Autor
Tech Lead (Flag Platform)

---

## Contexto

Em projetos de longa duração, as diretrizes e requisitos evolutivos inevitavelmente mudam. Quando uma diretriz arquitetônica muda, a ADR correspondente não deve ser simplesmente "substituída" por uma nova ADR — isso cria fragmentação histórica e perda de contexto sobre por que decisões antigas foram tomadas. Em vez disso, a própria ADR deve ser reescrita para refletir o novo direcionamento, preservando o histórico da decisão original.

Isso é especialmente relevante para o Flag Platform, onde ADRs cobrem decisões fundamentais sobre hierarquia, roles, autenticação, CQRS, e essas podem evoluir conforme o produto amadurece.

## Decisão

**Princípio: ADRs são documentos vivos.** Quando uma diretriz muda:

1. **A ADR original NÃO é deletada** — ela permanece no histórico como registro da decisão tomada naquele momento
2. **Uma nova seção "Consequências Atualizadas" é adicionada** no final da ADR, documentando os novos efeitos da mudança
3. **Uma seção "Decisão Final" é atualizada** para refletir o estado atual, referenciando a decisão original quando aplicável
4. **Um link "Versão Anterior" é adicionado** na seção de rodapé, apontando para o estado original da ADR (se houver um histórico de mudanças)
5. **Nunca se cria uma "ADR-Nova" apenas para substituir** — a menos que a mudança seja tão radical que justifique um registro totalmente novo (ex: migração de arquitetura completa de monolito para microsserviços, o que viola ADR-003)

## Motivação

- **Contexto Histórico**: Preserva o "porquê" da decisão original, importante para auditoria e compreensão de evolução
- **Rastreabilidade**: Evita perda de informações quando a equipe muda ou novos membros chegam
- **Menor Ruído**: Em vez de 3 ADRs "ADR-001, ADR-015, ADR-020" sobre o mesmo tema, temos 1 ADR atualizada com histórico claro
- **Conformidade com Gitflow**: A atualização acontece na mesma branch `develop` → `main`, com o PR servindo como registro da mudança

## Quando Aplicar essa Regra

Aplicar quando houver mudança em:

1. **Hierarquia de entidades** (ex: reorganização de Organization → Club → Time)
2. **Roles e permissões** (ex: expansão ou redução de roles)
3. **Stack tecnológica** (ex: mudar de Firebase Auth para outro IdP)
4. **Estratégia de dados** (ex: PostgreSQL ↔ Firestore sync mudar de modelo)
5. **Convenções de nomenclatura** (ex: renaming de campos ou entidades)
6. **Ferramentas ou infraestrutura** (ex: mudar de Cloud Functions para outro provedor)

## Não Aplicar quando:

1. A mudança é **temporária** (ex: feature flag, configuração ambiente-specific) — nesses casos, usar comentários no código ou configs de ambiente
2. O cenário é **experimental** e será revertido em breve — manter como anotação temporária, não ADR
3. É apenas **ajuste menor de documentação** (formatação, links corrigidos) — não requer reescrita da decisão

## Formato da Atualização

Quando aplicar, a ADR deve ser atualizada seguindo este modelo na seção final:

```markdown
## Histórico de Atualizações

| Data | Versão | Mudança | Motivo | Autor |
|------|--------|---------|--------|-------|
| 2026-09-11 | 1.1 | Diretiva de roles expandida de 3 para 4 layers | Necessidade de controle mais granular de acesso por clube | Tech Lead |
| 2026-05-05 | 1.0 | Versão original | Decisão arquitetural inicial | Tech Lead |

---

## Decisão Atualizada (update)

**Anterior:** [Descrever decisão original]

**Atual:** [Descrever nova direção, preservando o essência da decisão original quando possível]

**Motivo da Atualização:** [Por que a mudança era necessária]

**Impacto nas outras ADRs:** [Quais ADRs são afetadas por esta mudança, ex: esta mudança impacta ADR-006 que define a hierarquia de times]

---

## Consequências Atualizadas

### Positivas
- ✅ [Novo benefício após a mudança]

### Negativas
- ⚠️ [Novo risco ou consequência após a mudança]

### Riscos Mitigados
- ✅ [Como os riscos foram mitigados com esta mudança]

---

## Referências Cruzadas

- **ADR original:** ADR-001 — Filosofia do Projeto (versão 1.0, data: 2026-09-05)
- **Impacto:** Afeta ADR-006 (Team/Roster/Season), ADR-010 (Autenticação Firebase-First)
- **Relacionadas:** ADR-003 (Modular Monolith), ADR-004 (API First)

---

## Critérios de Aceitação da Atualização

- [ ] A decisão original é preservada na seção "Histórico de Atualizações"
- [ ] A nova direção é claramente documentada na seção "Decisão Atualizada"
- [ ] Impactos em outras ADRs são explicitamente listados na seção "Referências Cruzadas"
- [ ] A atualização foi discutida e aprovada em Pull Request (seguindo o fluxo gitflow da ADR-013)
- [ ] Tags de versão são atualizadas refletindo o novo estado da ADR

## Exemplo Prático: Atualização de ADR-001

Se no futuro a diretriz de "Hierarquia org→clube→time→elenco→atleta" for revisada para adicionar um nível intermediário:

1. A ADR-001 existente seria mantida com sua versão 1.0
2. Novas seções seriam adicionadas no final:
   - "Histórico de Atualizações" com entrada para a nova versão
   - "Decisão Atualizada" refletindo a nova hierarquia
   - "Consequências Atualizadas" com benefícios e riscos novos
3. Um Pull Request seria aberto na branch `develop` para `main`
4. Tag `v1.1-adr` seria criada após merge

---

## Integração com Gitflow (ADR-013)

Esta ADR trabalha em conjunto com a ADR-013 (Gitflow para Documentação):

1. Atualizações de diretivas acontecem em branch `feature/doc-atualizacao-<assunto>`
2. Pull Request para `develop` com a ADR reescrita
3. Após merge, tag de versionamento é criada (ex: `v1.1-adr`)
4. O histórico fica preservado tanto no Git (commits/PRs) quanto no conteúdo da ADR

---

## Referências

- Padrão de documentos vivos em projetos de arquitetura de software
- Convenção de versionamento semântico aplicada a ADRs
- GitFlow para Documentação (ADR-013)
- Padrão de "Decision Records" atuais (adr-tools, provengo, etc.)