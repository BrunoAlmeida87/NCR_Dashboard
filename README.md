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
- **Histórico construído a partir da primeira leitura**: a primeira aparição de uma NCR é
  registrada como *Estado inicial conhecido*; daí em diante só o que muda vira evento.
- **Classificação automática das movimentações**: avanço, retorno, movimentação lateral,
  fechamento, reabertura e permanência (que não gera evento).
- **Módulos**: Dashboard, NCRs, Produtos, Aging, Análise de Fluxo, Histórico, Importações,
  Arquivos e Configurações.
- **Gravação segura**: backup automático antes de cada importação, escrita em duas etapas
  com verificação de leitura e recuperação automática a partir do backup se um JSON estiver corrompido.
- **Configurável**: fluxo de status e ordens, faixas de aging, campos monitorados e retenção
  de backups podem ser alterados sem destruir o histórico existente.
