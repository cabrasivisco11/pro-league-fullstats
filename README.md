# Pro League Rounds — Official Fullstats Edition

Edição publicada para **Drunken Wrestlers 2** com relógio próprio no HUD, estatísticas completas entre rounds e integração de regras, pontuação, pausas e desempate. O projeto mostra trabalho com C#, Unity UI, depuração de scripts em um ambiente restrito e validação antes da publicação.

**[Ver o mod na Steam Workshop](https://steamcommunity.com/sharedfiles/filedetails/?id=3821094797)** · **[Ler o estudo de caso](docs/CASE_STUDY.md)** · **[Tecnologias e arquitetura](docs/TECH_STACK.md)**

> **Autoria:** Pro League Rounds foi criado originalmente por **Nixagorn**. A Workshop também credita a ele a correção original de pausa e melhorias de desempate. A edição FullStats publicada por **PREGUIÇA** integra relógio personalizado, estatísticas, ajustes visuais e outras adaptações. O desenvolvimento contou com assistência do **Codex** para análise, geração/adaptação de código, depuração e documentação. Não reivindico autoria integral do mod nem implementação manual integral dos scripts.

## Visão rápida

| Área | O que a edição apresenta |
| --- | --- |
| Modos | Unranked: 3 rounds de 3 min; Ranked: 3 rounds de 5 min; Title Fight: 5 rounds de 5 min. |
| Tempo | Cronômetro individual MM:SS integrado ao HUD e intervalo de 30 s entre rounds. |
| Placar | Pontos, vitórias de rounds e bônus nas telas de estatísticas. |
| Transições | Pausa do relógio, bloqueio de pontuação durante pausas e intervalos, decisão de empate e round extra de 1 min quando necessário. |
| Publicação | Item público na Steam Workshop, com créditos e descrição das funcionalidades. |

## O problema técnico

O jogo mantém um tempo nativo de sala, enquanto o mod precisa controlar o tempo de cada round. A interface, a pontuação e o resultado também dependem do estado da partida. Durante a investigação, foi necessário distinguir relatos de falha, hipóteses de causa, erros de compilação e comportamento observado em jogo.

Um exemplo concreto surgiu no painel FullStats: os textos completos de resultado desapareciam antes do fim do intervalo. A análise local identificou que a transição desabilitava esses textos e que uma rotina posterior os limpava. A solução experimental preservou os textos originais, capturou pontos e bônus antes do reset dos jogadores e sincronizou os dados entre anfitrião e cliente. O [estudo de caso](docs/CASE_STUDY.md) descreve a sequência e seus limites de validação.

## Como foi desenvolvido

```text
Modo escolhido → relógio e round → apuração → intervalo → próximo round ou resultado
                         └──────── pausa / pontuação / sincronização ────────┘
```

Os componentes C# da cena cooperam para configurar a partida, sincronizar relógio e dados por RPC, bloquear pontos em estados inválidos e apresentar resultados em Unity UI. A verificação combinou leitura de scripts, comparação com os `CustomScript` embutidos na cena, compilação externa, logs do Battlehub/Mod Editor e testes funcionais. Compilação externa e mensagens `compiled OK` **não são tratadas como prova isolada de gameplay ou rede**.

## Demonstração e evidência

O [roteiro de vídeo de aproximadamente 60 segundos](docs/DEMO_SCRIPT.md) está pronto para capturar **gameplay real** dos três modos, relógio, estatísticas, intervalo, pausa, desempate e resultado. **Ainda não há vídeo, GIF ou screenshot neste repositório.** As pastas em [`media/pro-league/`](media/pro-league/README.md) receberão somente material próprio ou autorizado, com créditos registrados.

A última versão publicada foi relatada como funcional nos testes pelo responsável pelo projeto. Este repositório não contém os logs ou a matriz dessa sessão final; por isso não apresenta números de usuários, downloads, cobertura de testes ou desempenho como resultados técnicos.

## Documentação

- [Estudo de caso técnico](docs/CASE_STUDY.md): objetivos, problemas, soluções, depuração, validação e aprendizados.
- [Tecnologias e arquitetura](docs/TECH_STACK.md): C#, Unity UI, DynamicCSharp/Battlehub, RPCs e processo de compilação.
- [Roteiro de demonstração](docs/DEMO_SCRIPT.md): cenas e instruções para gravação real.
- [Créditos e permissões de mídia](media/pro-league/credits.md): registro obrigatório antes de adicionar imagens ou vídeo.

## Direitos

Este repositório contém **documentação**, não o mod. Não distribui scripts, cenas `.rtsc`, DLLs, executáveis, assets proprietários do jogo ou arquivos originais de Nixagorn. Nenhuma licença de código ou mídia de terceiros é concedida aqui. Nomes de jogos, ferramentas e franquias pertencem a seus titulares.

## English summary

**Pro League Rounds — Official Fullstats Edition** is a published *Drunken Wrestlers 2* mod edition featuring a custom HUD timer, full round statistics, three match formats, pause and scoring integration, and tie-break handling. This repository documents the technical challenges, solutions, debugging and validation process. **Nixagorn** created the original mod and is credited for the original pause fix and tie-break improvements. The FullStats edition was published by **PREGUIÇA** with **Codex assistance**; no claim of sole or entirely manual authorship is made. [Read the case study](docs/CASE_STUDY.md).
