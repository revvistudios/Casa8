# Proposta Comercial · Centralização e automação do processo de suprimentos e despesas

| | |
|---|---|
| **Cliente** | Grupo CASA8 |
| **Proponente** | Revvi Studios |
| **Data** | Setembro de 2026 |
| **Validade da proposta** | 30 dias |

> Um sistema que transforma os **28 controles-chave** já escritos pela controladoria
> em verificações que o processo não consegue pular — sem substituir o SIENGE.
>
> Começa por um **mapeamento pago e independente**, que entrega valor por si só e
> define escopo, prazo e investimento das etapas seguintes com números firmes.

---

## 1. Entendimento da necessidade

A CASA8 documentou seu processo com um nível de detalhe raro. O **Procedimento de
Suprimentos** (v1.0) percorre a compra e a contratação da solicitação ao título, com
cinco pontos de aprovação. O **Procedimento de Custos e Despesas sem Pedido** (v0.1)
cobre tributos, ITBI, taxas, contas de consumo e reembolso, e apresenta dois cenários
de ferramenta para a empresa decidir.

O problema não é o processo — é que ele está distribuído entre SIENGE, planilha,
formulário Word, e-mail e atenção humana. Em particular:

- O **quadro de concorrência** é uma planilha, e a aprovação da escolha do fornecedor
  — que na obra percorre três empresas em cadeia ordenada — é registrada por
  assinatura no arquivo ou e-mail anexado.
- O **cadastro de fornecedor** é um formulário Word que vai e volta por e-mail, com
  até seis documentos anexos e nenhum controle de validade das certidões.
- A **matriz de alçadas é uma só**, mas seria parametrizada em dois sistemas — no
  SIENGE para pedidos e contratos, e na ferramenta nova para despesas.
- Os **seis prazos** do processo (dia 25, 15 dias, 2 dias úteis, 10 dias, encargos por
  medição e as datas de pagamento) não têm contagem regressiva nem alerta.
- A **retenção contratual de 5%** é retida em toda nota de obra e "registrada para
  liberação futura" — mas a liberação está explicitamente fora de todos os procedimentos.
- Nas despesas sem pedido, cerca de **40 títulos por mês** são redigitados no contas a
  pagar, com uma segunda pessoa conferindo a digitação.

Nenhum procedimento define indicador. Hoje não se sabe o prazo médio de uma cotação, o
percentual de compras com três propostas, a aderência ao orçamento por obra nem
quantas notas dependeram de aprovação de atraso.

---

## 2. O que propomos

Um sistema web próprio, integrado ao SIENGE, que centraliza o processo e faz valer os
controles automaticamente.

### Princípio que rege todo o projeto

> **O SIENGE continua sendo o sistema de registro. O sistema novo é a camada de processo.**

O SIENGE permanece dono dos cadastros, pedidos, contratos, medições, notas e contas a
pagar — é o que a auditoria e o fisco olham. O sistema novo é onde as pessoas
trabalham: cotação, aprovações, documentos, prazos, portal do fornecedor e
indicadores. Ao fim de cada etapa, **ele escreve o resultado no SIENGE**.

Não se constrói um ERP paralelo. Isso mantém o projeto viável, a contabilidade íntegra
e a CASA8 livre para trocar de fornecedor de software sem refazer o processo.

### O que o sistema passa a fazer

- **Matriz de alçadas única**, parametrizável por departamento e faixa de valor,
  servindo todas as aprovações do grupo.
- **Motor de aprovação** com cadeia ordenada, delegação por ausência, reprovação com
  motivo obrigatório e segregação de funções aplicada por regra, não por confiança.
- **Portal do fornecedor** para ficha cadastral, documentos com validade, propostas,
  notas e comprovantes de encargos — sem senha, por link de e-mail.
- **Quadro de concorrência nativo**, com equalização, rodadas de negociação, comparação
  com o orçamento e os três testes de justificativa obrigatória.
- **Relógio de prazos** com contagem regressiva, alerta e escalonamento nos seis prazos
  do processo.
- **Cofre de documentos** vinculado ao registro, encerrando o arquivamento em pastas e
  caixas de e-mail pessoais.
- **Integração com o SIENGE** para leitura de cadastros e criação de títulos, com fila
  de erros visível e reprocessável.
- **Painel de indicadores** de prazo, aderência a orçamento, concorrência e risco.

### O que permanece fora

Contabilidade, apuração de tributos e folha; programação e execução do pagamento;
orçamento de obra; recebimento físico de material; e qualquer substituição de módulo
do SIENGE.

---

## 3. O fluxo e os checkpoints do sistema

Checkpoint é o ponto em que o sistema verifica antes de deixar o processo avançar. São
doze, distribuídos pelos dois fluxos.

### Fig. 1 — Compra e contratação

```
                                    ┌─────────────────┐
                                    │ Pedido          │ ──────┐
                              ┌────►│ C4 · A2         │       │
                              │     └─────────────────┘       │
                              │        MATERIAL               │
┌────────────┐   ┌────────────┴───┐                           ▼
│ Solicitação│──►│ Cotação · QC   │                  ┌──────────────┐
│ C1         │   │ C2 · C3        │                  │ Nota fiscal  │
└────────────┘   └────────────┬───┘                  │ C6           │
                              │        SERVIÇO       └──────┬───────┘
                              │                             │
                              │     ┌───────────┐   ┌───────▼──────┐
                              └────►│ Contrato  │──►│ Medição      │
                                    │ C4 · A3   │   │ C5 · A4      │
                                    └───────────┘   └───────┬──────┘
                                                            │
                                                            └──► (nota fiscal)

                    ┌──────────┐   ┌────────────────────────────┐
  (nota fiscal) ──► │ Título   │──►│ Liberação p/ pagamento     │
                    │ C7       │   │ C8 · A5 se fora do prazo   │
                    └──────────┘   └────────────────────────────┘
```

O caminho se divide depois da cotação e volta a se juntar na nota fiscal. Cada
checkpoint é uma verificação que o sistema faz antes de liberar a etapa seguinte.

### Fig. 2 — Despesas sem pedido nem contrato

```
┌──────────────────────┐   ┌─────────────────────┐   ┌──────────────────┐   ┌────────────────────┐
│ Solicitação de       │──►│ Aprovação da        │──►│ Título no SIENGE │──►│ Conferência do     │
│ pagamento · C9       │   │ despesa · C10 · D1  │   │ C11              │   │ título · C12       │
└──────────────────────┘   └─────────────────────┘   └──────────────────┘   └────────────────────┘
```

Tributos, ITBI, taxas, contas de consumo e reembolso. Cerca de 40 títulos por mês.

### O que cada checkpoint impede

#### Compra e contratação

| Checkpoint | O que o sistema verifica | O que ele impede |
|---|---|---|
| **C1** · Entrada da solicitação | Campos mínimos por tipo, duplicidade com pedidos e contratos vigentes, saldo orçamentário registrado como alerta | Solicitação incompleta chegando à cotação; compra duplicada do que já está contratado |
| **C2** · Habilitação do fornecedor | Ficha assinada, documentos obrigatórios dentro da validade, verificação de MEI, conta bancária de titularidade do próprio CNPJ | Cotação com fornecedor não habilitado; contratação de MEI; pagamento para conta de terceiro |
| **C3** · Fechamento da cotação | Mínimo de 3 propostas sobre a mesma especificação e justificativa nos três gatilhos: menos de 3 propostas, escolha diferente do menor valor final, valor acima do orçamento | Escolha sem concorrência real ou sem justificativa registrada e visível aos aprovadores |
| **C4** · Aprovação do pedido ou contrato | Cadeia ordenada da obra (construtora → gerenciadora → incorporadora) na escolha; alçada por departamento e valor; segregação de funções; reaprovação quando o valor sobe e em todo aditivo | Pedido seguindo sem o QC aprovado anexado; compromisso acima da alçada de quem aprovou; aumento de valor sem nova aprovação |
| **C5** · Aprovação da medição | Saldo contratado por item, boletim assinado pelo prestador e, na obra, relação de funcionários, folha, INSS (DCTFWeb) e FGTS da competência | Medição acima do contratado sem aditivo; pagamento a prestador com encargos previdenciários em atraso |
| **C6** · Validação fiscal da nota | Autenticidade na SEFAZ ou na prefeitura, retenções, CNO quando há INSS de obra, vínculo com pedido ou medição aprovados, data de chegada | Nota inautêntica ou divergente entrando no sistema; retenção calculada errada; nota sem lastro gerando título |
| **C7** · Geração do título | Retenções idênticas às validadas pelo fiscal, retenção contratual de 5% na obra, dados de pagamento vindos exclusivamente do cadastro do fornecedor | Título divergente da nota; pagamento em conta informada por e-mail ou no corpo da nota |
| **C8** · Liberação para pagamento | Conferência contra a origem, prazo de 15 dias entre chegada e vencimento, aprovação do atraso e prorrogação quando fora do prazo | Título seguindo para pagamento sem conferência ou sem a aprovação formal do atraso |

#### Despesas sem pedido nem contrato

| Checkpoint | O que o sistema verifica | O que ele impede |
|---|---|---|
| **C9** · Entrada da despesa | Tipo restrito à lista de documentos autorizados, anexos obrigatórios por tipo, duplicidade por credor, documento e valor, prazo de 10 dias com justificativa | Despesa fora da lista sendo paga sem pedido; solicitação duplicada; guia registrada já vencida |
| **C10** · Aprovação da despesa | Alçada por departamento e valor, segregação de funções, reembolso do próprio aprovador subindo um nível | Título criado sem aprovação registrada; pessoa aprovando o próprio reembolso |
| **C11** · Criação do título | Origem exclusivamente em solicitação aprovada; registro de falha de integração até o reprocessamento; contingência identificada como exceção | Título avulso criado fora do fluxo aprovado; falha de integração passando despercebida |
| **C12** · Conferência do título | Comparação automática entre o título e a solicitação aprovada, com divergências destacadas; segregação entre quem lança e quem confere | Divergência entre o aprovado e o lançado chegando à programação de pagamento |

> **Os checkpoints não são novos.** Todos saem dos 28 controles-chave já declarados nos
> procedimentos da CASA8. A diferença é que hoje dependem de alguém lembrar de
> verificar, e passariam a ser condição para o processo avançar.

---

## 4. Etapas da implementação

Cinco etapas, começando por um mapeamento contratado à parte. Cada etapa termina em
algo em produção, usado por gente de verdade.

### Fig. 3 — Sequência das fases

```
              confirmação de escopo,
              prazo e investimento
                       ┆
┌───────────────────┐  ┆  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│ Fase 0            │  ┆  │ Fase 1       │   │ Fase 2       │   │ Fase 3       │   │ Fase 4       │
│ Mapeamento        │──┆─►│ Despesas     │──►│ Cotação      │──►│ Contratos    │──►│ Título       │
│ 3 a 4 semanas     │  ┆  │ 8 a 12 sem.  │   │ 10 a 14 sem. │   │ 10 a 14 sem. │   │ 8 a 12 sem.  │
└───────────────────┘  ┆  └──────────────┘   └──────────────┘   └──────────────┘   └──────────────┘
  entregável próprio   ┆
```

A Fase 0 é contratada isoladamente. As Fases 1 a 4 só têm prazo e investimento fechados
depois dela, na linha tracejada.

### Fase 0 · Mapeamento e prova técnica — 3 a 4 semanas

Antes de escrever qualquer linha de código, validamos o processo real e resolvemos o
maior risco técnico do projeto.

- Sessões de mapeamento com controladoria, suprimentos, fiscal e financeiro
- Confronto entre o procedimento escrito e a prática, incluindo leitura de quadros de
  concorrência já preenchidos
- **Prova técnica da API do SIENGE** em homologação: leitura de cadastros, criação de
  título e anexação de documentos
- Levantamento de volumes e consolidação da matriz de alçadas vigente
- Fechamento dos dez pontos em aberto do procedimento de despesas
- Alinhamento com gerenciadora e incorporadora, que aprovam etapas críticas e não são
  da casa

**Entregáveis:** mapa do processo validado em BPMN e em raias; especificação funcional
dos doze checkpoints; relatório da prova técnica com veredito sobre a integração;
backlog priorizado; cronograma e investimento firmes das fases seguintes.

→ *A CASA8 decide seguir ou não, com informação em vez de estimativa.*

### Fase 1 · Fundação e despesas sem pedido — 8 a 12 semanas

A base do sistema — acesso e perfis, espelho de cadastros do SIENGE, matriz de alçadas
única, motor de aprovação, cofre de documentos, relógio de prazos e auditoria — junto
com o processo de despesas completo, da solicitação ao título conferido.

Começamos por aqui porque é o menor processo que atravessa toda a arquitetura e o único
que ainda não tem ferramenta: não há nada para desalojar, e o volume é seguro para
aprender.

*Checkpoints entregues: C9 · C10 · C11 · C12*

→ *Fim da redigitação de ~40 títulos por mês · aprovação rastreável · prazo de 10 dias
sob controle.*

### Fase 2 · Fornecedores e cotação — 10 a 14 semanas

Portal do fornecedor com ficha eletrônica, documentos com controle de validade,
assinatura, bloqueio de MEI e fluxo de alteração bancária com nova assinatura. Quadro
de concorrência nativo com equalização, rodadas de negociação, comparação com o
orçamento e a cadeia ordenada de aprovação da obra.

*Checkpoints entregues: C2 · C3 · C4 (escolha)*

→ *Aposenta a planilha e o formulário Word · fecha o vetor de fraude bancária · começa a
acumular base própria de preços por insumo e fornecedor.*

### Fase 3 · Pedido, contrato e medição — 10 a 14 semanas

Solicitação com alerta orçamentário, pedido e contrato com aprovação por alçada,
controle de aditivos, medição com saldo por item e checklist estruturado de encargos, e
o controle de saldo da retenção contratual de 5% por contrato.

*Checkpoints entregues: C1 · C4 (alçada) · C5*

→ *Aprovações unificadas em um só lugar · risco previdenciário documentado · retenção
contratual finalmente visível.*

### Fase 4 · Nota fiscal, título e indicadores — 8 a 12 semanas

Recepção da nota pelo portal, validação fiscal com conferência de autenticidade e
retenções, geração do título com as retenções corretas, conferência e aprovação de
atraso, e o painel de indicadores de prazo, concorrência, aderência a orçamento e risco.

*Checkpoints entregues: C6 · C7 · C8*

→ *Prazo fiscal de 2 dias úteis medido · aprovação de atraso volta a ser exceção ·
processo gerenciável por número.*

### Sustentação — mensal, opcional

Disponível a partir da entrada em produção da Fase 1: correções, suporte aos usuários,
monitoramento da integração com o SIENGE, atualizações de segurança e pequenas
evoluções dentro de uma franja mensal de horas.

---

Somadas, as Fases 1 a 4 levam de **10 a 13 meses**. As faixas acima são estimativas de
ordem de grandeza e serão substituídas por números firmes ao fim da Fase 0 — que existe
exatamente para isso.

---

## 5. Investimento

Contratação por fase, com preço fechado. Nenhuma fase obriga a seguinte.

| Etapa | Duração | Modalidade | Investimento |
|---|---|---|---|
| **Fase 0 · Mapeamento e prova técnica** | 3 a 4 semanas | Preço fechado | `R$ __________` |
| Fase 1 · Fundação e despesas | 8 a 12 semanas | Preço fechado, definido ao fim da Fase 0 | `R$ __________` |
| Fase 2 · Fornecedores e cotação | 10 a 14 semanas | Preço fechado, definido ao fim da Fase 0 | `R$ __________` |
| Fase 3 · Pedido, contrato e medição | 10 a 14 semanas | Preço fechado, definido ao fim da Fase 0 | `R$ __________` |
| Fase 4 · Nota fiscal, título e indicadores | 8 a 12 semanas | Preço fechado, definido ao fim da Fase 0 | `R$ __________` |
| Sustentação | Mensal | Mensalidade, opcional | `R$ __________` /mês |

### Condição de pagamento por fase

| | |
|---|---|
| **30%** | Na assinatura da fase |
| **40%** | Na entrega em ambiente de homologação |
| **30%** | No aceite em produção |

A Fase 0, por ser curta, é faturada em duas parcelas iguais: na assinatura e na entrega
dos relatórios.

### Por que contratar por fase

**A CASA8 nunca fica presa a um compromisso maior do que já viu funcionar.** Cada fase
entrega algo em produção e pode ser a última. A decisão de seguir é tomada com o sistema
funcionando à frente, não com base em uma promessa de doze meses feita antes de a
primeira linha existir.

A Fase 0 leva isso ao limite: **ela tem valor próprio**. Mesmo que a CASA8 decida não
desenvolver nada, sai dela com o processo mapeado em BPMN, a especificação dos controles
e a resposta definitiva sobre a viabilidade de integrar o SIENGE — material que serve
inclusive para negociar com qualquer outro fornecedor.

---

## 6. O que precisamos da CASA8

Estes itens entram no cronograma. Atraso aqui desloca a entrega na mesma medida.

### Pessoas

- Um **responsável pelo processo** na controladoria, com autoridade para decidir,
  disponível cerca de 4 horas por semana
- Participação de suprimentos, fiscal e financeiro nas sessões de mapeamento da Fase 0
- Um ponto de contato de TI para acessos e ambientes
- Ponte com gerenciadora e incorporadora, que aprovam etapas críticas e são externas ao
  grupo
- Usuários indicados para homologar cada fase antes do aceite

### Acessos e informações

- Acesso de leitura ao SIENGE e credencial de API em **ambiente de homologação**
- Matriz de alçadas vigente de cada departamento
- Lista de documentos autorizados e política de reembolso
- Modelo padrão de contrato de prestação de serviço
- Amostra de quadros de concorrência preenchidos
- Volumes de compras, contratos, medições e notas
- Decisão sobre os dez pontos em aberto do procedimento de despesas

---

## 7. Premissas e exclusões

### Premissas

- O SIENGE está em ambiente de nuvem, com API disponível no contrato vigente.
  **A Fase 0 confirma isso** — se a instalação for em servidor local, a integração é
  replanejada com a CASA8 antes de qualquer fase seguinte.
- Existe ambiente de homologação separado do de produção.
- Os volumes de operação estão na ordem de grandeza informada.
- Os procedimentos vigentes são a base do escopo; mudanças de processo durante o projeto
  são tratadas como alteração de escopo.

### Não incluído

- Licenças e serviços de terceiros: assinatura eletrônica, infraestrutura de nuvem,
  domínios e certificados
- Licenças, módulos ou customizações do SIENGE
- Migração de histórico de processos encerrados
- Contabilidade, apuração de tributos e folha de pagamento
- Programação e execução de pagamentos
- Treinamento presencial além das sessões de homologação de cada fase

---

## 8. Condições comerciais

| Item | Condição |
|---|---|
| Validade da proposta | 30 dias a partir da data de emissão |
| Propriedade intelectual | **O código-fonte é da CASA8.** O repositório fica em conta da própria empresa desde o primeiro dia, com acesso integral |
| Documentação | Decisões de arquitetura, modelo de dados e manual de operação entregues junto com cada fase, no repositório |
| Garantia | `____` dias de correção de defeitos sem custo após o aceite de cada fase |
| Aceite | Homologação pelos usuários indicados, com prazo de `____` dias úteis para manifestação; sem manifestação, considera-se aceito |
| Alteração de escopo | Registrada por escrito, com impacto em prazo e valor acordado antes da execução |
| Confidencialidade | Todas as informações de processo, fornecedores e valores tratadas como confidenciais, com termo próprio se a CASA8 desejar |
| Proteção de dados | Dados pessoais e bancários tratados conforme a LGPD, com criptografia em repouso, acesso por papel e trilha de acesso |
| Reajuste | `____` após 12 meses, para fases contratadas depois desse prazo |

---

## 9. Próximos passos

1. **Reunião de alinhamento** com controladoria e diretoria, para revisar esta proposta
   e ajustar escopo da Fase 0. · *1 hora, pode ser remota*
2. **Aprovação da Fase 0** e assinatura do contrato de mapeamento. · *Sem compromisso
   com as fases seguintes*
3. **Agendamento das sessões de mapeamento** com controladoria, suprimentos, fiscal e
   financeiro. · *Início em até 10 dias da assinatura*
4. **Liberação dos acessos** de leitura ao SIENGE e da credencial de API em homologação.
   · *Pré-requisito da prova técnica*
5. **Apresentação dos resultados da Fase 0**, com o mapa do processo, a especificação
   dos checkpoints e o cronograma e investimento firmes das fases seguintes. · *Quando a
   CASA8 decide se segue*

> **Uma observação honesta sobre alternativas.** Se o objetivo da CASA8 for apenas
> resolver o fluxo de despesas sem pedido no próximo trimestre, sem mexer em
> suprimentos, o cenário de Microsoft 365 previsto no próprio procedimento resolve, e
> esta proposta seria sobre-engenharia.
>
> O que justifica um sistema próprio é o restante: cotação, cadastro de fornecedor,
> prevenção de fraude bancária, encargos previdenciários por medição e retenção
> contratual — nada disso cabe em SharePoint, e numa plataforma de mercado cada item
> vira uma customização cobrada à parte. Vale a empresa decidir isso conscientemente
> antes de contratar qualquer coisa.

---

*Proposta elaborada a partir do Procedimento de Suprimentos CASA8 (v1.0), do
Procedimento de Custos e Despesas sem Pedido ou Contrato CASA8 (v0.1), do modelo de
Quadro de Concorrência, da Ficha Cadastral de Fornecedor e dos fluxos em raias
fornecidos pela controladoria.*

*Os prazos das Fases 1 a 4 são estimativas de ordem de grandeza e serão substituídos por
compromissos firmes ao término da Fase 0.*
