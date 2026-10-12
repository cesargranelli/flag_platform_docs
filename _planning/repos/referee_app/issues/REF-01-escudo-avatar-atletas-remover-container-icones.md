# REF-01: Substituição de foto por escudo nos avatares & desencapsulamento de ícones

> **Type:** design
> **Status:** done
> **Effort:** repos/referee_app
> **Repo(s):** flag_referee_app
> **Blocks:** —
> **Blocked by:** —
> **Origin:** relato do usuário — 2026-10-11

## Sintoma

1. O avatar dos atletas nas telas de check-in e detalhe exibe iniciais ou espera uma foto individual que não existe cadastrada na base.
2. Na botoeira de jogadas e nos diálogos, os ícones de ação e jogadas estão cercados por uma caixa/container com borda, gerando ruído visual e ocupando espaço desnecessário no tablet e mobile.

## Correção proposta

1. **Avatar dos Atletas (`AthleteCheckInCard`, `KicksterAvatar`)**:
   - Apresentar o escudo da equipe (`KicksterTeamShield`) no avatar do atleta, mantendo o badge com o número da camisa `#N` posicionado no canto inferior direito.
2. **Desencapsulamento dos Ícones**:
   - Remover as caixas/containers que envolvem os ícones das jogadas no `PlayPadWidget` e `PlayDialog`, tornando os botões mais limpos, táteis e ágeis.

## Critérios de aceite

- [x] Avatar de cada atleta exibe o escudo de seu time com badge `#N` da camisa.
- [x] Botões de lances não possuem caixas/containers pesados isolando o ícone.
