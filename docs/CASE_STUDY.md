# Estudo de caso — Pro League Rounds Fullstats Edition

## 1. Objetivo

Evoluir um mod de rounds de Drunken Wrestlers 2 para apresentar regras de partida, cronômetro no HUD, estatísticas completas e desempate de modo consistente para os jogadores. O ponto de partida foi o projeto original de Nixagorn. O trabalho desta edição reuniu integrações, interface, depuração, testes e publicação, com assistência do Codex na análise e na geração ou adaptação de código.

## 2. Problemas investigados

- O fluxo de rounds precisava conviver com o tempo nativo da sala. Na investigação inicial, houve relato de reinício entre rounds, mas ele **não foi reproduzido naquela etapa**. O ciclo de vida da cena e o cronômetro nativo foram tratados como hipóteses de causa.
- O estado de pausa precisava ser respeitado pelo relógio e pelo cálculo de pontos. Sem uma barreira única, uma pontuação poderia ser recebida enquanto a partida aparentava estar parada.
- A versão original analisada não tinha uma regra explícita para empate final por pontos. A escolha do primeiro jogador da lista em um empate era um risco observado na leitura do código, não um resultado de partida medido.
- O relógio precisava aparecer no HUD sem conflitar visualmente com componentes nativos.
- Em um experimento FullStats, o painel de estatísticas completo desaparecia antes do fim do intervalo, embora um resumo simplificado permanecesse visível. O histórico local identificou duas causas no fluxo: `SetIntermission` desabilitava os textos originais e uma rotina os limpava cerca de 25 segundos depois de mostrá-los.
- Mensagens do jogo e do Mod Editor podiam divergir. Uma compilação externa bem-sucedida não comprovava compilação interna, interação de UI ou sincronização online.

## 3. Solução e arquitetura

O fluxo foi organizado ao redor de estado de partida, round, pausa, intervalo e desempate:

```text
Seleção de modo → contagem inicial → round e relógio → apuração
                                            ↓
                 resultado ← desempate ← intervalo / próximo round
```

O `TimeManager_PL` escolhe o formato da partida e coordena o relógio, RPCs e transições. `RoundManager` guarda configuração e vitórias. `PointsManager_PL` filtra pontos em pausa e intervalo. `ShowControls_PL` monta placar e relógio no HUD. `PLShowWinner` apresenta estatísticas e decisões de desempate. Os scripts ficam embutidos na cena do Mod Editor; a descrição aqui é arquitetural, sem redistribuir seu conteúdo.

Os formatos são Unranked (3 × 3 minutos), Ranked (3 × 5 minutos) e Title Fight (5 × 5 minutos), com intervalos de 30 segundos. Em empate persistente, o fluxo prevê round extra de 1 minuto.

## 4. Dificuldades técnicas e depuração

**Relógio e estado.** O projeto tem o tempo do round e o tempo da sala como conceitos distintos. A investigação conferiu quando cada um é iniciado, pausado, sincronizado e encerrado. Na implementação observada, o tempo restante é enviado aos clientes e a UI usa esse valor para o relógio do HUD.

**Pausa e pontuação.** O filtro de pontos consulta o estado `MatchPaused` e `IsIntermission`. Isso liga a regra de pontuação ao estado de partida, em vez de depender só de uma indicação visual.

**Empate.** A análise do mod original identificou ausência de desempate explícito em um resultado final empatado. A edição inclui estado de decisão, apresentação ao anfitrião e round extra. A contribuição de Nixagorn nesse fluxo é creditada.

**Estatísticas no intervalo.** O experimento FullStats preservou os textos originais de resultado e capturou nomes, pontos, vitórias e bônus antes do reset dos jogadores. O anfitrião enviava esses dados aos clientes por RPC. A visibilidade do painel era controlada durante o intervalo, com limpeza antes do round seguinte. Essa solução foi ampliada para Title Fight em um experimento posterior AllModes; o registro dessa etapa não bastava sozinho para afirmar validação funcional do modo de cinco rounds.

**Interface.** Ajustes de posicionamento e compatibilidade do texto do relógio foram feitos dentro das restrições de Unity UI e do ambiente de scripts do jogo. A verificação por código foi complementada por inspeção no editor e testes funcionais.

**Compilação em dois ambientes.** Um registro local anterior documenta compilação externa com `csc.exe` e DLLs locais, sem erro, além de `compiled OK` no log do Battlehub para cinco scripts. `PointsManager_PL` estava embutido e compilava externamente, mas não tinha confirmação individual no log. O registro anotou avisos de compilação e mensagens `Type mismatch!` sem erro CS. Esse nível de detalhe impediu tratar um sinal parcial como aprovação completa do mod.

**DynamicCSharp e componentes embutidos.** A cena usa `CustomScript` do Battlehub. Uma etapa de depuração evitou referências diretas ao namespace `DW2.` e símbolos de UI incompatíveis naquele ambiente, como `TextAnchor`, `FontStyle.Bold` e `HorizontalWrapMode`. Scripts longos traziam risco de truncamento na colagem do editor; os experimentos compararam o texto C# embutido na cena com os arquivos legíveis da cópia de trabalho. Um log posterior do experimento AllModes registrou seis scripts como `compiled OK` e `GameStarted()`, mas não identificava nominalmente a cena selecionada. Por isso o teste visual e funcional continuou necessário.

## 5. Versionamento, validação e publicação

Foram usadas cópias de trabalho e backups antes de ajustes de cena e scripts. O material local registra comparação entre textos dos scripts e os CustomScripts embutidos na cena, além de verificação de arquivos de metadados do Battlehub. O processo separou testes de sintaxe, compilação no editor, interação em jogo e publicação na Workshop.

O responsável pelo projeto informou que a última versão publicada funcionou corretamente nos testes realizados. O item público é [Pro League Rounds — Official Fullstats Edition](https://steamcommunity.com/sharedfiles/filedetails/?id=3821094797). Como não há neste repositório uma captura de logs e critérios da sessão final, o resultado é descrito como relato do responsável, sem percentuais de cobertura, número de usuários ou outras métricas. O [roteiro de demonstração](DEMO_SCRIPT.md) especifica quais funcionalidades ainda precisam ser captadas em gameplay real para este repositório.

### Evidências e o que elas permitem afirmar

| Fonte | Observação sustentada | Limite |
| --- | --- | --- |
| Scripts e documentação locais da cópia de trabalho | Estrutura entre gerenciadores, relógio, pontuação, UI e desempate. | Leitura de código não equivale a partida testada. |
| Registros de compilação externa e log Battlehub de etapas históricas | Ausência de erros C# em compilações específicas; mensagens `compiled OK` para componentes identificados. | Não são prova isolada da versão final, da UI ou da rede. |
| Relato do responsável pelo projeto | A última versão publicada funcionou corretamente nos testes realizados. | Não fornece matriz, logs ou números de desempenho neste repositório. |
| [Item público na Steam Workshop](https://steamcommunity.com/sharedfiles/filedetails/?id=3821094797) | Título, publicação, funcionalidades descritas e créditos públicos. | Descrição da Workshop não substitui inspeção técnica independente. |

## 6. Aprendizados

1. Diferenciar um relato de bug, uma hipótese de causa e uma falha reproduzida evita mudanças baseadas em suposição.
2. Quando UI, pontuação e relógio dependem do mesmo estado, as transições precisam ser examinadas como um fluxo único.
3. Compilar externamente é útil, mas o ambiente do jogo e o teste funcional continuam necessários.
4. Backups e comparações de versão ajudam a localizar regressões e preservam uma rota de recuperação.
5. Créditos claros são parte da documentação técnica quando um projeto deriva do trabalho de outra pessoa.

## Créditos e limites

Projeto original, correção original de pausa e melhorias de desempate: **Nixagorn**, conforme os créditos da Workshop. Edição FullStats: integração, relógio personalizado no HUD, estatísticas, ajustes visuais, depuração e publicação documentados neste portfólio. O desenvolvimento contou com **assistência do Codex**; trechos gerados ou adaptados com IA não são apresentados como código escrito integralmente à mão. Este estudo não inclui código, assets, cenas ou arquivos do jogo e não atribui licença a material de terceiros.
