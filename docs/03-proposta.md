# 3. Proposta — Sistema de Gestão de Suprimentos e Despesas

## 3.1 A tese

> **O SIENGE continua sendo o sistema de registro. O sistema novo é a camada de processo.**

Essa é a única decisão de arquitetura que importa de verdade, e é onde projetos assim costumam
morrer. A tentação é construir "um sistema melhor que o SIENGE". Não é isso.

- O **SIENGE** é o *system of record*: cadastros, pedidos, contratos, medições, notas, contas a
  pagar, contabilidade. É ele que a auditoria e o fisco olham.
- O **sistema novo** é o *system of engagement*: onde as pessoas trabalham. Cotação, aprovações,
  documentos, prazos, portal do fornecedor, indicadores. Ele orquestra e, no fim de cada etapa,
  **escreve o resultado no SIENGE**.

Regra prática para todo requisito novo: *se o dado precisa existir na contabilidade, ele nasce aqui
e termina no SIENGE; se o dado é sobre como se chegou até a decisão, ele vive só aqui.*

## 3.2 Escopo

### Dentro do escopo

**Núcleo (fundação)**
- Autenticação e perfis, incluindo acesso para **terceiros** (gerenciadora, incorporadora) e para
  **fornecedores** (portal externo)
- Espelho de cadastros do SIENGE (SPEs, obras, centros de custo, departamentos, plano financeiro,
  insumos, fornecedores)
- **Matriz de alçadas única**, parametrizável por departamento, faixa de valor e nível
- **Motor de aprovação** genérico: serial, paralelo, em cadeia ordenada (A1 de obra), com
  delegação, reprovação com motivo obrigatório e regras de segregação de funções
- **Cofre de documentos** com versionamento, validade e vínculo ao registro do SIENGE
- **Relógio de prazos**: SLA por etapa, contagem regressiva, alerta e escalonamento
- **Auditoria imutável**: quem fez, o quê, quando, com qual valor antes e depois

**Processo de despesas sem pedido (Processo B completo)**
- Formulário por tipo de documento autorizado, com anexos obrigatórios por tipo
- Validação de prazo (10 dias), duplicidade e campos
- Aprovação D1 pela matriz
- Acionamento de suprimentos quando o credor não existe
- Criação do título no SIENGE via integração, com anexos
- Conferência do título e liberação

**Processo de suprimentos (Processo A completo)**
- Solicitação de compra/serviço com alerta orçamentário
- **Cadastro de fornecedor com portal**: ficha eletrônica, upload de documentos, validação de MEI,
  controle de validade, fluxo de alteração bancária com nova assinatura
- **Quadro de concorrência nativo**: itens, propostas, equalização, rodadas de negociação,
  comparação com orçamento, os três alertas de justificativa obrigatória, escolha e A1 em cadeia
- Pedido de compra e contrato: espelhamento, anexação do QC, A2/A3, controle de aditivos
- Medição: boletim, atesto, **checklist de encargos** (folha, DCTFWeb, FGTS Digital), A4, saldo por item
- Recepção e validação fiscal da nota: autenticidade, retenções, prazo, recusa
- Geração do título e conferência, com A5 quando fora do prazo
- **Controle de retenção contratual de 5%** por contrato, com saldo e condição de liberação

**Gestão**
- Painel de pendências por pessoa ("o que espera por mim")
- Indicadores de processo (§3.7)

### Fora do escopo (explicitamente)
- Contabilidade, apuração de tributos, folha
- Programação e execução do pagamento (permanece no SIENGE/banco)
- Orçamento de obra (permanece no SIENGE; consumido como referência)
- Recebimento físico de material em canteiro
- Substituição de qualquer módulo do SIENGE

## 3.3 Arquitetura

### Princípios
1. **Um repositório, uma linguagem.** Com uma pessoa mantendo o sistema, a economia está em não
   trocar de contexto. TypeScript da UI ao banco.
2. **A integração com o SIENGE é assíncrona e reprocessável.** Nunca um `POST` síncrono no meio do
   clique do usuário. Fila, retry com backoff, fila de erro visível, reprocessamento manual.
3. **Toda escrita no domínio é um evento.** Auditoria não é um `log`; é a forma como o estado muda.
4. **Contingência é requisito, não exceção.** O procedimento já prevê "integração indisponível com
   vencimento próximo: o financeiro lança manualmente e registra a contingência". O sistema precisa
   saber marcar isso.

### Stack recomendada

| Camada | Escolha | Por quê |
|--------|---------|---------|
| UI + API | **Next.js (App Router) + TypeScript** | Um repo, um deploy, server actions cobrem 90% do CRUD |
| Banco | **PostgreSQL** | Transações sérias, JSONB para formulários variáveis, full-text |
| ORM/migrações | **Prisma** ou **Drizzle** | Migração versionada é inegociável num sistema com auditoria |
| Autenticação interna | **Microsoft Entra ID (SSO)** | A CASA8 já é M365; evita mais um usuário/senha |
| Acesso externo | Convite por e-mail + link assinado, escopo restrito | Gerenciadora, incorporadora e fornecedores não terão conta M365 |
| Arquivos | **S3-compatível** (Azure Blob / R2 / MinIO) | Nunca no banco; URLs assinadas de curta duração |
| Fila/jobs | **pg-boss** (usa o próprio Postgres) | Uma dependência a menos que Redis, suficiente nesse volume |
| Assinatura eletrônica | Integração com provedor (Clicksign/D4Sign/DocuSign) | A ficha cadastral e o contrato exigem assinatura |
| Observabilidade | Logs estruturados + Sentry + healthcheck | Você vai sustentar isso sozinho |
| Deploy | Container (Azure Container Apps, Fly.io ou VPS com Docker) | Backup automatizado do Postgres é requisito |

> **Sobre low-code.** Vale considerar honestamente. Um Retool/Appsmith sobre Postgres entrega o CRUD
> mais rápido. Mas o QC, o motor de alçadas em cadeia e o portal do fornecedor são exatamente as
> partes que low-code faz mal. A recomendação é código próprio com stack mainstream — a mitigação do
> risco de "um dev só" é a escolha de tecnologia comum, não a ferramenta.

### Desenho da integração com o SIENGE

```
  Sistema CASA8                      Fila (pg-boss)                    SIENGE
 ┌──────────────┐                  ┌───────────────┐              ┌─────────────┐
 │ Aprovação D1 │ ──emite evento──►│ criar_titulo  │──HTTP/API───►│ Contas      │
 │ concluída    │                  │ (retry 5x,    │              │ a pagar     │
 └──────────────┘                  │  backoff exp.)│◄──resposta───│             │
                                   └───────┬───────┘              └─────────────┘
                                           │ falha após retries
                                           ▼
                                 ┌─────────────────────┐
                                 │ Fila de erros       │  visível ao financeiro,
                                 │ (causa + payload)   │  reprocessável, ou
                                 └─────────────────────┘  marcada como contingência
```

**Sincronização de cadastros:** leitura periódica (job agendado) para espelhar SPEs, obras, centros
de custo, plano financeiro, insumos e fornecedores. O sistema **nunca** é dono desses dados — tem
cópia de leitura com carimbo de sincronização, para não travar quando o SIENGE estiver fora.

**Risco número 1 do projeto:** não está confirmado o que a API do SIENGE realmente expõe. Ver
`docs/05-integracao-sienge.md` — há um *spike* técnico de 1 semana proposto **antes** de qualquer
compromisso de prazo.

## 3.4 Plano de implantação em ondas

Cada onda termina em algo em produção, usado por gente de verdade. Nada de "fase de análise".

### Onda 0 · Descoberta e prova técnica — 2 a 3 semanas
- Responder os pontos em aberto (`docs/07-perguntas-abertas.md`)
- **Spike da API do SIENGE**: provar criação de título avulso com anexo, leitura de cadastros,
  e verificar medição/nota. Sem isso, nenhuma estimativa vale.
- Levantar volumes reais (compras/mês, contratos ativos, medições/mês, notas/mês)
- Fechar a matriz de alçadas vigente de cada departamento
- **Saída:** escopo confirmado, riscos técnicos resolvidos ou contornados, estimativa firme

### Onda 1 · Fundação + Despesas sem pedido — 8 a 12 semanas
Fundação: SSO, perfis, espelho de cadastros, matriz de alçadas, motor de aprovação, cofre de
documentos, relógio de prazos, auditoria.
Processo B completo, ponta a ponta, com integração de título.

**Por que começar aqui:** é o menor pedaço que atravessa toda a arquitetura, o processo ainda não
tem ferramenta (não há nada para desalojar), e o volume (~40/mês) é seguro para aprender. Substitui
a decisão A-vs-B que está em aberto.
**Valor entregue:** fim da redigitação de ~40 títulos/mês; rastreabilidade de D1; controle do prazo
de 10 dias.

### Onda 2 · Fornecedor + Cotação — 10 a 14 semanas
Portal do fornecedor (ficha eletrônica, documentos com validade, assinatura, bloqueio de MEI, fluxo
de alteração bancária). QC nativo com equalização, negociação, alertas e A1 em cadeia ordenada.

**Valor entregue:** aposenta a planilha e o Word; trilha de auditoria da escolha do fornecedor;
começa a acumular base de preços; fecha o vetor de fraude bancária (F3).

### Onda 3 · Pedido, Contrato, Medição — 10 a 14 semanas
Solicitação com alerta orçamentário; pedido e contrato com A2/A3 e controle de aditivos; medição com
boletim/atesto, checklist de encargos, A4 e saldo por item; controle de retenção de 5%.

**Valor entregue:** aprovações unificadas num lugar só; risco previdenciário sob controle
documentado; visibilidade da retenção contratual (F7).

### Onda 4 · Nota fiscal, Título, Indicadores — 8 a 12 semanas
Recepção de nota pelo portal, validação fiscal com conferência de retenções, prazos e recusa;
geração do título com retenções; conferência e A5; painel de indicadores.

**Valor entregue:** SLA fiscal de 2 dias úteis medido; A5 vira exceção real em vez de rotina; o
processo passa a ser gerenciável por número.

### Resumo

| Onda | Duração | Acumulado |
|------|---------|-----------|
| 0 · Descoberta e spike | 2–3 sem. | ~1 mês |
| 1 · Fundação + Despesas | 8–12 sem. | ~3–4 meses |
| 2 · Fornecedor + Cotação | 10–14 sem. | ~6–7 meses |
| 3 · Pedido/Contrato/Medição | 10–14 sem. | ~8–10 meses |
| 4 · NF/Título/Indicadores | 8–12 sem. | ~11–13 meses |

Faixas para **um desenvolvedor em tempo integral**, com um interlocutor da controladoria disponível
para decisões. São estimativas de ordem de grandeza, a serem revistas ao fim da Onda 0 — que é
exatamente para isso que a Onda 0 existe.

## 3.5 Como a proposta se compara aos cenários A e B

| Critério | A · Microsoft 365 | B · BPM contratado | **C · Sistema próprio** |
|---|---|---|---|
| Despesas sem pedido | Resolve | Resolve | Resolve |
| Cotação / QC | Não cobre | Customização paga | **Nativo, com base de preços** |
| Portal do fornecedor | Não cobre | Customização paga | **Nativo** |
| Encargos por medição | Não cobre | Customização paga | **Nativo** |
| Retenção contratual 5% | Não cobre | Customização paga | **Nativo** |
| Custo inicial | Baixo | Médio-alto | Alto (tempo de desenvolvimento) |
| Custo recorrente | Incluído no M365 | Assinatura por usuário + sustentação | Infra (baixa) + o desenvolvedor |
| Time-to-value | Semanas | Meses | ~3 meses para o 1º processo |
| Dependência de fornecedor | Microsoft | **Alta** | Nenhuma |
| Risco principal | Teto funcional baixo | Preso ao fornecedor e ao contrato | **Bus factor de 1 pessoa** |

**Recomendação honesta:** se a empresa só quer resolver as despesas sem pedido no próximo trimestre
e não pretende mexer em suprimentos, o cenário A é a resposta certa e este projeto não se justifica.
O cenário C se justifica se — e somente se — o objetivo for centralizar o processo inteiro. Vale a
pena colocar essa frase na proposta: ela mostra que você entendeu o negócio, não só a tecnologia.

## 3.6 Riscos e mitigações

| # | Risco | Impacto | Mitigação |
|---|-------|---------|-----------|
| R1 | A API do SIENGE não expõe criação de título avulso, anexos ou medição | **Crítico** — muda o desenho | Spike na Onda 0. Plano B: sistema aprova e gera arquivo/roteiro de lançamento, mantendo digitação no SIENGE mas com conferência automática |
| R2 | **Bus factor de 1 desenvolvedor** | Alto | Stack mainstream, código no GitHub da empresa, testes nos fluxos de aprovação, documentação viva, ADRs. Contratar o 2º dev na Onda 3 |
| R3 | Terceiros (gerenciadora, incorporadora) não adotam | Alto — A1 e A4 travam | Envolver na Onda 0; UX de aprovação por e-mail com um clique; nunca exigir instalação |
| R4 | Fornecedores não usam o portal | Médio | Portal sem cadastro de senha (link por e-mail); manter recepção por e-mail com ingestão assistida na transição |
| R5 | Escopo aberto (10 pontos pendentes só nas despesas) | Médio | Onda 0 fecha os pontos; contrato/plano por onda, não por projeto inteiro |
| R6 | Divergência entre o espelho de cadastros e o SIENGE | Médio | Sincronização com carimbo de data, tela de divergências, SIENGE sempre vence |
| R7 | Resistência a "mais um sistema" | Médio | O sistema tem que ser **mais rápido** que o Excel no primeiro dia, não só mais controlado. Medir o tempo de tarefa antes e depois |
| R8 | Dados pessoais e bancários (LGPD) | Médio | Criptografia em repouso dos dados bancários, controle de acesso por papel, retenção definida, trilha de acesso |

## 3.7 Indicadores que o sistema deve produzir

Isso encerra a fratura F9 e é o que sustenta o projeto politicamente depois da primeira onda.

**Suprimentos**
- Lead time da solicitação até o pedido/contrato, por obra e por comprador
- % de cotações com 3+ propostas · % com justificativa de exceção
- % de escolhas diferentes do menor valor final, por motivo
- Aderência ao orçamento: diferença (R$ e %) entre valor escolhido e orçado, por obra e por etapa
- Economia de negociação: total equalizado → valor final

**Aprovações**
- Tempo médio e p90 por código de aprovação (A1 a A5, D1) e por aprovador
- Fila atual: aprovações pendentes por idade
- Taxa de reprovação e motivos mais frequentes

**Fiscal e financeiro**
- % de notas validadas dentro dos 2 dias úteis
- % de notas recusadas, por motivo
- % de títulos que dependeram de **A5** (pagamento fora do prazo) — meta: tendência a zero
- % de despesas sem pedido enviadas com menos de 10 dias

**Risco**
- Contratos com encargos pendentes ou vencidos
- Fornecedores com documento vencido
- Saldo de retenção contratual por contrato e por obra
- Falhas de integração abertas e títulos lançados em contingência
