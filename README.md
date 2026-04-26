# Mapeamento de Melhorias e Falhas — SAJ Defensorias

Artefato HTML interativo que consolida as solicitações de melhorias e falhas do sistema **SAJ Defensorias** (Defensoria Pública do Estado de Mato Grosso do Sul) reportadas por defensores e colaboradores ao Gestor SAJ. O sistema é desenvolvido e mantido pela **Softplan**, e este projeto serve como ferramenta de organização, transparência e priorização das demandas a serem encaminhadas à empresa.

## Contexto

A DPE/MS migrou do **SAJ Tribunais** (usado pelo Poder Judiciário) para o **SAJ Defensorias** (versão específica para a Defensoria), ambos desenvolvidos pela Softplan. A transição revelou um conjunto de funcionalidades ausentes ou que funcionam de forma diferente, além de falhas que comprometem prazos processuais. Defensores e colaboradores reportam essas questões por email ao Gestor SAJ, que as consolida nesta ferramenta.

## O que faz

A aplicação roda inteiramente no navegador (single-file HTML, sem backend) e oferece cinco modos de visualização:

- **Priorização GT** (aba padrão): cada membro do Grupo de Trabalho classifica as melhorias pendentes em Crítica / Alta / Média / Baixa (limite de 3 itens em cada) ou Backlog, com ordem manual via drag-and-drop. Exporta um JSON com os votos.
- **Lista detalhada**: filtros por categoria, módulo, solicitante, integração, origem, status e ordenação.
- **Mais solicitadas**: ranking das demandas por número de solicitantes únicos.
- **Por colega**: distribuição de solicitações por colaborador.
- **Por módulo**: distribuição de solicitações por módulo do sistema.

Ao clicar em qualquer item, abre-se um modal com a descrição completa e os emails (ou registros do GT) que originaram a solicitação.

### Critérios de classificação

- **Falha**: o sistema não entrega o prometido ou cria risco processual.
- **Melhoria**: evolução ou funcionalidade nova.
- **Configuração**: ajuste de parametrização (não exige desenvolvimento).
- **Integração**: depende da Softplan Tribunais / TJMS, além da Softplan Defensorias.
- **Grupo de Trabalho**: item levantado em reunião do GT.
- **Resolvido**: correção implementada.

A contagem de "solicitantes" considera pessoas únicas (mesmo que tenham reportado em emails diferentes).

## Como usar

### Para os defensores (votação)

1. Acesse o link da aplicação.
2. Na aba **Priorização GT**, classifique cada melhoria pendente:
   - **Crítica, Alta, Média, Baixa**: máximo 3 itens em cada (força priorização consciente).
   - **Backlog**: itens pertinentes mas que não cabem no top 12 e não serão priorizados no momento.
   - **Descartar**: itens que não devem ser priorizados.
3. Use os botões dos cards ou arraste e solte entre classificações. Também é possível reordenar dentro de uma classificação por drag-and-drop — a posição (#1, #2, ...) é registrada.
4. Identifique-se no campo "Seu nome" e clique em **Exportar votos (.json)**.
5. Envie o arquivo `.json` gerado no grupo de WhatsApp do GT.
6. (Opcional) Use **Imprimir / Salvar PDF** para guardar registro pessoal da sua classificação.

Os votos ficam salvos no `localStorage` do navegador — fechar a aba e voltar depois preserva o estado.

### Para o gestor (mantenedor do projeto)

#### Atualizando os dados

Todos os dados ficam em um único arquivo (`index.html`), em duas constantes JavaScript:

- **`emails`**: dicionário com os emails (e registros equivalentes) que originaram as solicitações. Chaves seguem o padrão `e01`, `e02`, ..., além de `eGS` (inclusões do Gestor SAJ a partir de análise interna) e `eGT` (itens levantados em reunião do GT).
- **`dados.solicitacoes`**: array com as solicitações. Cada item tem:
  - `id` — numérico, único e sequencial.
  - `titulo`, `descricao` — textos visíveis.
  - `categoria` — `"Falha"`, `"Melhoria"` ou `"Configuração"`.
  - `modulo` — string livre (mas reutilize valores existentes para agrupar corretamente).
  - `integracao` — `true` se depende da Softplan Tribunais / TJMS.
  - `resolvido` — `true` após a Softplan implementar; combinar com `notaResolucao` (descrição da resolução).
  - `solicitantes` — array de `{nome, data: "YYYY-MM-DD", emailId}`, apontando para chaves do dicionário `emails`.

Para adicionar uma nova solicitação:

1. Adicione o email correspondente em `emails` (se vier de email novo).
2. Adicione o objeto em `dados.solicitacoes` com o próximo `id` sequencial.
3. Atualize a data em `<p class="disclaimer">` no `<body>` ("Conteúdo atualizado até DD/MM/AAAA").
4. Bump da constante `ARTEFATO_VERSAO` para a data atual (`'YYYY-MM-DD'`).
5. Commit e push — o deploy é automático via GitHub Pages.

#### Agregando os votos do GT

Após receber os JSONs dos defensores no WhatsApp, salve-os numa pasta. O esquema do JSON exportado:

```json
{
  "tipo": "votos-priorizacao-saj-defensorias-gt",
  "versaoArtefato": "2026-04-26",
  "votante": "Nome do Defensor",
  "dataExportacao": "2026-04-26T18:30:00.000Z",
  "totalMelhorias": 28,
  "totalClassificadas": 28,
  "votos": [
    {
      "id": 3,
      "titulo": "Assinatura e peticionamento em lote",
      "prioridade": "P0",
      "posicao": 1
    }
  ]
}
```

Valores possíveis de `prioridade`: `"P0"` (Crítica), `"P1"` (Alta), `"P2"` (Média), `"P3"` (Baixa), `"backlog"`, `"descartada"`, ou `null` (não classificada).

`posicao`: ordem dentro da classificação (1 = primeira). É `null` em "não classificadas" e "descartadas".

Para inspecionar um voto individual, basta usar o botão **Importar votos (.json)** na própria aplicação — ele carrega o arquivo no navegador e mostra como aquele defensor votou. Para análise consolidada com múltiplos votantes, um script Python ou planilha que percorra os JSONs resolve (somar votos por id, ponderar por prioridade etc.).

#### Hospedagem

A aplicação é um único arquivo HTML auto-contido (sem backend, sem build, sem dependências externas). Hospede gratuitamente em:

- **GitHub Pages**: Settings → Pages → Source: branch `main`, pasta `/ (root)`. URL ficará em `https://<usuario>.github.io/<repo>/`.
- **Cloudflare Pages**, **Netlify** ou **Vercel**: conectar o repositório, deploy automático a cada push.

Como não há build step, qualquer commit no `index.html` atualiza imediatamente a versão pública.

## Estrutura do projeto

```
.
├── index.html   # Aplicação completa (HTML + CSS + JavaScript inline)
└── README.md    # Este arquivo
```

## Stack técnica

- HTML5, CSS3, JavaScript ES2020 — vanilla, sem framework, sem dependências externas.
- HTML5 Drag and Drop API para a tela de priorização.
- `localStorage` para persistência dos votos no navegador.
- Funciona offline após o primeiro carregamento.
- Compatível com navegadores modernos (Chrome, Firefox, Edge, Safari).

## Limitações conhecidas

- A votação é descentralizada: cada defensor vota localmente e exporta um JSON. Não há agregação automática (consciente, para evitar dependência de backend). A consolidação fica com o gestor.
- Os votos exportados em uma versão mais antiga do artefato podem ter títulos diferentes dos itens atuais; a importação tenta correlacionar por `id` + `titulo`, com fallback só por `titulo`. Itens não correspondidos são descartados com aviso.
- O `localStorage` é específico por navegador/dispositivo. Se o defensor abrir em outro computador, começa do zero (mas pode importar o próprio JSON).

## Licença

A definir pelo projeto institucional.

## Contato

Pedro de Luna Souza Leite — Gestor SAJ Defensorias / DPE-MS
`gestorsaj@defensoria.ms.def.br`
