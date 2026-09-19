# Modelo Visual: Harmonização dos Formulários de Agremiação e Organização

![Harmonização de Formulários](C:\Users\Cesar\.gemini\antigravity-cli\brain\0014bce1-e955-4674-bef1-6aa587cfc764\harmonizacao_forms_admin_1788822988849.jpg)

---

## 1. Resposta sobre Componentes no Core (`lib/src/core`)

> [!NOTE]
> **Sim, praticamente todos os blocos de construção essenciais já estão centralizados no Core**:
> - [`KicksterInput`](file:///C:/Projetos/America/flag_admin_web/lib/src/core/widgets/kickster_input.dart): Campos de texto padrão pill 52px com raio 24.
> - [`KicksterDropdown`](file:///C:/Projetos/America/flag_admin_web/lib/src/core/widgets/kickster_dropdown.dart): Dropdown fechado em pill 52px com menu segmentado.
> - [`KicksterSectionTitle`](file:///C:/Projetos/America/flag_admin_web/lib/src/core/widgets/kickster_section_title.dart): Títulos de seções com ícone temático.
> - [`KicksterButton`](file:///C:/Projetos/America/flag_admin_web/lib/src/core/widgets/kickster_button.dart): Botões primários e outline.
> - [`KicksterImageUploader`](file:///C:/Projetos/America/flag_admin_web/lib/src/core/widgets/kickster_image_uploader.dart): Upload de logos e imagens.
> - [`DocumentUtils`](file:///C:/Projetos/America/flag_admin_web/lib/src/core/utils/document_utils.dart): Validações e máscaras para CPF e CNPJ.
>
> **Oportunidade de Melhoria**:
> A lista de UFs brasileiras e o seletor de País/Estado/Cidade podem ser consolidados de forma idêntica em ambos os formulários.

---

## 2. Mudanças Específicas por Formulário

### A. Agremiação (Cadastro e Edição - [`InstitutionFormScreen`](file:///C:/Projetos/America/flag_admin_web/lib/ui/institution/widgets/institution_form_screen.dart))
1. **Dados Básicos**:
   - Remoção dos labels dos componentes dropdown (`KicksterDropdown` de *Tipo de Agremiação* e de *Tipo de Documento* passam a ter `label: ''` com hint visual integrado ao pill).
   - Disposição harmonizada:
     - Linha 1: *Nome Fantasia*
     - Linha 2: *Razão Social*
     - Linha 3: *Tipo de Agremiação* (sem label, hint *"Tipo"*) + *Sigla / Abreviação*
     - Linha 4: *Tipo de Documento* (sem label, hint *"Tipo de Doc"*) + *Número do Documento*
2. **Localização**:
   - Padronizada exatamente como Organização:
     - *País* (`KicksterDropdown` com Brasil e opções internacionais).
     - *Estado*: Se Brasil (`BR`), dropdown com as 27 UFs (`São Paulo (SP)`, etc.); se exterior, campo de texto livre.
     - *Cidade*: Campo de texto.
3. **Filiação a Organizações**:
   - **Removida completamente** deste formulário (a inscrição e filiação serão gerenciadas na futura tela dedicada de inscrições).

---

### B. Organização (Cadastro e Edição - [`OrganizationCreateScreen`](file:///C:/Projetos/America/flag_admin_web/lib/ui/organization/widgets/organization_create_screen.dart))
1. **Harmonização de Disposição com Agremiação**:
   - **Dados Básicos**:
     - *Nome Fantasia*
     - *Razão Social*
     - Linha com *Tipo de Organização* (sem label) + *Sigla*
     - Linha com *CNPJ*
   - **Presidente / Diretoria**:
     - Disposição em 2 colunas: *Nome do Presidente* (2/3 da largura) + *CPF do Presidente* (1/3 da largura) em uma única linha harmoniosa, exatamente como em Agremiação.
   - **Contato**:
     - Disposição em 2 colunas:
       - Linha 1: *E-mail* + *Telefone*
       - Linha 2: *Site* + *Instagram (@)*
   - **Localização**:
     - Mantém o padrão estruturado (País + Estado UF + Cidade).
