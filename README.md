# Análise Diária de Técnicos

Dashboard 100% front-end (HTML + Tailwind CDN + JavaScript puro) que lê a planilha do dia no navegador. Nenhum dado é enviado a servidor.

## Como usar

1. Abra o `index.html` no navegador (ou rode `python3 -m http.server 8000` na pasta e acesse `http://localhost:8000`).
2. Entre com seu usuário e senha na tela de login (veja **Login** abaixo).
3. Clique em **Escolher arquivo** ou arraste um `.xlsx`, `.xls` ou `.csv` para a área tracejada.
4. Use os filtros e os cards para navegar; exporte em `.xlsx`.

## Login

As duas páginas exigem login (`login.html`). Só os usuários `adminTne`, `adminTmg` e `admin047` entram; a senha é combinada com a equipe e não fica escrita no projeto. O login vale para a aba do navegador; o botão de saída fica ao lado do botão de tema.

O login também protege o cadastro dos técnicos: ele é publicado só criptografado (`app/cadastro.enc.js`, AES-256-GCM) e cada usuário abre a chave com a própria senha, no navegador. O cadastro sem criptografia fica em `privado/cadastro.js`, que **não pode ser publicado** (está no `.gitignore`).

Para mudar o cadastro, os usuários (lista `USUARIOS`) ou a senha, edite `privado/cadastro.js` ou o script e gere de novo:

```
pip install cryptography
python ferramentas/criptografar_cadastro.py
```

O script pede a senha no terminal e regrava `app/cadastro.enc.js`.

## Report Backlog

O botão **Report Backlog** (abaixo de "Processamento local no navegador") abre o `backlog.html` já com a planilha importada na Análise, na aba `Page 1`. O link **Análise de técnicos** faz o caminho de volta. A planilha fica guardada na sessão da aba do navegador (`app/shared-file.js`, sessionStorage): vale enquanto a aba estiver aberta e some ao clicar em **Limpar** em qualquer uma das páginas.

Abaixo do **Volume encerrado por bucket** fica o **Aging das TSKs em backlog**: o tempo desde a Criação do NTT (coluna AS) até agora, nas faixas <1d, 1d a 3d, 3d a 5d, 5d a 10d e >10d, por ANF, com filtro Todas / Fila técnica / Centralizado. A ANF sai do código do CM (coluna AK; sem CM, do bucket): 31 BHE, 32 JFA, 33 GVS, 34 URA/ULA, 35 VGA/VGS, 37 DVL, 38 MCL (a tela mostra só o número); o resto (CM de fora de MG ou sem código) fica em Outros. Embaixo, a lista de todo o backlog (com a prioridade P1 a P5 da coluna GI), da mais antiga para a mais recente; o botão **Acima de 1 dia**, ao lado de Exportar aging, deixa só as TSKs paradas há mais de 1 dia. A lista é filtrável pelo seletor **Prioridade** (P1 a P5 ou sem prioridade) e por ANF (no seletor ou clicando na ANF do quadro; Ctrl/Cmd + clique junta mais de uma, e clicar de novo na ANF volta a mostrar todas); **Exportar aging** baixa o quadro e a lista exibida em Excel.

**Copiar parcial** termina com a linha **SITES FORA COM FLAG 1**, consultada no Genesis na hora do clique (veja abaixo); se o Genesis não responder, a linha fica fora do parcial e o painel avisa.

## Sites fora com Flag 1 (Genesis)

O número vem do Painel Online do Genesis: NE IDs (coluna Site) distintos com **Flag 1**, o mesmo número que o Genesis mostra com o filtro "Flag 1" marcado. No Genesis, a TMG fica dentro da regional **TLE** (só sites de MG). O Genesis não deixa outras páginas lerem os dados dele, então quem consulta é o **TrocaIP.exe** (projeto Troca-de-IPs), o mesmo programa da troca de IP:

1. Conecte na VPN.
2. Abra o `TrocaIP.exe` (ícone TLP perto do relógio). Ele responde em `http://127.0.0.1:8765/genesis/ne-fora?regional=TMG`.
3. Abra o `backlog.html` (pelo site ou com duplo clique) e use **Copiar parcial**.
4. No site publicado, o navegador pede na primeira vez para o site acessar **dispositivos na rede local**: clique em **Permitir**. Sem essa permissão a linha fica fora do parcial. Se tiver bloqueado, libere no ícone à esquerda do endereço → **Permissões do site** e recarregue a página.

O endereço e a porta ficam em `GENESIS_NE_DOWN_URL`, no início do script do `backlog.html`.

## Extração do TOA (Page 1)

Basta subir a extração bruta do TOA (aba `Page 1`, uma linha por atividade). Quando a aba tem as colunas `Provedor`, `Tipo da Atividade` e `Estado`, o sistema monta sozinho a mesma contagem da tabela dinâmica:

- **Linhas**: `Provedor`, só os técnicos (`TLP-T…-NOME`). CMs, centrais, `COMMAND_CENTER`, `TMG_FMMT` etc. ficam de fora.
- **Colunas**: `Estado` (Cancelada, Concluída, Iniciado, Não iniciado, pendente).
- **Tipo de técnico** (filtro na tela): classificado pelas TSKs em fila (Iniciado, Não iniciado, pendente). Tem corretiva em fila → **Corretivas**; só preventiva em fila → **Preventivas**; sem nada em fila, vale o tipo das atividades do dia. Na visão Corretivas só contam as atividades não preventivas (igual à tabela dinâmica), na Preventivas só `Atividade Preventiva` (`TOA_PREVENTIVA_TYPES` no `app/app.js`) e em **Todos** (padrão) contam todas.
- **Cadastro**: CM, responsável, horário, região e gerente vêm do cadastro de técnicos, pelo código (`TLP-T0000000`). Técnicos da extração que não estão no cadastro não entram na análise e aparecem listados no aviso após a importação.

### Cadastro de técnicos

O cadastro padrão é aberto no login (`app/cadastro.enc.js`, gerado a partir de `privado/cadastro.js`; TMG: Planilha3 do report; TNE1/TNE2: aba Cadastro). Para trocar, use **Atualizar cadastro** na tela com uma planilha que tenha `Nome do recurso` (ou `Técnicos`) e, de preferência, `Tempo de trabalho`, `Unidade organizacional`, `Coordenador`/`Responsável`, `Regional` e `Gerente`. O cadastro enviado fica salvo no navegador e tem prioridade; quem não estiver nele continua vindo do cadastro padrão. **Restaurar padrão** descarta o cadastro enviado.

## Abas e colunas esperadas

**Aba de atividades**: `Planilha2` por padrão (se não existir, a primeira aba). Se o arquivo tiver mais de uma aba, um seletor permite escolher outra.

| Campo | Cabeçalhos aceitos |
|---|---|
| Técnico* | Técnicos, Tecnicos, Técnico, Rótulos de Linha |
| Concluída* | Concluída, Concluida |
| Pendente* | Pendente |
| Cancelada | Cancelada |
| Iniciado | Iniciado |
| Não iniciado | Não iniciado, Nao iniciado |
| Total Geral | Total Geral |
| Horário | Horário, Horario, Tempo de trabalho |
| Região | Região, Regiao, Regional |
| CM | Unidade organizacional, CM |
| Responsável | Responsável, Coordenador, Responsável Coordenador |
| Gerente | Gerente, Gerente Responsável |

\* obrigatórias. Os cabeçalhos são comparados por igualdade exata, ignorando acento, caixa e espaços. Não há fallback por posição de coluna.

Região, responsável e gerente vêm da própria aba de atividades. Quem não tiver região fica como "Região não identificada" e aparece na validação.

O técnico no formato `XXX-NNNN-Nome` é separado em código (`XXX-NNNN`) e nome.

## Situações

- **Com conclusão**: concluída > 0
- **Sem conclusão + com pendência**: concluída = 0 e pendente > 0
- **Sem conclusão + sem pendência** (crítico): concluída = 0 e pendente = 0
- **Com atividade iniciada**: pelo menos uma atividade com status Iniciado, de qualquer tipo (card, filtro Situação e tabela de técnicos). Com a extração do TOA, a visão vira uma lista enxuta (Técnico, TSK/ATSK iniciada, NE ID, Região, CM, Responsável): a TSK vem de `Número de Ordem` e o NE ID de `NE ID` (ou de `END_ID` quando o NE ID vem vazio, comum em preventivas e atividades conjuntas); passar o mouse na TSK mostra tipo, cidade/UF e ETA. A busca por técnico também acha pelo número da TSK, e o Exportar planilha leva as colunas TSK iniciada e NE ID.

"Sem conclusão" sempre inclui os dois subgrupos.

## Configuração

No topo do `app/app.js`: `ACTIVITY_SHEET_NAME`, `REGION_ORDER` (vazio = ordem em que as regiões aparecem na aba) e os aliases de cabeçalho.
# Report-Fila-Tec-TMG
