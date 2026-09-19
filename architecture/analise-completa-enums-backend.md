# Análise Completa de Enums: Backend × Requisitos de Jogo

> **Objetivo**: Comparar linha a linha o que o backend possui hoje, o que removemos do app por simplicidade de arbitragem e quais enums realmente **faz sentido incluir no backend** para suportar estatísticas completas no app público / súmula.

---

## 1. Comparativo Geral: O que o Backend tem vs O que Levantamos

| Categoria | Enum no Backend Atual | Enum no Levantamento Completo | Status / Recomendação para o Backend |
|---|---|---|---|
| **Pontuação** | `TOUCHDOWN` | `TOUCHDOWN` | ✅ Manter |
| **Pontuação** | `EXTRA_POINT_1` | `EXTRA_POINT_1` | ✅ Manter |
| **Pontuação** | `EXTRA_POINT_2` | `EXTRA_POINT_2` | ✅ Manter |
| **Pontuação** | `SAFETY` | `SAFETY` | ✅ Manter |
| **Pontuação** | `FIELD_GOAL` | `FIELD_GOAL` | ✅ Manter |
| **Pontuação** | `PICK_SIX` | `PICK_SIX` | ✅ Manter |
| **Pontuação** | `MINI_TOUCHDOWN` | `MINI_TOUCHDOWN` | ✅ Manter |
| **Pontuação** | ❌ *Ausente* | `PICK_TWO` | 🟢 **ADICIONAR**: Retorno defensivo no Ponto Extra (+2 pts defesa). |
| **Pontuação** | ❌ *Ausente* | `SCOOP_SIX` | 🟡 *Opcional*: Fumble retornado para TD (apenas Full Pads 11x11). |
| **Ataque** | `PASS` | `PASS` | ✅ Manter |
| **Ataque** | `INCOMPLETE_PASS` | `INCOMPLETE_PASS` | ✅ Manter |
| **Ataque** | `RUN` | `RUN` | ✅ Manter |
| **Ataque** | `FIRST_DOWN` | `FIRST_DOWN` | ⚠️ Manter no backend para registro interno/stats (mesmo que oculto do botão de input rápido no app). |
| **Ataque** | ❌ *Ausente* | `LATERAL_PASS` | 🟡 *Opcional*: Passe lateral / Pitch (pode ser registrado como `RUN` ou `PASS`). |
| **Ataque** | ❌ *Ausente* | `SPIKE` | 🟡 *Opcional*: Parar relógio (Full Pads / 8x8+). |
| **Ataque** | ❌ *Ausente* | `KNEEL` | 🟡 *Opcional*: Ajoelhar / gastar tempo. |
| **Defesa** | `SACK` | `SACK` | ✅ Manter |
| **Defesa** | `DEFLECTION` | `DEFLECTION` | ✅ Manter (Passe Desviado) |
| **Defesa** | `INTERCEPTION` | `INTERCEPTION` | ✅ Manter |
| **Defesa** | ❌ *Ausente* | `FLAG_PULL` | 🟢 **ADICIONAR**: Retirada de fita (Evento principal de defesa no Flag Football). |
| **Defesa** | ❌ *Ausente* | `TACKLE` | 🟢 **ADICIONAR**: Derrubada física (Evento principal de defesa no Full Pads 11x11). |
| **Defesa** | ❌ *Ausente* | `FUMBLE_FORCED` | 🟡 *Opcional*: Fumble forçado (Full Pads 11x11). |
| **Defesa** | ❌ *Ausente* | `FUMBLE_RECOVERY` | 🟡 *Opcional*: Recuperação de Fumble (Full Pads 11x11). |
| **Chutes** | `PUNT` | `PUNT` | ✅ Manter |
| **Chutes** | `KICKOFF` | `KICKOFF` | ✅ Manter |
| **Chutes** | ❌ *Ausente* | `TOUCHBACK` | 🟡 *Opcional*: Touchback no chute. |
| **Chutes** | ❌ *Ausente* | `PUNT_RETURN` | 🟡 *Opcional*: Retorno de Punt. |
| **Chutes** | ❌ *Ausente* | `KICKOFF_RETURN` | 🟡 *Opcional*: Retorno de Kickoff. |
| **Faltas** | `PENALTY` | `PENALTY` | ✅ Manter |

---

## 2. As 3 Adições Essenciais que Faltam no Backend

Para atender **todas as modalidades** (Flag 5x5, 7x7, 8x8, 9x9 e Full Pads 11x11) com precisão estatística, recomendamos adicionar apenas estas **3 constantes** no Java:

1. **`PICK_TWO`**:
   - Permite registrar a pontuação defensiva de 2 pontos no Ponto Extra.
   - Atualmente, sem ele, o app precisaria gambiarrar usando `EXTRA_POINT_2`.
2. **`FLAG_PULL`**:
   - É o evento defensivo básico do **Flag Football**. No Flag não existe "Tackle", existe "Retirada de Fita".
3. **`TACKLE`**:
   - É o evento defensivo básico do **Full Pads 11x11**.

---

## 3. Catálogo Final Recomendado para o Backend Java (20 Enums)

Com essa inclusão pontual das 3 constantes, o backend Java fica perfeitamente enxuto e completo:

```java
public enum PlayType {
    // Pontuação (8)
    TOUCHDOWN,
    EXTRA_POINT_1,
    EXTRA_POINT_2,
    PICK_TWO,            // ⬅️ NOVO: Retorno defensivo no PAT (+2 pts)
    SAFETY,
    FIELD_GOAL,
    PICK_SIX,
    MINI_TOUCHDOWN,

    // Ataque (4)
    PASS,
    INCOMPLETE_PASS,
    RUN,
    FIRST_DOWN,

    // Defesa (5)
    SACK,
    DEFLECTION,
    INTERCEPTION,
    FLAG_PULL,           // ⬅️ NOVO: Retirada de fita (Flag Football)
    TACKLE,              // ⬅️ NOVO: Derrubada (Full Pads 11x11)

    // Chutes (2)
    PUNT,
    KICKOFF,

    // Falta (1)
    PENALTY
}
```

---

## 4. O que isso muda no App Flutter (`flag_referee_app`)

- **Interface da Mesa (Árbitro)**: Mantém a botoeira enxuta e rápida (sem botões desnecessários que poluem a operação ao vivo).
- **Backend & App Público**: Recebem a informação exata se a jogada foi um `FLAG_PULL`, `TACKLE` ou `PICK_TWO` para montar estatísticas ricas sem complicar a vida do árbitro de mesa.
