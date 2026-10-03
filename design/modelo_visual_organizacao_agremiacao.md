# Proposta Visual e Arquitetural: Organizações & Agremiações

Esta proposta alinha visualmente e funcionalmente as telas de **Agremiações** e **Organizações**, estabelecendo um padrão consistente de busca/filtros, grade de 2 colunas, componentes de card harmonizados e atualização dinâmica contínua com o banco de dados.

---

## 1. Modelo Visual Proposto

![Mockup Visual - Lista de Agremiações e Organizações](file:///C:/Users/Cesar/.gemini/antigravity-cli/brain/0014bce1-e955-4674-bef1-6aa587cfc764/admin_list_redesign_preview_1788818622476.jpg)

---

## 2. Detalhamento dos Componentes

### A. Barra de Pesquisa & Filtro Unificada
Tanto em **Agremiações** quanto em **Organizações**, a barra superior de filtros passará a utilizar a mesma composição em `Card` Kickster:
* **Input de Busca Kickster**:
  - Estilo pill (altura ~52px, raio 24px, fundo `surface`, borda sutil).
  - Ícone de lupa à esquerda (`Icons.search`), texto de dica `Buscar por nome...` e botão de limpar busca (`Icons.close`).
  - Debounce integrado para digitação fluida.
* **Divisor Vertical**: Separação elegante de 1px entre a busca e o filtro.
* **Dropdown de Tipos Kickster**:
  - `KicksterDropdown`: pill com raio 24px, rótulo das opções com ícones e checkbox circular para seleção rápida (`Todos os tipos`, `Clubes`, `Universidades`, etc.).

```
+-----------------------------------------------------------------------------------------------+
|  [ 🔍 Buscar por nome...                    (X) ]  |  [ Filtrar por tipo: Todos os tipos  ▼ ] |
+-----------------------------------------------------------------------------------------------+
```

---

### B. Grade Responsiva em 2 Colunas
* **Substituição da lista vertical**: `ListView.separated` é substituído por um `GridView.builder` de **2 colunas** com espaçamento de 12px (em telas desktop/tablet) e 1 coluna (telas menores/mobile).
* Permite visualizar o dobro de entidades por viewport, otimizando o uso do espaço de tela na Web.

---

### C. Padronização dos Cards da Lista
Os cards de agremiação e organização seguirão o mesmo padrão Kickster:
* **Leading**: Logo com borda arredondada (48x48px) ou ícone esportivo padrão.
* **Título**: Nome Fantasia (`tradeName`) em SemiBold 16px.
* **Subtítulo**: Razão Social ou Tipo (`CLUB` / `UNIVERSITY`) + Sigla em 13px.
* **Trailing**:
  - Badge de status (`KicksterBadge`) quando desativada.
  - Indicador de carregamento (`CircularProgressIndicator`) durante ações assíncronas.
  - Menu de ações Kickster (`PopupMenuButton` com opções: *Editar*, *Desativar/Excluir* e *Reativar*).

---

### D. Atualizações Dinâmicas e Reatividade com o Banco de Dados
Para garantir que as alterações no banco reflitam imediatamente ao navegar:
1. **Invalidação de Cache no Repositório**:
   - Ajustar `OrganizationRepository` e `InstitutionRepository` para limpar cache nas mutações e suportar recarga imediata.
2. **Ciclo de Vida do ViewModel**:
   - Chamar `load(forceRefresh: true)` ao entrar nas telas de listagem, garantindo dados frescos do backend a cada navegação.
3. **Carregamento de Detalhes sem dependência de cache stale**:
   - `OrganizationDetailScreen` e `InstitutionDetailScreen` carregarão os dados reais diretamente pelo ID através de seus ViewModels/Providers, eliminando estados defasados.

---

Você aprova este modelo visual e estrutura para iniciarmos a implementação?
