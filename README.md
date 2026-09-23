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
│   ├── historico_sistema.json     histórico capturado da aba Histórico do NCR Integrada
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
- **Histórico exato do NCR Integrada** (tela *Importações → Histórico do sistema*): a aba
  *Histórico* de cada NCR no sistema traz cada mudança de estado com data, hora e usuário.
  1. **Copiar extrator** e colar no Console (F12) de uma página do NCR Integrada, já logado.
  2. No painel que aparece, colar a lista de links (`…/#/ncr/<id>/editar`) ou usar
     *Pegar links desta página*; **Iniciar**. O extrator abre cada NCR num quadro invisível
     do próprio site (mesma sessão, só leitura), lê o histórico e guarda o progresso no
     navegador — dá para pausar e continuar. Ao fim, **Baixar JSON** (há também CSV).
  3. **Importar histórico extraído (.json)** no NCR Control.
  4. Para atualizar depois (no mesmo navegador, que guarda o que já foi capturado):
     - **NCRs novas**: lista completa do sistema com *Capturar: Só as novas*;
     - **abertas e reabertas**: depois de importar o Excel mais recente, *Copiar links a
       recapturar* no NCR Control (abertas + status diferente entre Excel e captura, que é
       como uma reabertura aparece) e rodar com *Capturar: Todas da lista*.
     Importar o novo JSON: a captura mais recente de cada NCR prevalece.

  Na importação, a trajetória de cada NCR até a data da captura é **substituída** pelas
  transições reais do sistema (as datas estimadas e as mudanças vistas entre duas fotos
  saem); o que os Excel registrarem depois continua valendo. Visitas, contadores, "status
  desde" e última movimentação são recalculados, e a correção é reaplicada a cada nova
  importação — NCRs que só entrarem no banco depois já chegam corrigidas. Backup antes.
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
  - `SBR` é **normalizado**: `53 d`, `53-D` e `53D` caem no mesmo grupo; `53A`–`53D` ficam
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
  - **Reprodução**: o relógio corre sobre as datas reais (de 1 dia/s a 1 ano/s) e cada NCR é
    um ponto que anda entre as colunas, com contagem por coluna e destaque para quem se mexeu
    nos últimos 15 dias. As colunas podem ser as etapas do fluxo (agrupadas) ou cada etapa
    da NCR (status); por padrão entram todas as NCRs do recorte, ou as N primeiras por uma
    prioridade escolhida.
- **Módulos**: Dashboard, NCRs, Produtos, Aging, Análise de Fluxo, Fluxos das NCRs, Histórico,
  Importações, Arquivos e Configurações.
- **Gravação segura**: backup automático antes de cada importação, escrita em duas etapas
  com verificação de leitura e recuperação automática a partir do backup se um JSON estiver corrompido.
- **Configurável**: fluxo de status e ordens, faixas de aging, campos monitorados e retenção
  de backups podem ser alterados sem destruir o histórico existente.
