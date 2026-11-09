## Sumário
 
1. [Objetivo pedagógico](#objetivo-pedagógico)
2. [Como executar](#como-executar)
3. [Modos de jogo](#modos-de-jogo)
4. [Pontuação](#pontuação)
5. [O disco](#o-disco)
6. [Ligação com o xv6](#ligação-com-o-xv6)
7. [Design](#design)
8. [Heurísticas de Nielsen](#heurísticas-de-nielsen)
9. [Estrutura do código](#estrutura-do-código)
10. [Acessibilidade e compatibilidade](#acessibilidade-e-compatibilidade)
11. [Limitações e ideias futuras](#limitações-e-ideias-futuras)
---
 
## Objetivo pedagógico
 
O público-alvo são colegas que já estudaram o capítulo 8 do xv6 (sistema de arquivos) e querem **ver o algoritmo acontecendo**. O jogo parte de um disco compartilhado de 32 blocos e usa dois modos complementares:
 
| Modo | Conceito treinado | Pergunta central |
|---|---|---|
| **A: Alocação** | Onde colocar os dados | Quantos blocos e quantos acessos cada estratégia custa? |
| **B: Journaling** | O que fazer quando a escrita é interrompida | Como o disco permanece consistente após uma queda? |
 
Ao final, a tela de resultado compara os dois discos e explica por que o xv6 usa **alocação indexada + log ao mesmo tempo**.
 
## Como executar
 
Não há instalação, build ou dependências.
 
1. Baixe `disco-sob-ameaca.html`.
2. Abra o arquivo em qualquer navegador moderno (duplo clique ou *Arquivo → Abrir*).
3. Jogue. Em celular, abra o link da demo ou envie o arquivo para o aparelho.
**Única dependência externa (opcional):** a fonte *Archivo Black*, carregada do Google Fonts para os títulos. Sem internet, o jogo funciona normalmente com uma fonte de reserva (Impact / Arial Black).
 
## Modos de jogo
 
### Menu principal
 
Quatro cartões em formato de pasta: **Modo A**, **Modo B**, **Modo Livre** e **Resultado comparativo**. O botão **Como jogar** (cabeçalho) abre as regras, e o botão **⊞ Iniciar** da barra de tarefas abre um menu rápido com os mesmos destinos.
 
### Modo A: Alocação de blocos
 
1. Escolha a estratégia: **Contígua**, **Encadeada** ou **Indexada**.
2. Uma fila de arquivos é alocada bloco a bloco (botão **Alocar próximo bloco** ou **Automático/Pausar**):
   | Arquivo | Tamanho |
   |---|---|
   | `relatorio.txt` | 3 blocos |
   | `foto.jpg` | 5 blocos |
   | `log.txt` | 2 blocos |
   | `video.mp4` | 4 blocos |
3. Cada passo mostra uma frase explicativa, por exemplo:
   - Contígua: *“precisa de 3 blocos consecutivos → encontrados em 12–14”*
   - Encadeada: *“bloco 12 → ponteiro para 20”*
   - Indexada: *“bloco de índices em 8 aponta para 12, 20, 25”*
4. O disco já começa **pré-fragmentado** (blocos do “sistema”). Na alocação contígua, `foto.jpg` **não cabe** em nenhum espaço livre: é preciso **Desfragmentar**, o que ilustra a fragmentação externa.
5. **Desafio final:** estimar quantos acessos ao disco são necessários para ler o **último bloco** de `foto.jpg` (acesso aleatório).
   | Estratégia | Acessos ao último bloco (arquivo de *n* blocos) | Blocos gastos |
   |---|---|---|
   | Contígua | 1 | *n* |
   | Encadeada | *n* (segue os ponteiros desde o início) | *n* |
   | Indexada | 2 (índice + dado) | *n* + 1 |
### Modo B: Journaling (“Sobreviva à queda”)
 
São **3 operações** consecutivas. Cada uma consiste em *escrever 2 blocos no arquivo A e atualizar o inode* (3 suboperações).
 
1. **Registrar no log:** cada clique grava uma suboperação num bloco de log (28–31) e atualiza o contador `log: x/y`.
2. **Commit:** só habilita quando todas as suboperações estão no log.
3. **⚡ Simular queda agora!** está sempre disponível:
   - **Antes do commit:** o recovery ignora o log. Disco consistente, operação perdida.
   - **Depois do commit:** o recovery lê o cabeçalho e reaplica as suboperações, uma a uma (animado). Disco consistente, operação preservada.
4. **Concluir (instalar):** finaliza a operação sem queda (equivale ao `end_op`).
5. **+ escrita extra:** adiciona suboperações até **encher o log** (capacidade de 4 blocos). O jogo bloqueia e explica que o xv6 força o commit (`begin_op`/`end_op`).
6. **Usar log (desmarcar):** repete a operação **sem log**. Uma queda no meio deixa blocos **vermelhos** (corrompidos) e o jogo explica o que quebrou.
### Modo Livre / Sandbox
 
Laboratório sem roteiro: escolha a estratégia, ligue ou desligue o log, aloque arquivos de 1 a 10 blocos, desfragmente e simule quedas a qualquer momento. Ideal para demonstração ao vivo.
 
### Tela de resultado
 
- Discos finais do Modo A e do Modo B lado a lado.
- Tabela comparativa das três estratégias.
- Três perguntas de fechamento, com resposta destacada ligando os dois modos ao xv6.
- Botões **Jogar de novo**, **Voltar ao menu** e **Ver justificativa técnica** (modal com o texto e o mapa de heurísticas).
## Pontuação
 
**Modo A**
 
| Evento | Pontos |
|---|---|
| Arquivo alocado | +10 |
| Bônus por não ter precisado desfragmentar | +10 por arquivo |
| Acertar o palpite de acessos | +50 |
 
**Modo B**
 
| Evento | Pontos |
|---|---|
| Queda antes do commit (consistente) | +10 |
| Queda depois do commit (recovery reaplica) | +25 |
| Operação instalada com log | +20 |
| Operação concluída sem log e sem queda | +10 |
| Queda sem log, nada escrito ainda (sorte) | +5 |
| Queda sem log com escrita pela metade (corrompido) | 0 |
 
## O disco
 
Grade de **32 blocos (8×4)**, compartilhada entre os modos. Os blocos **28–31** são reservados ao log.
 
| Estado | Cor | Significado |
|---|---|---|
| Livre | cinza claro | Bloco disponível |
| Dados | verde | Bloco de dados de arquivo |
| Índice | roxo | Bloco de índices (ponteiros) ou inode |
| Log | amarelo | Bloco reservado ao log (cheio = em uso) |
| Corrompido | vermelho | Inconsistente após queda sem log |
| Em foco | contorno preto + pulso | Bloco do passo atual |
 
A letra dentro do bloco é a inicial do dono (arquivo). Uma **faixa de explicação** abaixo do disco (balão 💡) descreve cada passo.
 
## Ligação com o xv6
 
O painel **Saiba mais**, sempre visível ao lado do jogo, relaciona cada modo ao código do xv6:
 
- **Alocação indexada:** o inode guarda ponteiros diretos e indiretos; `bmap()` em `fs.c` traduz o bloco lógico para o físico.
- **Log:** em `kernel/log.c`, `write_log()` copia para o log, `commit()` grava o cabeçalho e `recover_from_log()` reaplica no boot. `begin_op`/`end_op` delimitam a transação, e `bio.c` (buffer cache) faz a ponte com o disco.
## Design
 
O visual combina duas referências:
 
- **Neo-brutalismo:** fundo amarelo com formas rosa e roxa, bordas pretas grossas, sombras duras deslocadas, cartões em formato de pasta e tipografia pesada.
- **Windows XP / Windows 7:** janela com barra de título azul e botões de janela, barra de tarefas com botão verde **Iniciar** e relógio, botões com brilho vítreo, balão de dica e uma **tela azul (BSOD)** de `POWER_LOSS_DURING_WRITE` a cada queda simulada.
**Animações:** entrada de telas com “salto”, blocos que brotam em sequência, *pop* com onda ao alocar, brilho nos blocos de log, pulso nos corrompidos, tremor do disco na queda, pontuação que salta e confete ao acertar ou concluir. Quem tem **“reduzir movimento”** ativado no sistema recebe a versão sem animações.
 
## Heurísticas de Nielsen
 
| Heurística | Onde aparece no jogo |
|---|---|
| #1 Visibilidade do estado | Disco colorido, contador `log: x/y`, faixa de explicação, pontuação |
| #2 Correspondência com o mundo real | Blocos, arquivos, queda de energia, commit; metáforas de janela e barra de tarefas |
| #3 Controle e liberdade | Pausar, Reiniciar, Menu, botão ✕ da janela, queda a qualquer momento |
| #4 Consistência e padrões | Mesmo disco, cores e botões em todos os modos |
| #5 Prevenção de erros | Commit só habilita com log completo; confirmação antes de reiniciar |
| #6 Reconhecer em vez de lembrar | Legenda fixa e painel “Saiba mais” sempre visíveis |
| #7 Flexibilidade e eficiência | Modo guiado, modo automático e Modo Livre |
| #8 Design estético e minimalista | Uma tela, um disco, uma faixa de explicação |
| #9 Diagnóstico de erros | Blocos vermelhos e frase explicando o que corrompeu |
| #10 Ajuda e documentação | “Como jogar” e modal de justificativa técnica |
 
## Estrutura do código
 
Tudo está em `disco-sob-ameaca.html`, organizado em três partes: `<style>` (variáveis CSS, componentes e animações), marcação HTML (menu, jogo, resultado, modal, barra de tarefas) e `<script>`.
 
**Principais funções JavaScript**
 
| Área | Funções |
|---|---|
| Disco e interface | `newDisk`, `drawDisk`, `render`, `say`, `show`, `modal`, `ask`, `petals` |
| Alocação (compartilhada) | `plan`, `stepsFor`, `apply`, `defrag`, `cost` |
| Modo A | `startA`, `pickA`, `panelA`, `nxtA`, `autoA`, `defragA`, `guessA` |
| Modo B | `startB`, `newOp`, `panelB`, `regB`, `extraB`, `commitB`, `instB`, `wrB`, `crashB`, `finishB` |
| Modo Livre | `startS`, `panelS`, `allocS`, `defragS`, `crashS` |
| Resultado | `showResult`, `just` |
 
**Estruturas de dados**
 
```js
// Disco: array de 32 blocos
D[i] = { id, estado: "livre|dados|indice|log|corrompido", dono, usado };
 
// Plano de alocação de um arquivo
plan(estrategia, n) → { blocks: [...], idx? } | null   // null = não cabe
 
// Transação (Modo B)
B = { subs: [{t:"dado|inode", b:blocoAlvo}], lg, cm, ap, useLog, over, ... };
```
 
**Como personalizar:** os tamanhos e nomes dos arquivos estão em `startA()` (`queue`); os blocos pré-ocupados, em `newDisk()`; o tamanho do disco e o início do log, nas constantes `N` e `LOG0`; as cores, nas variáveis CSS (`--dat`, `--idx`, `--log`, `--bad`…).
 
## Acessibilidade e compatibilidade
 
- Layout fluido: grade de 8 colunas que se adapta a telas pequenas; painéis lado a lado no desktop e empilhados no celular.
- Botões com altura mínima de 44 px para toque.
- Faixa de explicação com `aria-live="polite"` para leitores de tela.
- Respeita `prefers-reduced-motion` e a área segura (*safe area*) de celulares com entalhe.
- Testado conceitualmente para navegadores modernos (Chrome, Edge, Firefox e Safari recentes). Requer JavaScript habilitado.
## Limitações e ideias futuras
 
- O disco é uma **simulação didática**: não modela bitmap de blocos, inodes completos nem ponteiros indiretos do xv6.
- Os acessos a disco do desafio consideram apenas o caso de leitura do último bloco.
- Ideias: bloco de bitmap visível, ponteiros indiretos na indexada, mais rodadas no Modo B, ranking local, efeitos sonoros e exportação do relatório final.
---
 
Projeto acadêmico sobre sistemas de arquivos (xv6, capítulo 8). Defina aqui a licença e os créditos do grupo.
 
