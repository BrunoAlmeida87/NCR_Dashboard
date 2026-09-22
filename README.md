# NCR Control

Sistema **local, standalone e offline** para controle, acompanhamento, análise e visualização
de NCRs (Não Conformidades).

Toda a aplicação está contida em **um único arquivo**:

```
NCR_Control.html
```

HTML, CSS, JavaScript, regras de negócio, leitor de Excel (.xlsx), motor de gráficos,
banco de dados, histórico e interface — tudo embutido. Sem CDN, sem servidor, sem Node,
sem Python, sem banco SQL e sem instalação. Basta copiar o arquivo e abrir no navegador.

## Como usar

1. Copie `NCR_Control.html` para uma pasta (ex.: `C:\NCR_CONTROL\`) e abra com **Chrome** ou **Edge**.
2. Clique em **Selecionar Pasta de Trabalho** e autorize o acesso.
3. Coloque as exportações do Excel em `imports/` (ou na própria pasta).
4. Clique em **Atualizar Banco de Dados**.

O sistema cria e mantém:

```
/Sistema_NCR
├── NCR_Control.html
├── BD/
│   ├── ncr.json                   estado atual de cada NCR
│   ├── produtos.json              vínculos NCR × produto
│   ├── historico.json             eventos de alteração (avanços, retornos, laterais…)
│   ├── arquivos_processados.json  hash SHA-256 de cada Excel já lido
│   ├── importacoes.json           log de cada atualização
│   ├── configuracao.json          fluxo, aging, preferências
│   └── backups/                   backups automáticos com retenção
└── imports/                       arquivos .xlsx de entrada
```

Em navegadores sem File System Access API, o sistema funciona em modo alternativo,
guardando o banco no armazenamento interno do navegador e importando os Excel manualmente.

## Características

- **Identificação por conteúdo**: cada Excel é reconhecido por **hash SHA-256**, nunca pelo nome.
  Arquivo idêntico é ignorado; mesmo nome com conteúdo novo é reprocessado.
- **Atualização incremental**: insere NCRs novas, atualiza as existentes campo a campo e
  preserva todo o histórico anterior.
- **Histórico reconstruído já na primeira leitura**: exports de NCR normalmente trazem, na
  própria linha, as datas em que a NCR avançou, foi rejeitada, aprovada ou encerrada em cada
  etapa (`Data avanço 2.1 - …`, `Data da rejeição 7.2 - TA`, `CEDOC Closure Data`, …). Essas
  colunas são reconhecidas pelo **padrão do cabeçalho** — nunca por lista fixa nem pelo nome
  do arquivo — e a trajetória completa entra no banco na primeira importação, com a data e o
  responsável registrados na planilha. Daí em diante só o que muda entre duas exportações
  vira evento novo.
  - Etapas citadas apenas nessas colunas e ausentes da coluna *Status* (ex.: `7.2 - TA`) têm a
    ordem **inferida pelo código numérico** do rótulo e aparecem com borda tracejada no fluxograma.
  - Quando o export data apenas a **saída** de uma etapa, a chegada é registrada na mesma data e
    marcada como **estimada**; esse intervalo sem informação fica fora das médias por etapa.
  - A lista de colunas reconhecidas fica visível em **Configurações → Fluxo e status**, onde a
    reconstrução também pode ser desligada.
- **Data da "foto" deduzida do conteúdo**: quando o export traz o tempo de tramitação, a data da
  exportação é calculada a partir de *data de criação + dias de tramitação* (mediana das linhas),
  em vez da data do arquivo — que muda ao copiar ou baixar novamente. O tempo no status atual vem
  da coluna de *tempo mais recente*, quando existe.
- **Classificação automática das movimentações**: avanço, retorno, movimentação lateral,
  fechamento, reabertura e permanência (que não gera evento).
- **Fluxo gráfico de cada NCR, em duas leituras** (aba *Fluxo gráfico* no detalhe):
  - **Trajetória** (padrão): um passo por linha, na ordem em que aconteceram, com data,
    tempo de permanência naquela passagem e contador de reincidência. Como toda seta liga
    apenas linhas vizinhas, nenhuma cruza outra — a ordem é a própria leitura de cima para baixo.
  - **Mapa do fluxo**: todas as etapas, inclusive as não percorridas. Cada seta recebe uma
    faixa exclusiva (um vão entre colunas ou uma faixa sob a grade) e nunca passa por cima de
    uma caixa; os números de passo se afastam sozinhos quando cairiam um sobre o outro.
- **Recortes (dimensões) com multisseleção**: colunas como `SBR`, `SBY` e `Bigramas` viram
  filtros próprios, presentes em todas as telas de análise.
  - A coluna de cada recorte é achada pelo **cabeçalho**; quando o export usa outro nome, ela é
    apontada à mão em **Configurações → Dimensões**. Recorte sem coluna correspondente
    simplesmente não aparece — e volta sozinho quando um Excel com essa coluna for importado.
  - `SBR` é **normalizado**: `SBR 2`, `sbr-02` e `2` caem no mesmo grupo; `SBR1`–`SBR4` ficam
    sempre na lista, mesmo zerados, e `SBRall` aparece quando o export marca alguma NCR como
    comum a todos. Os valores originais continuam visíveis, nunca são substituídos.
  - `Bigramas` é **multivalorado**: uma célula com `ES, FP, ER, DP` entra nos quatro sistemas,
    e cada bigrama vira uma opção de filtro e um grupo próprio.
- **Aviso discreto de filtro**: quando há recorte ativo, o topo mostra quantos filtros estão
  aplicados e quantas NCRs restam — e, nas telas que não usam o recorte (Importações, Arquivos,
  Configurações), avisa que elas seguem mostrando os dados completos. Um clique lista e limpa.
- **Fluxos das NCRs**: tela dedicada às trajetórias, em duas leituras.
  - **Galeria**: uma miniatura por NCR, com o caminho percorrido no tempo (etapa no eixo
    vertical, passos no horizontal), colorida por tipo de movimento. Agrupa por sistema, SBR,
    etapa ou status.
  - **Reprodução**: o relógio corre sobre as datas reais do export e cada NCR é um ponto que
    anda entre as etapas, com contagem por etapa e destaque para quem se mexeu nos últimos
    15 dias.
- **Módulos**: Dashboard, NCRs, Produtos, Aging, Análise de Fluxo, Fluxos das NCRs, Histórico,
  Importações, Arquivos e Configurações.
- **Gravação segura**: backup automático antes de cada importação, escrita em duas etapas
  com verificação de leitura e recuperação automática a partir do backup se um JSON estiver corrompido.
- **Configurável**: fluxo de status e ordens, faixas de aging, campos monitorados e retenção
  de backups podem ser alterados sem destruir o histórico existente.
