# REF-05: Gestão sequencial de descidas (downs), turnover on downs automático e auto-reset

> **Type:** regra
> **Status:** open
> **Effort:** repos/referee_app
> **Repo(s):** flag_referee_app
> **Blocks:** —
> **Blocked by:** —
> **Origin:** relato do usuário — 2026-10-11

## Sintoma

1. A operação não controla automaticamente as descidas na sequência do drive (1ª, 2ª, 3ª e 4ª descidas).
2. Na 4ª descida sem conquista de First Down, o app não executa nem oferece a marcação de Turnover on Downs.
3. Após o registro de cada jogada, os seletores da tela não resetam para o estado padrão, retendo seleções antigas.

## Correção proposta

1. **Automação de Descidas**:
   - Drive se inicia em 1ª descida.
   - Jogadas normais (sem First Down e sem turnover) avançam para a próxima descida: 1ª → 2ª → 3ª → 4ª.
   - Se na 4ª descida o time não converter nem pontuar: registrar automaticamente **Turnover on Downs**, inverter a posse de bola e iniciar a 1ª descida da outra equipe no ponto da bola.
2. **Auto-Reset pós-Lance**:
   - Ao confirmar o registro de qualquer jogada, zerar todos os seletores secundários (jardas, atletas selecionados, tipo de falta) para o padrão limpo da tela.

## Critérios de aceite

- [ ] Descidas avançam automaticamente conforme as regras esportivas.
- [ ] 4ª descida falha inverte a posse (Turnover on Downs) e reseta para 1ª descida.
- [ ] Formulário/botoeira reseta para o default após cada lance registrado.
