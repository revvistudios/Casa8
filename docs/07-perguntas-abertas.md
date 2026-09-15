# 7. Perguntas a levar para a empresa

Organizadas por urgência. As da §7.1 bloqueiam a estimativa; as demais podem ser respondidas ao
longo da Onda 0.

## 7.1 Bloqueantes — responder antes de estimar

| # | Pergunta | Para quem | Por que bloqueia |
|---|----------|-----------|------------------|
| 1 | **O SIENGE está em nuvem (DC) ou em servidor local?** | TI / Softplan | On-premise = sem API. Muda o desenho inteiro (`docs/05`) |
| 2 | O contrato inclui uso de API? Há custo ou limite? | Controladoria / TI | Custo e viabilidade |
| 3 | Existe ambiente de homologação do SIENGE? | TI | Sem homologação, testar em produção é inaceitável |
| 4 | **Existe endpoint de criação de título avulso sem NF-e?** | Spike técnico | É a Onda 1 inteira |
| 5 | Qual o volume real: compras/mês, contratos ativos, medições/mês, notas/mês? | Suprimentos / Financeiro | Só se sabe ~40 títulos/mês de despesas sem pedido. Dimensiona tudo |
| 6 | Qual o plano de licenciamento do M365? Todos têm conta? | TI | Define se o SSO cobre todos ou se precisa de autenticação própria |
| 7 | Quantas SPEs ativas, quantas obras simultâneas, quantos usuários por papel? | Controladoria | Dimensiona multi-empresa e acesso |
| 8 | Este projeto é um sistema próprio ou a empresa ainda decide entre os cenários A e B? | Diretoria | Define se a conversa é técnica ou de decisão |

## 7.2 Escopo e processo

**Decisões que a própria CASA8 já listou como abertas** (§13 do Procedimento de Despesas):

| # | Ponto em aberto | O que ainda falta decidir |
|---|-----------------|---------------------------|
| 1 | Tributos abrangidos | Entram tributos próprios, retenções de fornecedores e IPTU? (Encargos de folha ficam fora) |
| 2 | ITBI e taxas de prefeitura | Quais casos e quem emite cada guia |
| 3 | Contas de consumo | Controle de unidades consumidoras por obra/sede e troca de titularidade na entrega |
| 4 | Reembolso | Confirmar política; adiantamento de viagem e cartão corporativo entram? |
| 5 | Lançamento do título (cenário A) | Centralizado no financeiro ou distribuído, como em suprimentos? |
| 6 | Manutenção da lista e da matriz | Responsável (proposta: controladoria) |
| 7 | Verificação orçamentária | Aplica o alerta sem trava? Em qual etapa? |
| 8 | Documentos do cadastro de credor | O que suprimentos exige de órgão público, concessionária e colaborador |
| 9 | Acesso dos usuários | Solicitantes de obra, colaboradores e aprovadores têm conta M365? |
| 10 | Fornecedor do cenário B | Se a empresa optar por BPM, este projeto muda de natureza |

**Perguntas adicionais do projeto:**

| # | Pergunta | Impacto |
|---|----------|---------|
| 11 | Pedido e contrato nascem no sistema novo ou continuam no SIENGE? | Reduz ou aumenta muito a Onda 3 |
| 12 | Gerenciadora e incorporadora aceitam aprovar em sistema da CASA8? | A1 e A4 dependem deles. **Conversar cedo** |
| 13 | Quantos fornecedores ativos? Qual o perfil de maturidade digital? | Adoção do portal |
| 14 | Quem é o dono do processo do lado do negócio? | Sem um interlocutor com autoridade, o projeto trava em decisões |
| 15 | Há auditoria externa ou exigência de compliance específica? | Define o rigor da trilha de auditoria |
| 16 | Existe assinatura eletrônica contratada? Qual? | Ficha cadastral e contrato dependem |
| 17 | A matriz de alçadas vigente está documentada e atualizada? | É o insumo do motor de aprovação |
| 18 | Como são tratadas férias e ausências de aprovadores hoje? | Define o modelo de delegação |
| 19 | O orçamento da obra é acessível por API ou só por relatório? | Define o alerta orçamentário |
| 20 | Qual o apetite para mudar o processo, ou o sistema deve replicá-lo como está? | Muda profundamente a abordagem |

## 7.3 Perguntas sobre o seu papel

Valem uma conversa franca antes de assinar:

1. **Você é o único desenvolvedor?** O procedimento de despesas afirma textualmente que *"não há
   desenvolvedor interno"*. Se o plano é um projeto de 12 meses com uma pessoa, o risco R2 é real e
   precisa estar na proposta — não escondido nela.
2. **Qual o horizonte de sustentação?** Construir é metade; manter é a outra. Combine desde já como
   fica a sustentação depois da última onda.
3. **Há orçamento por onda ou aprovação única?** Recomendação forte: **contratar por onda**. Protege
   os dois lados e força a entrega de valor a cada ciclo.
4. **De quem é a decisão A vs. B vs. C?** Se a diretoria já se inclinou para o cenário A, entrar com
   uma proposta de sistema próprio sem reconhecer isso queima a conversa. A abordagem que funciona é
   a da §3.5: *"o cenário A resolve o que vocês estão decidindo agora; a proposta C só se justifica
   se o objetivo for o processo inteiro."*
5. **Quem responde pelas regras de negócio?** Você vai precisar de alguém da controladoria
   disponível toda semana. Sem isso, cada dúvida vira uma semana parada.

## 7.4 O que pedir para ver, se possível

- [ ] A **matriz de alçadas** vigente de cada departamento
- [ ] A **lista de documentos autorizados** (referenciada, mas não anexada)
- [ ] A **política de reembolso** (marcada como "a confirmar" no procedimento)
- [ ] O **modelo padrão de contrato** de prestação de serviço
- [ ] Os três arquivos **.drawio** e os **.bpmn** dos fluxos
- [ ] Acesso de leitura ao SIENGE, ou uma sessão de tela compartilhada percorrendo o fluxo real
- [ ] Uma amostra de 5 a 10 **QCs preenchidos de verdade** — nada revela mais sobre o processo real
      que ver como as exceções são de fato tratadas
- [ ] Os relatórios que a controladoria e a diretoria já consomem hoje
