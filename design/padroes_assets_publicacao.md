# Padrões Oficiais de Assets para Publicação (Mobile & Web)

Este documento estabelece a especificação mandante para os assets visuais de publicação dos aplicativos mobile (`flag_public_app` e `flag_referee_app`) na Google Play Store e da aplicação web (`flag_admin_web`).

---

## 1. Identidade Visual e Paleta de Cores

Todas as aplicações seguem a paleta e tokens do **UI Kit Kickster** adotados no Flag Platform:

- **Azul Royal Primário**: `#083879` (Fundo de ícones e barras de navegação)
- **Midnight Navy**: `#041833` / `#0B192C` (Contraste escuro, gradientes de feature graphic e listras de árbitro)
- **Branco Puro**: `#FFFFFF` (Corpo principal da bola de futebol americano)
- **Amarelo Penalty Flag / Destaque**: `#FACC15` (Costura da bola do Referee App e badges das fichas)
- **Cinza Suave / Surface**: `#FEFEFE` (Fundo dos splashes nativos)

---

## 2. Diferenciação Visual das Aplicações

| Aplicação | Público-Alvo | Motivo Visual Principal | Cores do Ícone |
|---|---|---|---|
| **`flag_public_app`** (Android) | Atletas, torcedores e público geral | Bola de futebol americano Kickster inclinada a -18° | Bola branca sobre fundo azul royal (`#083879`) com costura azul royal |
| **`flag_referee_app`** (Android) | Árbitros, mesários e delegados | Bola de futebol americano com listras verticais de árbitro | Bola branca com listras navy (`#0B192C`), costura em amarelo penalty flag (`#FACC15`) sobre azul royal |
| **`flag_admin_web`** (Web) | Organizadores, federações e ligas | Ícone Kickster e PWA otimizado para navegadores | Bola branca sobre fundo azul royal (`#083879`), com zona segura W3C para ícones maskable |

---

## 3. Especificações Técnicas de Publicação

### 3.1 Google Play Store (Android)

Exigido obrigatoriamente para publicação da ficha na Play Console:

1. **Ícone do App da Ficha (App Icon)**:
   - **Dimensão**: Exatamente `512 x 512 px`.
   - **Formato**: PNG de 32 bits (com canal alfa).
   - **Tamanho Máximo**: 1 MB.
   - **Regra**: Enviar com fundo pleno preenchido até as bordas (sem pré-arredondar os cantos nem aplicar sombra externa, pois o Google Play aplica os cantos arredondados dinamicamente com raio de 20%).

2. **Gráfico de Recursos (Feature Graphic)**:
   - **Dimensão**: Exatamente `1024 x 500 px`.
   - **Formato**: PNG ou JPEG (24 bits, sem transparência).
   - **Tamanho Máximo**: 15 MB.
   - **Composição**: Gradiente esportivo `#041833` a `#083879`, linhas diagonais sutis de campo, motivo da bola à esquerda e tipografia oficial "FLAG PLATFORM", badge do app e subtítulo à direita. Elementos mantidos na zona segura central para não sofrer cortes em telas de diferentes proporções.

3. **Assets Internos do Pacote Android (Launcher & Splash)**:
   - **Adaptive Icon (Android 8+)**: Foreground de `1024 x 1024 px` em fundo transparente, contido na zona segura central de ~66%, com background XML `#083879`.
   - **Ícone Legado (Mipmaps)**: Gerado automaticamente em mdpi, hdpi, xhdpi, xxhdpi e xxxhdpi.
   - **Splash Nativo (Legacy)**: Imagem de abertura `1024 x 1024 px` sobre fundo `#FEFEFE`.
   - **Splash Nativo (Android 12+)**: Imagem centralizada de `1152 x 1152 px`, com o logo ocupando até 45% do diâmetro total para respeitar o corte circular do Android 12.

### 3.2 Flutter Web (`flag_admin_web`)

Padrões da W3C para Progressive Web Apps (PWA) e navegadores modernos:

1. **Favicon**:
   - `web/favicon.png`: `48 x 48 px`, nítido em abas desktop e mobile.
2. **Ícones PWA**:
   - `web/icons/Icon-192.png`: `192 x 192 px`
   - `web/icons/Icon-512.png`: `512 x 512 px`
3. **Ícones Maskable (PWA Safe Zone)**:
   - `web/icons/Icon-maskable-192.png`: `192 x 192 px` (conteúdo contido no círculo de 80% do diâmetro).
   - `web/icons/Icon-maskable-512.png`: `512 x 512 px` (conteúdo contido no círculo de 80% do diâmetro).
4. **Manifesto e Metatags**:
   - `web/manifest.json`: Background e theme color `#083879`, título "Flag Platform - Painel Administrativo".
   - `web/index.html`: Metatag `theme-color` `#083879`, Apple touch icon e título oficial.

---

## 4. Scripts Utilitários de Geração

Todos os repositórios possuem scripts automatizados em PowerShell utilizando a API GDI+ (.NET) nativa do Windows, sem necessidade de softwares gráficos externos:

- `flag_public_app/tool/generate_brand_assets.ps1`
- `flag_referee_app/tool/generate_brand_assets.ps1`
- `flag_admin_web/tool/generate_brand_assets.ps1`

### Fluxo de Atualização dos Apps Mobile:

```powershell
# 1. Gerar os PNGs matematicamente calibrados
powershell -ExecutionPolicy Bypass -File .\tool\generate_brand_assets.ps1

# 2. Propagar para mipmaps nativos e splash
dart run flutter_launcher_icons
dart run flutter_native_splash:create
```
