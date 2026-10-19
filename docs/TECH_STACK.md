# Tecnologias e arquitetura — Pro League FullStats

Esta página descreve **o papel observado** de cada tecnologia na edição FullStats. O código-fonte do mod e as cenas do jogo não são distribuídos aqui.

| Tecnologia ou ferramenta | Uso no projeto | Limite da evidência |
| --- | --- | --- |
| C# | Regras de partida, relógio, estado de pausa, pontuação e apresentação de resultados. | Scripts locais foram inspecionados, sem serem copiados para este repositório. |
| Unity / Unity UI | `Canvas`, `Text` e `RectTransform` usados pelo HUD, placar, relógio e telas de resultado. | A análise estática mostra as integrações; a aparência final exige captura do jogo. |
| DynamicCSharp / Battlehub | `CustomScript` embutidos na cena e compilação pelo Mod Editor; serialização da cena e metadados pelo Battlehub RTSaveLoad. | Compilar externamente não substitui a compilação e execução no editor. |
| RPCs do jogo | O anfitrião sincroniza configuração, relógio, pontos, estatísticas e transições de round com os clientes. | A presença das chamadas não é prova isolada da qualidade da sincronização online. |
| Steam Workshop | Distribuição da edição publicada. | O [item público](https://steamcommunity.com/sharedfiles/filedetails/?id=3821094797) confirma título, publicação e créditos; métricas de uso não são apresentadas como resultado do projeto. |
| Git, cópias de trabalho e backups | Controle desta documentação e preservação de estados durante a investigação do mod. | Os backups do mod ficam fora deste repositório. |
| Codex | Assistência para análise, geração/adaptação de código, documentação e depuração. | Atribuição explícita; não há alegação de implementação manual integral. |

## Fluxo entre componentes

```text
Seleção do modo pelo anfitrião
  └─ TimeManager_PL: configuração, relógio, pausa e RPCs
       ├─ RoundManager: round atual, duração e vitórias
       ├─ PointsManager_PL: pontos e bloqueio em pausa/intervalo
       ├─ ShowControls_PL: placar, relógio e contagem do intervalo
       └─ PLShowWinner: estatísticas, resultado e decisão de empate
```

O relógio do round é diferente do tempo nativo da sala. A edição sincroniza os segundos restantes e os apresenta no HUD. `PointsManager_PL` consulta o estado de partida, pausa e intervalo antes de aceitar eventos de pontuação. Na apuração, dados de estatísticas são capturados antes da redefinição de jogadores para que o painel entre rounds não perca pontos e bônus.

## Compilação e restrições do ambiente

Os registros locais mostram dois tipos de verificação:

1. **Compilação externa:** C# compilado com `csc.exe` contra DLLs instaladas do jogo. Uma etapa histórica terminou com código de saída 0 e dois avisos preexistentes.
2. **Compilação interna:** logs do Battlehub/Mod Editor registraram `compiled OK`. Um registro inicial confirmou cinco scripts individualmente; em um experimento posterior de FullStats AllModes, o log registrou os seis scripts e `GameStarted()` sem erro CS. Esse log posterior não identificava nominalmente a cena selecionada, então não serve sozinho como confirmação visual ou funcional do Title Fight.

Alguns símbolos de UI, incluindo `TextAnchor`, `FontStyle.Bold` e `HorizontalWrapMode`, causaram incompatibilidade em uma etapa de teste. O conjunto validado externamente evitou esses símbolos e referências diretas ao namespace `DW2.`. Também havia risco de truncamento ao colar scripts longos no campo `CustomScript`; por isso os experimentos verificaram o texto embutido na cópia da cena. Mensagens `Type mismatch!` apareceram em uma recompilação histórica sem erro CS. Esses sinais foram registrados, não tratados como prova automática de falha ou sucesso em jogo.

## Estratégia de depuração

- Inspecionar o fluxo da cena e distinguir tempo de sala de tempo do round.
- Comparar scripts legíveis com o texto embutido na cena de trabalho.
- Compilar externamente para detectar sintaxe e referências indisponíveis.
- Abrir a cópia no Mod Editor, ler logs e conferir UI e transições em jogo.
- Testar anfitrião e cliente para pontos, bônus, relógio, pausa e mudança de round.
- Preservar backups antes de cada alteração e documentar o que foi efetivamente observado.

Veja [o estudo de caso](CASE_STUDY.md) para problemas, decisões e resultados. Os detalhes de implementação são descritos sem publicar código ou assets de terceiros.
