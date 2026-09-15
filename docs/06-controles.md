# 6. Mapa de controles — de regra escrita a regra executável

Os dois procedimentos declaram **28 controles-chave** (15 em Suprimentos, 13 em Despesas sem
Pedido). Hoje quase todos dependem de alguém lembrar de verificar. Esta tabela mostra como cada um
vira mecanismo de sistema.

A coluna **Tipo** classifica a força do controle:

- **Bloqueio** — o sistema impede a transição. Não há como seguir.
- **Exigência** — o sistema exige um dado ou documento antes de liberar.
- **Alerta** — o sistema avisa e registra, mas não trava (o próprio procedimento manda não travar o
  alerta orçamentário).
- **Evidência** — o sistema registra automaticamente o que hoje se registra à mão.

## 6.1 Suprimentos

| # | Controle | Como vira mecanismo | Tipo |
|---|----------|---------------------|------|
| 1 | Alerta orçamentário registrado e visível até A2/A3 | Campo na solicitação propagado para o QC e carregado na tela de aprovação de A2/A3 | Alerta |
| 2 | Solicitação incompleta ou duplicada não avança | Validação de campos obrigatórios por tipo + detecção de duplicidade (mesmo insumo, obra e janela de tempo) | Bloqueio |
| 3 | Fornecedor só com ficha assinada e documentos; MEI não é contratado | Portal exige upload de todos os documentos aplicáveis + assinatura eletrônica; verificação de MEI no cadastro; status `ativo` só ao final | Bloqueio |
| 4 | Mínimo de 3 propostas ou justificativa no QC | `count(propostas) < 3 AND justificativa vazia` → bloqueia envio para A1 | Bloqueio |
| 5 | Pedido/contrato só segue para aprovação com QC aprovado anexado | Pedido e contrato só podem ser criados a partir de um QC em estado `aprovado`; o QC é anexado automaticamente | Bloqueio |
| 6 | Segregação: quem emite pedido ou elabora contrato não aprova | Regra de SoD no motor de aprovação: `decisor ≠ autor` | Bloqueio |
| 7 | Aumento de valor no pedido e todo aditivo passam por nova aprovação | Gatilho de estado: alteração que eleva o valor total ou criação de aditivo reabre A2/A3 automaticamente | Bloqueio |
| 8 | Serviço só começa com contrato vigente e, na obra, documentação de segurança entregue | Liberação de início é ação do sistema, condicionada a `contrato.status = vigente` + checklist de segurança completo | Bloqueio |
| 9 | Medição limitada ao saldo contratado de cada item | Validação por item: `acumulado + medido ≤ contratado`. Excedente exige aditivo aprovado | Bloqueio |
| 10 | Comprovantes de encargos verificados antes de A4 na obra | Checklist estruturado (relação de funcionários, folha, DCTFWeb, FGTS Digital) com competência conferida; A4 não abre sem ele | Exigência |
| 11 | Nota de material só aceita com confirmação de entrega | Anexo obrigatório do tipo `confirmacao_entrega` para avançar a nota | Exigência |
| 12 | Autenticidade e retenções validadas antes do registro; nota divergente recusada | Estado `em_validacao_fiscal` obrigatório entre recebimento e entrada; recusa é transição com motivo | Bloqueio |
| 13 | Retenções lançadas iguais às validadas pelo fiscal | Os valores do título são **copiados** do registro de validação fiscal, não redigitados; divergência volta ao fiscal | Bloqueio |
| 14 | Dados de pagamento somente do cadastro; beneficiário do boleto = CNPJ do fornecedor | Campos de pagamento do título são somente leitura, preenchidos do cadastro; conferência do CNPJ do beneficiário na leitura da linha digitável | Bloqueio |
| 15 | Pagamento de nota fora do prazo depende de A5 | Cálculo automático de `vencimento − chegada < 15 dias` dispara A5 | Bloqueio |

## 6.2 Despesas sem pedido ou contrato

| # | Controle | Como vira mecanismo | Tipo |
|---|----------|---------------------|------|
| 1 | Somente documentos da lista autorizada | Tipo de despesa vem de lista parametrizada; **não existe campo livre** | Bloqueio |
| 2 | Fatura conferida quanto à unidade consumidora e ao titular | Cadastro de unidades consumidoras por obra/sede; o formulário valida a UC informada contra o cadastro da empresa | Exigência |
| 3 | Envio com <10 dias exige justificativa visível ao aprovador | Cálculo no envio; campo de justificativa obrigatório e destacado na tela de D1 | Bloqueio |
| 4 | Solicitação duplicada bloqueada na entrada | Chave de deduplicação: credor + documento + valor + competência | Bloqueio |
| 5 | Nenhum título sem D1 registrada | A criação do título é disparada **exclusivamente** pelo evento de D1 aprovada | Bloqueio |
| 6 | Segregação: quem registra não aprova | Regra de SoD; reembolso do próprio aprovador sobe um nível na matriz | Bloqueio |
| 7 | Credor cadastrado somente por suprimentos | Permissão por papel; o fluxo aciona suprimentos e aguarda, sem caminho alternativo | Bloqueio |
| 8 | Tipo de documento com anexo obrigatório em título sem NF | Por tipo da lista autorizada, anexos obrigatórios declarados; envio bloqueado sem eles | Bloqueio |
| 9 | Cadastro de título avulso restrito ao financeiro | Permissão no SIENGE (ações 1604/1605/1607/1608) + permissão no sistema | Bloqueio |
| 10 | Cadastro restrito ao usuário de integração e à contingência | Usuário de API dedicado; contingência é estado registrado, não exceção silenciosa | Evidência |
| 11 | Segregação: quem lança o título não confere | SoD na etapa de conferência | Bloqueio |
| 12 | Falha de integração registrada até o reprocessamento | Fila de erro persistente com causa, payload e histórico de tentativas; não fecha sozinha | Evidência |
| 13 | Título conferido contra a solicitação aprovada antes da programação | Tela de conferência lado a lado (aprovado × título lido do SIENGE), com divergências destacadas | Exigência |

## 6.3 Controles que o sistema pode acrescentar

Não estão nos procedimentos, mas surgem naturalmente e valem a conversa:

| Controle | Por quê |
|----------|---------|
| **Alteração de dados bancários em quarentena** | Nova ficha assinada + confirmação por canal independente + janela de espera antes do primeiro pagamento na conta nova. É a defesa padrão contra a fraude do boleto (fratura F3) |
| **Validade de documentos do fornecedor** | Certidões vencidas bloqueiam nova cotação e alertam contratos vigentes. O procedimento exige "certidões atualizadas" mas nada monitora |
| **Saldo de retenção contratual** | Controle por contrato com condição de liberação (termo de encerramento + fim de garantia). Hoje ninguém tem essa visão (fratura F7) |
| **Fornecedor com encargos em atraso** | Se um fornecedor falhou nos comprovantes em medições anteriores, sinalizar antes de nova contratação |
| **Concentração de fornecedor** | % do volume de compras por fornecedor e por obra — indicador de risco e de dependência |
| **Alerta de preço fora da curva** | Com a base de preços acumulada do QC, sinalizar proposta muito acima ou abaixo do histórico do insumo |

## 6.4 O princípio por trás de tudo

O procedimento de suprimentos tem uma frase que deve virar regra de arquitetura:

> *"Não há fluxo de urgência: todas as compras e contratações seguem este procedimento."*

**Não construa um botão de bypass.** Todo sistema de aprovação que ganha um "modo urgente" vê esse
modo virar o caminho padrão em seis meses. Se o caminho normal for lento demais para a realidade, o
problema é o caminho normal — corrija a etapa, não crie o desvio.

A única exceção legítima já está prevista e nomeada: a **contingência** de integração indisponível
com vencimento próximo. E ela é um estado registrado e conferido com a mesma regra, não um atalho.
