# Dashboard Analista de Extratos Bancários

Ferramenta web para analisar extratos bancários: importa os arquivos, ajuda a categorizar os lançamentos, consolida vários bancos e gera dashboard e relatório executivo — **tudo no navegador, sem servidor e sem enviar nenhum dado para a internet**.

Feito para uso de consultoria financeira (Somar+ Consultores).

---

## Índice

- [Privacidade](#privacidade)
- [Principais recursos](#principais-recursos)
- [Formatos de extrato suportados](#formatos-de-extrato-suportados)
- [Como rodar](#como-rodar)
- [Scripts disponíveis](#scripts-disponíveis)
- [Fluxo de uso](#fluxo-de-uso)
- [Perfil de categorização (`perfil.json`)](#perfil-de-categorização-perfiljson)
- [Stack técnica](#stack-técnica)
- [Estrutura do projeto](#estrutura-do-projeto)
- [Testes](#testes)
- [Documentação de especificação (SDD)](#documentação-de-especificação-sdd)
- [Limitações conhecidas](#limitações-conhecidas)
- [Licença](#licença)

---

## Privacidade

Esta é a característica central do projeto: **os dados bancários nunca saem do seu dispositivo.**

- Aplicação 100% client-side: sem backend, sem banco de dados, sem telemetria.
- O `index.html` aplica uma Content Security Policy restritiva, com `connect-src 'none'` — o próprio navegador bloqueia qualquer tentativa de requisição de rede.
- Todo o processamento (leitura dos arquivos, categorização, cálculos, gráficos e exportações) acontece em memória, no navegador.
- Nada é gravado automaticamente: o que persiste é apenas o que você baixa explicitamente (`perfil.json`, `extrato.xlsx`).

---

## Principais recursos

### Ingestão e consolidação
- Importação de múltiplos extratos de bancos diferentes de uma vez.
- Consolidação com remoção de duplicatas exatas e contagem do que foi removido.
- Conciliação automática de transferências entre contas (débito em um banco casado com crédito equivalente em outro), para não contar o mesmo dinheiro duas vezes.
- Aviso quando os extratos importados parecem ser de correntistas diferentes.
- Extração de banco, agência e conta corrente do arquivo (quando disponível no OFX).

### Revisão e categorização
- Três abas: **Pendentes**, **Categorizados** e **Categorias**.
- Gravação só acontece ao clicar em **Gravar** — escolher no dropdown não grava nada sozinho.
- Categorização **em massa**: selecione vários lançamentos e aplique uma categoria de uma vez.
- Seleção mista (receitas + despesas juntas) é bloqueada, evitando categorizar um crédito como despesa por engano.
- O dropdown mostra **somente receitas** para valores positivos e **somente despesas** para negativos.
- Memória de categorização: a escolha é lembrada por descrição normalizada e reaplicada em importações futuras.
- Detecção automática por texto para aplicações e resgates de investimento, com revisão manual sempre disponível.
- Cadastro de categorias editável (rótulo e tipo). A troca de tipo é bloqueada quando a categoria já está em uso, com atalho para reclassificar os lançamentos afetados.
- Atalho para ver o dashboard parcial mesmo com pendências.

### Dashboard
- KPIs: Receitas, Despesas, **Saldo sem investimentos**, Taxa de poupança e % Categorizado.
- KPIs de investimento: **Investimentos**, **Rendimentos**, **Resgates de investimento** e **Resultado de Investimentos** (= Investimentos + Rendimentos − Resgates). Esses valores são excluídos de Receitas/Despesas para não distorcer a movimentação operacional.
- **Fluxo de caixa** em cascata (waterfall), agrupando numa única barra "Outros" as despesas menores cuja soma fique dentro de 10% do total.
- **Comentários do consultor financeiro** gerados por regras determinísticas sobre os agregados (sem IA).
- Receitas e despesas por categoria (gráficos de rosca) e comparativo entre períodos.
- Lançamentos sinalizados como não recorrentes, com sensibilidade ajustável.
- **Drill-down:** clicar numa categoria de um gráfico filtra o extrato abaixo ("Extrato Filtrado"), com botão **Limpar Filtros** para voltar ao "Extrato Completo".

### Extrato detalhado
- Colunas: Dia, Banco (nome + agência + conta), Descrição, Categoria, Valor, **Soma Filtro** e **Saldo Extrato**.
  - **Saldo Extrato** = saldo real acumulado (saldo inicial + todos os lançamentos do escopo), independente dos filtros da tabela.
  - **Soma Filtro** = acumulado apenas das linhas visíveis após os filtros, sem somar o saldo inicial.
- Primeira linha fixa com o **saldo inicial** (zero quando o extrato não o identifica).
- Busca livre que procura em qualquer coluna (data, banco, descrição, categoria) e também por valor.
- Filtros por tipo (débito/crédito), intervalo de datas, banco e categoria.
- Edição de categoria direto na tabela pelo ícone de lápis, individualmente ou em massa.
- Exportação para XLSX respeitando os filtros aplicados na tela.

### Relatório executivo
- Versão para impressão/PDF com seções em ordem fixa: cabeçalho, fluxo de caixa, categorias, extrato, comentários do consultor, aviso de divergência (quando houver) e comentários do usuário.

### Padrão brasileiro
- Valores em BRL e datas em `dd/mm/aaaa` em todas as telas.

---

## Formatos de extrato suportados

| Formato | Suporte | Observação |
|---|---|---|
| **OFX / QFX** | Recomendado | Melhor assertividade: traz valores com sinal, período, saldo e (quando presente) agência/conta |
| XLS / XLSX | Suportado | Detecção automática de colunas; pode exigir revisão manual |
| PDF | Suportado | Requer camada de texto — PDFs escaneados (só imagem) não são suportados, pois não há OCR |

---

## Como rodar

**Pré-requisitos:** Node.js 18+ (exigência do Vite 5) e npm.

```bash
npm install
```

```bash
npm run dev
```

O app sobe em `http://localhost:5173`.

Para gerar a versão de produção:

```bash
npm run build
```

O resultado vai para `dist/` e pode ser servido por qualquer hospedagem estática (ou aberto localmente), já que não existe backend.

---

## Scripts disponíveis

| Script | O que faz |
|---|---|
| `npm run dev` | Sobe o servidor de desenvolvimento (Vite) |
| `npm run build` | Checa tipos e gera o build de produção em `dist/` |
| `npm run preview` | Serve localmente o build de produção |
| `npm test` | Roda a suíte de testes uma vez (Vitest) |
| `npm run test:watch` | Roda os testes em modo watch |
| `npm run typecheck` | Checagem de tipos com `tsc --noEmit` |

---

## Fluxo de uso

1. **Ingestão** — selecione um ou mais extratos (e, opcionalmente, importe um `perfil.json` com suas categorizações anteriores).
2. **Revisão** — categorize as pendências (individualmente ou em massa), ajuste itens já categorizados e mantenha o cadastro de categorias.
3. **Dashboard** — analise KPIs, fluxo de caixa, categorias e o extrato detalhado; clique nos gráficos para investigar a composição de cada valor.
4. **Relatório** — gere o relatório executivo para impressão ou PDF.

A qualquer momento você pode baixar o extrato em XLSX e o `perfil.json` para reutilizar depois.

---

## Perfil de categorização (`perfil.json`)

O `perfil.json` é o arquivo que guarda o seu conhecimento de categorização:

- a lista de categorias (as sementes e as que você criou);
- o mapa de memória `descrição normalizada → categoria`;
- as regras de detecção por trecho de texto.

Ele é gerado pelo botão **Baixar perfil.json** e pode ser reimportado na tela de Ingestão. Como esse arquivo contém descrições reais dos seus lançamentos, **trate-o como dado sensível** — evite versioná-lo ou compartilhá-lo.

---

## Stack técnica

- **React 18** + **TypeScript** + **Vite 5**
- **Zustand** para estado da aplicação
- **ECharts** (import modular, renderer SVG) para os gráficos
- **SheetJS (xlsx)** para leitura de planilhas e exportação
- **pdf.js (pdfjs-dist)** para extração de texto de PDFs
- **Vitest** + **Testing Library** + **jsdom** para os testes
- Parser OFX próprio, tolerante a OFX 1.x (SGML) e 2.x (XML)
- CSS puro com design tokens (sem framework de UI, sem biblioteca de ícones e sem fontes externas — exigência do CSP)

---

## Estrutura do projeto

```
src/
  app/            # shell da aplicação e store (Zustand)
  core/           # núcleo sem dependência de UI
    analysis/     # KPIs, fluxo de caixa, categorias, transferências, não recorrentes, investimentos
    categorize/   # motor de categorização, categorias semente e regras
    consolidate/  # consolidação multi-banco e dedupe
    export/       # exportação XLSX
    format/       # formatação BRL, datas e rótulo de banco
    ingest/       # orquestração da importação
    normalize/    # modelo canônico e normalização de descrições
    parsers/      # OFX, planilha e PDF
    profile/      # leitura/escrita e validação do perfil.json
    report/       # modelo do relatório e comentários do consultor
  ui/
    charts/       # componentes de gráfico (ECharts)
    components/   # componentes compartilhados (extrato, seleção, KPIs, ícones)
    hooks/        # hooks reutilizáveis
    report/       # relatório executivo (com estilos de impressão)
    screens/      # Ingestão, Revisão e Dashboard
    styles/       # tokens de tema e responsividade
sdd/              # documentação de especificação (ver abaixo)
```

O núcleo (`src/core`) é desacoplado da interface: os cálculos e parsers não importam React nem ECharts, o que mantém a lógica testável isoladamente.

---

## Testes

```bash
npm test
```

A suíte cobre o núcleo (parsers, consolidação, categorização, análises, exportação) e as telas/componentes de interface com Testing Library.

---

## Documentação de especificação (SDD)

A pasta `sdd/` guarda os artefatos do fluxo de especificação usado no projeto, úteis para entender **por que** cada decisão foi tomada:

| Arquivo | Conteúdo |
|---|---|
| `BRAINSTORM_*.md` | Exploração inicial, alternativas consideradas e cortes de escopo |
| `DEFINE_*.md` | Requisitos, critérios de aceite (EARS), escopo e gate de verificação |
| `DESIGN_*.md` | Arquitetura, decisões técnicas (ADRs), manifesto de arquivos e padrões |
| `BUILD_REPORT_*.md` | Relatório de implementação com evidências de verificação |
| `HANDOFF_*.md` | Estado do trabalho e próximos passos |

---

## Limitações conhecidas

- **PDF sem camada de texto** (escaneado) não é suportado — não há OCR; nesses casos, use o extrato em OFX.
- **Agência e conta** só são extraídas quando o arquivo as fornece (típico de OFX). Sem essas informações, a coluna "Banco" exibe o nome do arquivo/planilha.
- A **detecção automática de investimentos** usa regras por trecho de texto e pode gerar falsos positivos (por exemplo, uma remuneração de aplicação automática classificada como aplicação). A correção manual está sempre disponível na Revisão ou no Extrato.
- **Nada é persistido automaticamente**: ao recarregar a página, a sessão recomeça. Baixe o `perfil.json` para não perder as categorizações.

---

## Licença

Projeto privado (`"private": true`), sem licença pública definida. Todos os direitos reservados.
