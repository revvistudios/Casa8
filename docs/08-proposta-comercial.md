# Proposta Comercial

## Centralização e automação do processo de suprimentos e despesas

| | |
|---|---|
| **Cliente** | Grupo CASA8 |
| **Proponente** | Revvi Studios |
| **Data** | Setembro de 2026 |
| **Validade da proposta** | 30 dias |
| **Base documental** | Procedimento de Suprimentos v1.0 · Procedimento de Custos e Despesas sem Pedido v0.1 · Quadro de Concorrência · Ficha Cadastral de Fornecedor · fluxos em raias |

---

## Sumário executivo

**O que a CASA8 tem hoje.** Dois procedimentos bem escritos, com 28 controles-chave
declarados, seis pontos de aprovação e seis prazos duros — executados sobre SIENGE,
planilha, formulário Word, e-mail e atenção humana. O processo de despesas sem pedido
ainda está em minuta e não tem ferramenta definida.

**O problema.** Não é o processo: é que ele está distribuído. A cotação e o cadastro de
fornecedor vivem fora de qualquer sistema. A matriz de alçadas, que é uma só, seria
parametrizada em dois lugares. Nenhum dos seis prazos tem contagem regressiva. A
retenção contratual de 5% é retida em toda nota de obra e ninguém controla o saldo.
Quarenta títulos por mês são redigitados no contas a pagar.

**O que propomos.** Um sistema web próprio, integrado ao SIENGE, que centraliza o
processo e converte os 28 controles em **doze checkpoints** que o processo não consegue
pular. O SIENGE permanece como sistema de registro; o sistema novo é a camada de
processo.

**Como começamos.** Por uma **Fase 0 de mapeamento e prova técnica**, de 3 a 4 semanas,
contratada isoladamente e com entregável próprio. Ela valida o processo real, resolve o
maior risco técnico do projeto — a disponibilidade da API do SIENGE — e devolve
cronograma e investimento firmes para as fases seguintes. Se a CASA8 decidir não
prosseguir, fica com o mapeamento, a especificação dos controles e a resposta sobre a
integração.

**Prazo total.** Fases 1 a 4 somam de 10 a 13 meses, contratadas uma a uma. Cada fase
entrega algo em produção e pode ser a última.

**Uma ressalva honesta, logo de início.** Se o objetivo for apenas resolver o fluxo de
despesas sem pedido no próximo trimestre, o cenário de Microsoft 365 previsto no próprio
procedimento da CASA8 resolve, e esta proposta é sobre-engenharia. O que justifica um
sistema próprio é o restante — cotação, cadastro de fornecedor, prevenção de fraude
bancária, encargos previdenciários e retenção contratual. A seção 8 trata disso
abertamente.

---

## 1. Entendimento da necessidade

### 1.1 O que analisamos

Esta proposta parte da leitura integral do material fornecido pela controladoria:

- **Procedimento de Suprimentos** (v1.0) — 9 etapas, 5 aprovações, 15 controles-chave
- **Procedimento de Custos e Despesas sem Pedido ou Contrato** (v0.1) — 4 etapas,
  1 aprovação, 13 controles-chave, com dois cenários de ferramenta em aberto
- **Quadro de Concorrência** (modelo, exemplo preenchido e instruções) — incluindo as
  fórmulas de equalização, negociação e os três testes de justificativa obrigatória
- **Ficha Cadastral de Fornecedor** — campos, documentos obrigatórios e regras de aceite
- **Fluxos em raias** dos três processos, incluindo os dois cenários de despesas

### 1.2 Como o processo funciona hoje

O grupo opera por **SPEs** — uma sociedade por empreendimento, com CNPJ, endereço de
faturamento e CNO próprios. Isso torna multi-empresa um requisito estrutural: errar a
SPE não é erro de digitação, é nota fiscal emitida contra o CNPJ errado, com retenção
previdenciária no CNO errado.

Parte dos aprovadores é externa ao grupo: **gerenciadora de obra** e **incorporadora**
participam da escolha do fornecedor e da aprovação das medições. Não têm conta
corporativa da CASA8, o que condiciona qualquer solução de acesso.

**As seis aprovações, e onde cada uma mora hoje**

| Cód. | Etapa | Quem aprova | Onde é registrada |
|---|---|---|---|
| **A1** | Cotação · QC | Obra: construtora → gerenciadora → incorporadora, nesta ordem. Administrativo: diretor da área | **Fora de sistema** — assinatura na planilha ou e-mail anexado |
| **A2** | Pedido de compra | Matriz de alçadas do departamento | SIENGE |
| **A3** | Contrato e todo aditivo | Matriz de alçadas do departamento | SIENGE |
| **A4** | Medição | Obra: gerenciadora. Administrativo: diretor | SIENGE |
| **A5** | Pagamento fora do prazo | Financeiro | SIENGE |
| **D1** | Despesa sem pedido | Matriz de alçadas do departamento | **Não existe ainda** |

**Os seis prazos do processo**

| Prazo | Regra |
|---|---|
| Dia 25 | Nota emitida e enviada até o dia 25, para conferir e recolher retenções no prazo legal |
| 15 dias | Nota chega com no mínimo 15 dias do vencimento; abaixo disso depende de A5 |
| 2 dias úteis | Validação fiscal, contados do recebimento pelo fiscal |
| 10 dias | Despesa sem pedido enviada com no mínimo 10 dias do vencimento |
| Por medição | Comprovantes de encargos do prestador, obrigatórios para aprovar a medição da obra |
| 5 · 14 · 21 · 28 | Datas de pagamento do grupo, exceto impostos |

Os prazos se comem: nota emitida no dia 25 com vencimento no dia 5 já nasce com 11 dias,
abaixo do mínimo de 15. Somados os 2 dias de validação fiscal, a folga real cai para
nove. É por isso que a aprovação A5, criada como exceção, é acionada com frequência.

### 1.3 Onde o processo perde valor hoje

Nove pontos, identificados na análise e ordenados por impacto:

| | Ponto | Consequência |
|---|---|---|
| **1** | A **cotação** vive numa planilha, e a A1 — que na obra percorre três empresas em cadeia ordenada — é registrada por assinatura no arquivo ou e-mail | Sem trilha de auditoria confiável; sem saber onde um QC está parado; **o histórico de preços por insumo e fornecedor não se acumula em lugar nenhum** |
| **2** | O **cadastro de fornecedor** é um formulário Word que vai e volta por e-mail, com até seis documentos anexos | Ciclo lento bloqueando cotação; conferência sem rastro; nenhum controle de validade das certidões que o contrato exige atualizadas |
| **3** | **Alteração de dados bancários** é validada por uma pessoa lendo um e-mail | É o vetor da fraude do boleto no setor. A regra escrita está correta; a aplicação é manual |
| **4** | Aprovar tem **três mecânicas diferentes**, e a matriz de alçadas — que é uma só — seria parametrizada em dois sistemas | Ninguém tem uma visão de "o que espera por mim"; não há SLA por aprovação; divergência entre as duas cópias da matriz é questão de tempo |
| **5** | Nenhum dos **seis prazos** tem contagem regressiva ou alerta | A aprovação A5 existe para tratar uma falha recorrente a jusante, em vez de evitá-la a montante |
| **6** | **Comprovantes de encargos** (folha, DCTFWeb, FGTS) conferidos visualmente a cada medição | Controle de responsabilidade solidária previdenciária — um dos maiores riscos financeiros de uma construtora — executado sem registro estruturado |
| **7** | A **retenção contratual de 5%** é retida em toda nota de obra e "registrada para liberação futura"; a liberação está fora de todos os procedimentos | Valor relevante parado, sem dono, sem relatório e sem controle de termo de encerramento ou prazo de garantia |
| **8** | **Redigitação** de ~40 títulos por mês no contas a pagar, com uma segunda pessoa conferindo a digitação | A conferência existe para pegar o erro que a redigitação cria |
| **9** | **Nenhum indicador** definido em qualquer procedimento | Ninguém sabe o lead time da cotação, a aderência ao orçamento por obra nem quantas notas dependeram de A5 |

### 1.4 Uma evidência concreta do que analisamos

Para dar substância ao entendimento, refizemos o quadro de concorrência do exemplo
constante da planilha da CASA8 — QC-2026-015, concreto usinado com bombeamento para as
lajes do 3º ao 5º pavimento, orçamento de R$ 63.900:

| Etapa | Alfa | Beta | Gama |
|---|---:|---:|---:|
| Subtotal dos itens | 63.150 *(2º)* | 64.200 *(3º)* | **62.640** *(1º)* |
| Frete, quando FOB | — | — | +3.200 |
| Descontos comerciais | — | −1.500 | — |
| **Total equalizado** | 63.150 *(2º)* | **62.700** *(1º)* | 65.840 *(3º)* |
| 1ª negociação | 62.000 | 62.100 | — |
| 2ª negociação | — | 61.500 | — |
| **Valor final** | 62.000 *(2º)* | **61.500** *(1º)* | 65.840 *(3º)* |

A equalização inverteu o resultado: a Gama era a mais barata no papel e virou a mais
cara, só por causa dos R$ 3.200 de frete FOB que não apareciam no preço unitário. A
negociação valeu R$ 1.200, e a escolha final ficou R$ 2.400 abaixo do orçamento — 3,76%
de folga para a obra.

**É exatamente esse raciocínio que hoje morre num arquivo.** Registrado em sistema, cada
item cotado vira base de preços por insumo, fornecedor, obra e data — o ativo de dado
mais valioso que o sistema passa a gerar, e que nenhuma das alternativas em avaliação
entrega.

---

## 2. A abordagem

### 2.1 O princípio que rege todo o projeto

> **O SIENGE continua sendo o sistema de registro. O sistema novo é a camada de processo.**

O SIENGE permanece dono dos cadastros, pedidos, contratos, medições, notas e contas a
pagar — é o que a auditoria e o fisco olham. O sistema novo é onde as pessoas trabalham:
cotação, aprovações, documentos, prazos, portal do fornecedor e indicadores. Ao fim de
cada etapa, **ele escreve o resultado no SIENGE**.

A regra prática para todo requisito: *se o dado precisa existir na contabilidade, ele
nasce no sistema novo e termina no SIENGE; se o dado é sobre como se chegou até a
decisão, ele vive só no sistema novo.*

Não se constrói um ERP paralelo. Isso mantém o projeto viável, a contabilidade íntegra e
a CASA8 livre para trocar de fornecedor de software sem refazer o processo.

### 2.2 O que o sistema faz

**Núcleo**

- **Matriz de alçadas única**, parametrizável por departamento e faixa de valor, servindo
  todas as aprovações do grupo — fim das duas cópias da mesma regra
- **Motor de aprovação** com modos serial, paralelo e em cadeia ordenada (a A1 da obra),
  delegação por ausência, reprovação com motivo obrigatório e **segregação de funções
  aplicada por regra**: quem pede não aprova, quem emite não aprova, quem mede não
  aprova, quem lança o título não confere, e reembolso do próprio aprovador sobe um nível
- **Acesso para externos** — gerenciadora, incorporadora e fornecedores entram por link
  assinado enviado por e-mail, sem conta corporativa e sem instalar nada
- **Cofre de documentos** com versionamento, controle de validade e vínculo ao registro
  do SIENGE, encerrando o arquivamento em pastas e caixas de e-mail pessoais
- **Relógio de prazos** com contagem regressiva, alerta e escalonamento nos seis prazos
- **Auditoria imutável** — quem fez, o quê, quando, com qual valor antes e depois

**Processo de suprimentos**

- Solicitação com alerta orçamentário registrado e visível até a aprovação
- **Portal do fornecedor**: ficha cadastral eletrônica, upload de documentos com
  validade, verificação de MEI, propostas, notas fiscais, comprovantes de encargos e
  fluxo de alteração bancária com nova assinatura
- **Quadro de concorrência nativo**: itens, propostas, equalização (frete FOB, impostos
  não inclusos, descontos), rodadas de negociação, comparação com o orçamento, os três
  testes de justificativa obrigatória e a cadeia ordenada de A1
- Pedido de compra, contrato e aditivos, com aprovação por alçada e controle de
  reaprovação quando o valor sobe
- Medição com saldo por item, boletim assinado e **checklist estruturado de encargos**
  (relação de funcionários, folha, DCTFWeb, FGTS Digital)
- Recepção e validação fiscal da nota: autenticidade, retenções, CNO, prazo e recusa
- Geração e conferência do título, com **controle do saldo de retenção contratual de 5%**
  por contrato, incluindo termo de encerramento e prazo de garantia

**Processo de despesas sem pedido**

- Formulário por tipo, restrito à lista de documentos autorizados, com anexos
  obrigatórios por tipo
- Validação automática de prazo, duplicidade e campos
- Aprovação por alçada, acionamento de suprimentos quando o credor não existe
- Criação do título no SIENGE pela integração, com anexos, e conferência

**Gestão**

- Painel de pendências por pessoa — "o que espera por mim"
- Indicadores de prazo, concorrência, aderência a orçamento e risco (seção 7)

### 2.3 O que permanece fora

Contabilidade, apuração de tributos e folha de pagamento; programação e execução do
pagamento; orçamento de obra (consumido como referência, mantido no SIENGE); recebimento
físico de material em canteiro; e qualquer substituição de módulo do SIENGE.

---

## 3. Os checkpoints do sistema

Checkpoint é o ponto em que o sistema verifica antes de deixar o processo avançar. São
doze, distribuídos pelos dois fluxos. Todos saem dos 28 controles-chave já declarados
nos procedimentos da CASA8 — a diferença é que hoje dependem de alguém lembrar de
verificar, e passariam a ser condição para o processo avançar.

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

### Fig. 2 — Despesas sem pedido nem contrato

```
┌──────────────────────┐   ┌─────────────────────┐   ┌──────────────────┐   ┌────────────────────┐
│ Solicitação de       │──►│ Aprovação da        │──►│ Título no SIENGE │──►│ Conferência do     │
│ pagamento · C9       │   │ despesa · C10 · D1  │   │ C11              │   │ título · C12       │
└──────────────────────┘   └─────────────────────┘   └──────────────────┘   └────────────────────┘
```

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

### Como os controles são aplicados

Cada checkpoint atua de uma das quatro formas, conforme o que o próprio procedimento
determina:

| Forma | O que faz | Exemplo |
|---|---|---|
| **Bloqueio** | Impede a transição; não há como seguir | Menos de 3 propostas sem justificativa → o QC não vai para A1 |
| **Exigência** | Exige um dado ou documento antes de liberar | A4 da obra não abre sem folha, DCTFWeb e FGTS da competência |
| **Alerta** | Avisa e registra, sem travar | Alerta orçamentário visível até A2 ou A3 — o procedimento manda não travar |
| **Evidência** | Registra automaticamente o que hoje se anota à mão | Título lançado em contingência fica identificado como exceção |

> **Não haverá botão de urgência.** O procedimento de suprimentos determina que *"não há
> fluxo de urgência; todas as compras e contratações seguem este procedimento"*. Todo
> sistema de aprovação que ganha um modo urgente vê esse modo virar o caminho padrão em
> seis meses. A velocidade será construída no caminho normal, não num desvio.

---

## 4. Arquitetura e tecnologia

### 4.1 Composição do sistema

Escolhas guiadas por dois critérios: **manutenibilidade de longo prazo** e **ausência de
dependência de fornecedor**.

| Camada | Escolha | Por quê |
|---|---|---|
| Aplicação e interface | Next.js com TypeScript | Um único repositório e um único deploy para interface e back-end; tecnologia amplamente adotada, com mercado de profissionais |
| Banco de dados | PostgreSQL | Transações confiáveis, migrações versionadas, base madura para auditoria |
| Autenticação interna | Microsoft Entra ID (SSO) | A CASA8 já usa Microsoft 365; evita mais um usuário e senha para gerenciar |
| Acesso externo | Convite por e-mail com link assinado e escopo restrito | Gerenciadora, incorporadora e fornecedores não terão conta corporativa |
| Armazenamento de arquivos | Serviço compatível com S3, com URLs assinadas de curta duração | Documentos nunca no banco; acesso controlado e expirável |
| Fila de integração | Fila persistente com repetição e fila de erros | A integração com o SIENGE nunca bloqueia o usuário |
| Assinatura eletrônica | Integração com provedor de mercado | A ficha cadastral e o contrato exigem assinatura |
| Monitoramento | Registro estruturado, captura de erros e verificação de saúde | Operação previsível desde o primeiro dia |
| Hospedagem | Container, com backup automatizado do banco | Portável entre provedores; sem aprisionamento |

**Código-fonte e repositório são da CASA8 desde o primeiro dia**, em conta da própria
empresa. Decisões de arquitetura, modelo de dados e manual de operação são entregues
junto com cada fase, no mesmo repositório.

### 4.2 Como a integração com o SIENGE funciona

Quatro princípios:

1. **Assíncrona sempre.** O usuário nunca espera o SIENGE responder. Concluída a
   aprovação, um evento entra na fila; a criação do título acontece em segundo plano.
2. **Falha é tela, não registro de log.** O financeiro vê "3 títulos falharam, este é o
   motivo, este é o botão de reprocessar" — e a falha permanece registrada até ser
   resolvida.
3. **Contingência é estado de primeira classe.** O próprio procedimento prevê que, com a
   integração indisponível e vencimento próximo, o financeiro lança manualmente. O título
   fica marcado como contingência e é conferido com a mesma regra.
4. **Nas divergências de cadastro, o SIENGE sempre vence.** O sistema mantém cópia de
   leitura com carimbo de sincronização visível, e nunca se torna dono desses dados.

```
  Sistema CASA8                    Fila                         SIENGE
 ┌──────────────┐            ┌───────────────┐            ┌─────────────┐
 │ Aprovação    │──evento───►│ criar título  │───API─────►│ Contas      │
 │ concluída    │            │ repete 5x     │◄──resposta─│ a pagar     │
 └──────────────┘            └───────┬───────┘            └─────────────┘
                                     │ falha persistente
                                     ▼
                           ┌─────────────────────┐
                           │ Fila de erros       │  visível ao financeiro,
                           │ causa + reprocessar │  ou marcada como contingência
                           └─────────────────────┘
```

### 4.3 Segurança e proteção de dados

- Dados bancários de fornecedores e colaboradores **criptografados em repouso**
- Acesso por papel, com o princípio do menor privilégio; externos veem apenas o que lhes
  diz respeito, e **nunca a proposta de um concorrente**
- Trilha de acesso auditável, além da trilha de alteração
- Credenciais de integração como segredo gerenciado, com rotação prevista — nunca em
  código
- Tratamento conforme a **LGPD**, com finalidade, retenção e base legal definidas para os
  dados pessoais da ficha cadastral (nome, CPF e contatos de representantes)
- **Quarentena em alteração de dados bancários**: nova ficha assinada, confirmação por
  canal independente e janela de espera antes do primeiro pagamento na conta nova —
  defesa padrão contra a fraude do boleto

---

## 5. O maior risco do projeto, e como ele é tratado

Esta seção existe porque um fornecedor que esconde o risco principal cobra por ele
depois.

### A pergunta que precede todas as outras

> **O SIENGE da CASA8 está em ambiente de nuvem ou em servidor local?**

A documentação pública do Sienge Plataforma indica que **clientes com servidor local não
têm acesso às APIs** — os recursos de integração são disponibilizados para clientes em
datacenter. Se a instalação da CASA8 for local, a integração desta proposta — e o
cenário B previsto no próprio procedimento de despesas — ficam inviáveis como desenhados.

Há uma segunda pergunta logo atrás: a API de **inserção** de títulos que consta na
documentação pública é a baseada em **nota fiscal eletrônica recebida**. Mas o caso de
uso das despesas sem pedido é exatamente o **título avulso sem nota** — guia de tributo,
ITBI, taxa de prefeitura e reembolso não têm NF-e.

### Como tratamos

A **Fase 0** inclui uma prova técnica com chamadas reais em ambiente de homologação,
cobrindo acesso e autenticação, leitura de cadastros, criação de título avulso, anexação
de documentos, comportamento sob erro e idempotência. O resultado é um relatório com
veredito — antes de qualquer compromisso de prazo ou valor nas fases seguintes.

### Se a escrita não for possível

O projeto não morre; o desenho muda e a maior parte do valor é preservada. O sistema
entrega uma ficha de lançamento pronta para digitar e, em seguida, **lê o título criado e
o compara automaticamente com o que foi aprovado**, apontando divergências. A leitura em
massa tem muito mais chance de estar disponível que a escrita.

A conferência humana deixa de ser digitação contra digitação e passa a ser análise da
divergência que a máquina apontou. Preserva-se aprovação, rastreabilidade, prazos,
anexos e indicadores; perde-se apenas a eliminação da digitação.

---

## 6. Etapas da implementação

Cinco etapas, começando por um mapeamento contratado à parte. Cada etapa termina em algo
em produção, usado por gente de verdade — não há "fase de análise" sem entrega.

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

### Fase 0 · Mapeamento e prova técnica — 3 a 4 semanas

Antes de escrever qualquer linha de código, validamos o processo real e resolvemos o
maior risco técnico do projeto.

**O que fazemos**

- Sessões de mapeamento com controladoria, suprimentos, fiscal e financeiro
- Confronto entre o procedimento escrito e a prática, incluindo leitura de quadros de
  concorrência já preenchidos — é nas exceções que o processo real aparece
- **Prova técnica da API do SIENGE** em homologação (seção 5)
- Levantamento de volumes reais: compras por mês, contratos ativos, medições por mês e
  notas por mês
- Consolidação da matriz de alçadas vigente de cada departamento
- Fechamento dos dez pontos em aberto do procedimento de despesas: tributos abrangidos,
  ITBI e taxas, controle de unidades consumidoras, política de reembolso, manutenção da
  lista de documentos autorizados, verificação orçamentária e documentos do cadastro de
  credor
- Alinhamento com gerenciadora e incorporadora, que aprovam etapas críticas e não são da
  casa

**Entregáveis**

1. Mapa do processo validado, em BPMN e em raias, refletindo a prática e não apenas o
   documento
2. Especificação funcional dos doze checkpoints, com regra, gatilho e comportamento
3. Relatório da prova técnica da integração, com veredito e plano alternativo se
   necessário
4. Backlog priorizado, com o escopo de cada fase detalhado
5. **Cronograma e investimento firmes das Fases 1 a 4**

→ *A CASA8 decide seguir ou não com informação, em vez de estimativa.*

### Fase 1 · Fundação e despesas sem pedido — 8 a 12 semanas

A base do sistema — acesso e perfis, espelho de cadastros do SIENGE, matriz de alçadas
única, motor de aprovação, cofre de documentos, relógio de prazos e auditoria — junto com
o processo de despesas completo, da solicitação ao título conferido.

Começamos por aqui porque é o menor processo que atravessa toda a arquitetura e o único
que ainda não tem ferramenta: não há nada para desalojar, e o volume de ~40 títulos por
mês é seguro para aprender. Substitui, na prática, a decisão entre os cenários A e B que
está em aberto.

*Checkpoints entregues: C9 · C10 · C11 · C12*

→ *Fim da redigitação de ~40 títulos por mês · aprovação rastreável · prazo de 10 dias
sob controle.*

### Fase 2 · Fornecedores e cotação — 10 a 14 semanas

Portal do fornecedor com ficha eletrônica, documentos com controle de validade,
assinatura, bloqueio de MEI e fluxo de alteração bancária com nova assinatura. Quadro de
concorrência nativo com equalização, rodadas de negociação, comparação com o orçamento e
a cadeia ordenada de aprovação da obra.

*Checkpoints entregues: C2 · C3 · C4 (escolha)*

→ *Aposenta a planilha e o formulário Word · fecha o vetor de fraude bancária · começa a
acumular base própria de preços por insumo e fornecedor.*

### Fase 3 · Pedido, contrato e medição — 10 a 14 semanas

Solicitação com alerta orçamentário, pedido e contrato com aprovação por alçada, controle
de aditivos, medição com saldo por item e checklist estruturado de encargos, e o controle
de saldo da retenção contratual de 5% por contrato.

*Checkpoints entregues: C1 · C4 (alçada) · C5*

→ *Aprovações unificadas em um só lugar · risco previdenciário documentado · retenção
contratual finalmente visível.*

### Fase 4 · Nota fiscal, título e indicadores — 8 a 12 semanas

Recepção da nota pelo portal, validação fiscal com conferência de autenticidade e
retenções, geração do título com as retenções corretas, conferência e aprovação de
atraso, e o painel de indicadores.

*Checkpoints entregues: C6 · C7 · C8*

→ *Prazo fiscal de 2 dias úteis medido · aprovação de atraso volta a ser exceção ·
processo gerenciável por número.*

### Sustentação — mensal, opcional

Disponível a partir da entrada em produção da Fase 1: correções, suporte aos usuários,
monitoramento da integração com o SIENGE, atualizações de segurança e pequenas evoluções
dentro de uma franja mensal de horas.

---

Somadas, as Fases 1 a 4 levam de **10 a 13 meses**. As faixas acima são estimativas de
ordem de grandeza e serão substituídas por números firmes ao fim da Fase 0 — que existe
exatamente para isso.

---

## 7. Indicadores que o sistema entrega

Hoje nenhum procedimento define métrica. A partir da Fase 1, e completos na Fase 4, o
sistema passa a produzir:

**Suprimentos**
- Prazo da solicitação até o pedido ou contrato, por obra e por comprador
- Percentual de cotações com 3 ou mais propostas, e percentual com justificativa de
  exceção
- Percentual de escolhas diferentes do menor valor final, por motivo
- Aderência ao orçamento: diferença em reais e em percentual entre valor escolhido e
  orçado, por obra e por etapa
- Economia de negociação: do total equalizado ao valor final

**Aprovações**
- Tempo médio e de percentil 90 por código de aprovação e por aprovador
- Fila atual: aprovações pendentes por idade
- Taxa de reprovação e motivos mais frequentes

**Fiscal e financeiro**
- Percentual de notas validadas dentro dos 2 dias úteis
- Percentual de notas recusadas, por motivo
- Percentual de títulos que dependeram de aprovação de atraso — meta: tendência a zero
- Percentual de despesas sem pedido enviadas com menos de 10 dias

**Risco**
- Contratos com comprovantes de encargos pendentes ou vencidos
- Fornecedores com documento vencido
- **Saldo de retenção contratual por contrato e por obra**
- Falhas de integração abertas e títulos lançados em contingência

---

## 8. Comparação com as alternativas

O procedimento de despesas da CASA8 apresenta dois cenários para escolha. Esta proposta é
um terceiro, e cabe compará-los abertamente.

| Critério | A · Microsoft 365 | B · Plataforma BPM contratada | **C · Sistema próprio** |
|---|---|---|---|
| Despesas sem pedido | Resolve | Resolve | Resolve |
| Cotação e quadro de concorrência | Não cobre | Customização paga | **Nativo, com base de preços** |
| Portal do fornecedor | Não cobre | Customização paga | **Nativo** |
| Encargos por medição | Não cobre | Customização paga | **Nativo** |
| Retenção contratual de 5% | Não cobre | Customização paga | **Nativo** |
| Indicadores de processo | Limitado | Conforme plataforma | **Nativo** |
| Custo inicial | Baixo | Médio-alto | Alto — tempo de desenvolvimento |
| Custo recorrente | Incluído no Microsoft 365 | Assinatura por usuário + sustentação | Infraestrutura baixa + sustentação |
| Tempo até o primeiro valor | Semanas | Meses | ~3 meses para o primeiro processo |
| Dependência de fornecedor | Microsoft | **Alta** | **Nenhuma — código da CASA8** |
| Risco principal | Teto funcional baixo | Preso ao contrato e ao roadmap do fornecedor | Equipe reduzida (ver seção 9) |

### A ressalva honesta

**Se o objetivo da CASA8 for apenas resolver o fluxo de despesas sem pedido no próximo
trimestre, sem mexer em suprimentos, o cenário A é a resposta certa** — quarenta títulos
por mês, um formulário e uma aprovação. SharePoint com Power Automate resolve, e um
sistema próprio seria sobre-engenharia.

O cenário C se justifica se — e somente se — o objetivo for **centralizar o processo
inteiro**. O argumento não está nas despesas: está na cotação, no cadastro de fornecedor,
na prevenção de fraude bancária, nos encargos previdenciários e na retenção contratual.
Nada disso cabe em SharePoint, e numa plataforma de mercado cada item vira uma
customização cobrada à parte.

Vale a empresa decidir isso conscientemente antes de contratar qualquer coisa — e a
Fase 0 é barata justamente para permitir essa decisão com dados.

---

## 9. Riscos e mitigações

| | Risco | Impacto | Mitigação |
|---|---|---|---|
| **R1** | A API do SIENGE não expõe criação de título avulso, anexos ou medição | **Crítico** — muda o desenho | Prova técnica na Fase 0, antes de qualquer compromisso. Plano alternativo detalhado na seção 5, que preserva a maior parte do valor |
| **R2** | Equipe reduzida de desenvolvimento | **Alto** | Tecnologias de mercado amplo; código e repositório da CASA8 desde o primeiro dia; testes automatizados nos fluxos de aprovação; decisões de arquitetura documentadas; ampliação da equipe prevista a partir da Fase 3 |
| **R3** | Gerenciadora e incorporadora não adotam — A1 e A4 travam | **Alto** | Envolvimento já na Fase 0; aprovação por e-mail em um clique; nunca exigir instalação ou conta corporativa |
| **R4** | Fornecedores não usam o portal | Médio | Acesso sem senha, por link assinado; recepção por e-mail mantida com ingestão assistida durante a transição |
| **R5** | Escopo aberto — dez pontos pendentes só no procedimento de despesas | Médio | A Fase 0 fecha os pontos; contratação por fase, não por projeto inteiro |
| **R6** | Divergência entre o espelho de cadastros e o SIENGE | Médio | Sincronização com carimbo de data, tela de divergências, SIENGE sempre prevalece |
| **R7** | Resistência a "mais um sistema" | Médio | O sistema precisa ser **mais rápido** que a planilha no primeiro dia, não apenas mais controlado. Mediremos o tempo de tarefa antes e depois |
| **R8** | Tratamento de dados pessoais e bancários | Médio | Criptografia em repouso, acesso por papel, retenção definida e trilha de acesso (seção 4.3) |

---

## 10. Investimento

Contratação por fase, com preço fechado. **Nenhuma fase obriga a seguinte.**

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
funcionando à frente, não com base numa promessa de doze meses feita antes de a primeira
linha existir.

A Fase 0 leva isso ao limite: **ela tem valor próprio**. Mesmo que a CASA8 decida não
desenvolver nada, sai dela com o processo mapeado em BPMN, a especificação dos controles
e a resposta definitiva sobre a viabilidade de integrar o SIENGE — material que serve
inclusive para negociar com qualquer outro fornecedor, nos cenários A ou B.

---

## 11. O que precisamos da CASA8

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
- Confirmação sobre o ambiente do SIENGE (nuvem ou servidor local) e sobre a inclusão de
  uso de API no contrato vigente
- Matriz de alçadas vigente de cada departamento
- Lista de documentos autorizados e política de reembolso
- Modelo padrão de contrato de prestação de serviço
- Amostra de quadros de concorrência preenchidos
- Volumes de compras, contratos, medições e notas
- Confirmação sobre contas Microsoft 365 para solicitantes de obra, colaboradores e
  aprovadores
- Decisão sobre os dez pontos em aberto do procedimento de despesas

---

## 12. Premissas e exclusões

### Premissas

- O SIENGE está em ambiente de nuvem, com API disponível no contrato vigente. **A Fase 0
  confirma isso** — se a instalação for em servidor local, a integração é replanejada com
  a CASA8 antes de qualquer fase seguinte, conforme a seção 5
- Existe ambiente de homologação separado do de produção
- Os volumes de operação estão na ordem de grandeza informada
- Os procedimentos vigentes são a base do escopo; mudanças de processo durante o projeto
  são tratadas como alteração de escopo

### Não incluído

- Licenças e serviços de terceiros: assinatura eletrônica, infraestrutura de nuvem,
  domínios e certificados
- Licenças, módulos ou customizações do SIENGE
- Migração de histórico de processos encerrados
- Contabilidade, apuração de tributos e folha de pagamento
- Programação e execução de pagamentos
- Treinamento presencial além das sessões de homologação de cada fase

---

## 13. Condições comerciais

| Item | Condição |
|---|---|
| Validade da proposta | 30 dias a partir da data de emissão |
| Propriedade intelectual | **O código-fonte é da CASA8.** O repositório fica em conta da própria empresa desde o primeiro dia, com acesso integral |
| Documentação | Decisões de arquitetura, modelo de dados e manual de operação entregues junto com cada fase, no repositório |
| Garantia | `____` dias de correção de defeitos sem custo após o aceite de cada fase |
| Aceite | Homologação pelos usuários indicados, com prazo de `____` dias úteis para manifestação; sem manifestação, considera-se aceito |
| Alteração de escopo | Registrada por escrito, com impacto em prazo e valor acordado antes da execução |
| Confidencialidade | Todas as informações de processo, fornecedores e valores tratadas como confidenciais, com termo próprio se a CASA8 desejar |
| Proteção de dados | Conforme a LGPD, nos termos da seção 4.3 |
| Reajuste | `____` após 12 meses, para fases contratadas depois desse prazo |

---

## 14. Próximos passos

1. **Reunião de alinhamento** com controladoria e diretoria, para revisar esta proposta e
   ajustar o escopo da Fase 0. · *1 hora, pode ser remota*
2. **Aprovação da Fase 0** e assinatura do contrato de mapeamento. · *Sem compromisso com
   as fases seguintes*
3. **Agendamento das sessões de mapeamento** com controladoria, suprimentos, fiscal e
   financeiro. · *Início em até 10 dias da assinatura*
4. **Liberação dos acessos** de leitura ao SIENGE e da credencial de API em homologação.
   · *Pré-requisito da prova técnica*
5. **Apresentação dos resultados da Fase 0**, com o mapa do processo, a especificação dos
   checkpoints e o cronograma e investimento firmes das fases seguintes. · *Quando a
   CASA8 decide se segue*

---

## Anexos

Material de análise que sustenta esta proposta, disponível para consulta da CASA8:

| Documento | Conteúdo |
|---|---|
| Entendimento do domínio | Fluxos, aprovações, prazos, entidades e as regras de negócio extraídas dos procedimentos |
| Diagnóstico | Os nove pontos de perda de valor e a priorização por retorno sobre esforço |
| Modelo de dados | Entidades, o modelo que substitui a planilha de cotação e as máquinas de estado |
| Integração com o SIENGE | O que a documentação pública confirma, o roteiro da prova técnica e o plano alternativo |
| Mapa de controles | Os 28 controles-chave dos procedimentos convertidos em mecanismo de sistema |
| Questões em aberto | O que precisa ser respondido para fechar escopo, incluindo os dez pontos do procedimento de despesas |

---

*Proposta elaborada a partir do Procedimento de Suprimentos CASA8 (v1.0), do Procedimento
de Custos e Despesas sem Pedido ou Contrato CASA8 (v0.1), do modelo de Quadro de
Concorrência, da Ficha Cadastral de Fornecedor e dos fluxos em raias fornecidos pela
controladoria.*

*As informações sobre a API do Sienge Plataforma baseiam-se em documentação pública,
sujeita a mudança e ao contrato específico da CASA8; a verificação definitiva ocorre na
Fase 0.*

*Os prazos das Fases 1 a 4 são estimativas de ordem de grandeza e serão substituídos por
compromissos firmes ao término da Fase 0.*
