# ADR-013: Gitflow para Gestão de Documentação

## Status
Proposto

## Data
2026-09-11

## Autor
Tech Lead (Flag Platform)

---

## Contexto

A Flag Platform adota gitflow para desenvolvimento de código, mas a gestão de documentação ainda não estava alinhada a esse fluxo. Documentos em `docs/` precisam de um processo consistente que funcione com branches feature, develop, release e hotfix, sem depender de issues formais no GitHub para criação.

## Decisão

Adotar gitflow para ciclo de vida da documentação, com as seguintes fases:

### Branch `main` (Produção)
- Apenas documentos **publicados e versionados**
- Qualquer alteração deve passar por branch `release` ou `hotfix`
- Tags de release (ex: `v1.0-docs`, `v1.1-docs`) marcam versões estáveis
- O site Jekyll é gerado a partir do conteúdo nesta branch

### Branch `develop` (Integração)
- Documentos em desenvolvimento ou revisão
- Conflitos de merge são resolvidos aqui antes de promover a `main`
- PRs (pull requests) são o mecanismo de entrada, mas **não** exigem issue formal — o próprio PR serve como rastreamento

### Branches `feature/*` (Novos Documentos)
- Criados para cada novo documento ou reestruturação major
- Nome: `feature/doc-<tipo>-<assunto>` (ex: `feature/doc-adr-hierarquia`, `feature/doc-design-tokens-v2`)
- Quando pronto, abre-se um Pull Request para `develop`
- **Critério de aceitação do PR**: documentação segue convenções, links verificados, consistência com ADRs existentes
- **Não é criada issue separada** — o PR é o rastreamento

### Branch `release/*` ( Pré-lançamento)
- Criado a partir de `develop` quando há um conjunto de documentos pronto para versão
- Nome: `release/v1.0-docs`, `release/v1.1-docs`, etc.
- Última chance de correções antes de promover a `main`
- Tag de release é criada aqui

### Branch `hotfix/*` (Correções Urgentes)
- Criado a partir de `main` quando um documento crítico tem erro (link quebrado, dado desatualizado)
- Nome: `hotfix/doc-corrige-links-adr-001`
- Após correção, merge para `main` e `develop`

## Fluxo de Trabalho (Gitflow Docs)

```
feature branch
    ↓
Pull Request para develop
    ↓
develop branch
    ↓
release branch (quando houver conjunto de mudanças)
    ↓
main branch + tag de versionamento
    ↓
Deploy no site Jekyll
```

## Vantagens

- ✅ **Sem custo de issues formais** — o PR é suficiente para rastrear mudanças
- ✅ **Versionamento claro** — tags indicam qual versão da docs está publicada
- ✅ **Integração com código** — mudanças de documentação acompanham mudanças de código nas mesmas branches
- ✅ **Rollback fácil** — hotfix branches corrigem docs rapidamente
- ✅ **Pull Request como documento de revisão** — discussão e feedback acontecem no PR

## Desvantagens

- ⚠️ **Exige familiaridade com gitflow** — equipe precisa estar confortável com branches e PRs
- ⚠️ **Menor rastreabilidade pontual** — não há número de issue para referência cruzada rápida (compensado pelo nome do branch/Tag)

## Critérios de Aceitação

- [ ] Novo documento criado em branch `feature/doc-<tipo>-<assunto>`
- [ ] Documento segue convenções de formatação padrões (headings, links, convenção de nomes)
- [ ] PR aberto para `develop` com título claro e descrição do conteúdo
- [ ] Pelo menos 1 revisão aprova o conteúdo e a formatação
- [ ] Quando promovido a `main`, tag de release é criada (ex: `v1.0-docs`)
- [ ] Site Jekyll rebuild bem-sucedido após merge para `main`

## Alternativas Consideradas

### A. Apenas master branch (sem gitflow)
- **Prós**: Simples, sem necessidade de aprender gitflow
- **Contras**: Nenhum controle de versão de documentos, risco de sobrescrever docs produzidas, difícil rollback

### B. Issues formais + branches soltas
- **Prós**: Rastreabilidade via numbers de issue
- **Contras**: **Descartado** — eleva custo do projeto (como solicitado), overhead de criar issues para cada documento pequeno

### C. Documentos soltos sem versionamento
- **Prós**: Sem complexidade de git
- **Contras**: Documentação desatualizada rapidamente, impossível rastrear mudanças, nenhum histórico

---

## Decisão Final

**Adotar gitflow para gestão de documentação** (this ADR), com as seguintes regras chave:

1. **Nunca criar issue formal para documento novo** — usar branch `feature/` + Pull Request
2. **Sempre promover através de develop → main** com tags de versionamento
3. **Hotfix para correções críticas** apenas a partir de `main`
4. **Tags de release** indicam versão da documentação publicada no site

---

## Plano de Implementação

### Fase 1: Configuração (Imediato)
- [ ] Documentar este ADR e garantir que a equipe esteja ciente
- [ ] Garantir que todos os desenvolvedores possem acesso Git com fluxo gitflow
- [ ] Criar modelo de PR template para documentos (`docs/PULL_REQUEST_TEMPLATE.md`)

### Fase 2: Migração de Documentos Existentes
- [ ] Mover documentos atuais para fluxo `feature/` + `develop` → `main`
- [ ] Criar tags iniciais: `v0.1-docs` (estado atual), `v1.0-docs` (primeira versão estruturada)

### Fase 3: Rotina Contínua
- [] Novos documentos: branch `feature/doc-...` → PR para `develop`
- [] Revisões: feedback no próprio PR, sem issue separada
- [] Versionamento: tag a cada release de docs (a cada 2-3 meses ou a cada release de produto)
- [] Hotfix: somente para documentos críticos com links quebrados ou dados errados

---

## Referências

- [GitFlow Success Model](https://nvie.com/posts/gitflow-success-model/)
- [Florian Baier's gitflow extension](https://github.com/nvie/gitflow)
- Padrão atual de ADRs neste repositório