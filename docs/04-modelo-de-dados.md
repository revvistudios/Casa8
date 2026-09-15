# 4. Modelo de dados e máquinas de estado

Primeira leitura do modelo, suficiente para discutir e para dimensionar. Não é DDL final.

## 4.1 Os três anéis do modelo

```
┌─ ANEL 1 · Espelho do SIENGE (somente leitura, sincronizado) ────────────┐
│  Empresa/SPE · Obra · CentroDeCusto · Departamento · ContaFinanceira    │
│  Insumo · ItemOrcamento · Fornecedor · CondicaoPagamento                │
└─────────────────────────────────────────────────────────────────────────┘
┌─ ANEL 2 · Parametrização própria (o sistema é dono) ────────────────────┐
│  MatrizAlcada · NivelAlcada · Papel · Usuario · RegraSoD                │
│  DocumentoAutorizado · DeParaSienge · TipoAnexoObrigatorio              │
│  CalendarioPagamento · PoliticaSLA                                      │
└─────────────────────────────────────────────────────────────────────────┘
┌─ ANEL 3 · Processo (o coração) ─────────────────────────────────────────┐
│  Solicitacao · QuadroConcorrencia · Proposta · PedidoCompra · Contrato  │
│  Aditivo · Medicao · NotaFiscal · Titulo · SolicitacaoPagamento         │
│  FichaCadastral · RetencaoContratual                                    │
│  + transversais: Aprovacao · Anexo · Evento · Prazo · Notificacao       │
└─────────────────────────────────────────────────────────────────────────┘
```

## 4.2 Entidades transversais (construídas primeiro)

### `Aprovacao`
O motor que serve A1, A2, A3, A4, A5 e D1. Uma única implementação.

```
Aprovacao
  id
  codigo              A1 | A2 | A3 | A4 | A5 | D1
  objeto_tipo         qc | pedido | contrato | aditivo | medicao | titulo | solicitacao_pagamento
  objeto_id
  modo                serial | paralelo | cadeia_ordenada
  status              pendente | em_andamento | aprovada | reprovada | cancelada
  valor_referencia    base para a alçada (nulo em A1 e A4, que não têm alçada)
  criada_em, concluida_em

NivelAprovacao
  aprovacao_id
  ordem               1, 2, 3 — A1 de obra: construtora, gerenciadora, incorporadora
  papel_esperado      ou usuario_id direto
  usuario_decisor     quem de fato decidiu
  decisao             pendente | aprovado | reprovado
  motivo              OBRIGATÓRIO quando reprovado
  decidido_em
  delegado_de         para férias/ausência, mantendo rastro
```

Regras de segregação de funções implementadas aqui, como predicados verificados antes de atribuir um
nível: `decisor ≠ autor do objeto`, `decisor ≠ quem mediu`, `decisor ≠ quem lançou o título`,
`se beneficiário do reembolso == decisor então sobe um nível`.

### `Anexo`
```
Anexo
  id, objeto_tipo, objeto_id
  tipo_documento      cartao_cnpj | contrato_social | proposta | boletim_medicao |
                      dctfweb | fgts | xml_nf | pdf_nf | guia | comprovante | ...
  nome_arquivo, mime, tamanho, hash_sha256
  storage_key
  valido_ate          NULO quando não vence — preenchido em certidões
  enviado_por, enviado_em
  sienge_anexo_id     preenchido quando replicado ao SIENGE
```

`hash_sha256` serve para detectar reenvio do mesmo documento e para provar integridade em auditoria.

### `Evento` (auditoria)
```
Evento
  id, ocorrido_em, ator_id, ator_tipo (usuario | sistema | integracao)
  objeto_tipo, objeto_id
  acao                criado | alterado | aprovado | reprovado | enviado | ...
  antes, depois       JSONB
  ip, user_agent
```
Somente inserção. Sem `UPDATE`, sem `DELETE`.

### `Prazo`
```
Prazo
  objeto_tipo, objeto_id
  tipo                nf_ate_dia_25 | nf_15_dias | validacao_fiscal_2du |
                      despesa_10_dias | encargos_medicao
  inicio, vence_em
  status              no_prazo | em_risco | estourado | cumprido
  alerta_em[]         marcos de notificação
```

## 4.3 Cotação — o modelo que substitui a planilha

O XLSX já define a lógica com precisão; ela vira modelo relacional quase diretamente.

```
QuadroConcorrencia
  id, numero, revisao
  solicitacao_id, escopo
  empresa_id, obra_id, centro_custo_id
  tipo                material | servico
  aplicacao           obra | administrativo
  alerta_orcamentario boolean (veio da solicitação)
  orcamento_atualizado  quando informado, sobrepõe o total orçado dos itens
  criterio_escolha, descricao_criterio
  fornecedor_escolhido_id, valor_escolhido
  justificativa       obrigatória nos três gatilhos abaixo
  status              rascunho | em_cotacao | em_negociacao | em_aprovacao |
                      aprovado | reprovado | cancelado

ItemQC
  qc_id, ordem, centro_custo_id, insumo_id, descricao, unidade, quantidade
  preco_unitario_orcado

Proposta
  qc_id, fornecedor_id, numero_proposta, data_proposta, contato
  frete_fob, impostos_nao_inclusos, outros_custos_descontos
  total_equalizado        calculado: subtotal + frete + impostos + outros
  valor_final             última rodada de negociação, ou o equalizado
  classificacao           posição por valor final
  condicoes               JSONB: pagamento, prazos, validade, CIF/FOB, garantia,
                          retenção 5% aceita, composição MAT+MO, LC 116, local
  doc_trabalhista         JSONB (obra): EPI, manutenção, ART/RRT, PCMSO, PGR, checklist

ItemProposta
  proposta_id, item_qc_id, preco_unitario     total = quantidade × preço

RodadaNegociacao
  proposta_id, rodada (1 | 2), total, registrada_em, registrada_por
```

**Os três alertas de justificativa obrigatória** (hoje fórmulas nas células I79, I84 e K85) viram
regras avaliadas antes de enviar para A1:

1. `count(propostas) < 3`
2. `fornecedor_escolhido.valor_final > min(valor_final de todas)`
3. `valor_escolhido > orcamento_considerado`

Se qualquer uma for verdadeira e `justificativa` estiver vazia → bloqueia o envio.

**Ganho que a planilha não dá:** `ItemProposta` acumulado ao longo do tempo é uma base de preços por
insumo, fornecedor, obra e data. É o ativo de dado mais valioso que o sistema vai gerar.

## 4.4 Máquinas de estado

### Solicitação de compra
```
rascunho → registrada → em_analise →┬→ devolvida → (ajusta) → registrada
                                    ├→ cancelada
                                    └→ aceita → em_cotacao → atendida
```

### Quadro de concorrência
```
rascunho → em_cotacao → em_negociacao → em_aprovacao(A1)
                                             │
              ┌──────────────────────────────┼─────────────┐
              ▼                              ▼             ▼
        reprovado → revisao(+1)         aprovado      cancelado
                                             │
                          ┌──────────────────┴────────┐
                          ▼                           ▼
                  pedido de compra                contrato
```
Na obra, `em_aprovacao` percorre os três níveis **na ordem**. Reprovação em qualquer nível devolve o
QC inteiro a suprimentos e incrementa a revisão.

### Pedido de compra
```
emitido → em_aprovacao(A2) →┬→ reprovado → ajustado → em_aprovacao(A2)
                            │            → cancelado
                            └→ aprovado → enviado_fornecedor → em_atendimento
                                                                    │
                                     alteração que aumenta valor ────┘
                                              ↓
                                       em_aprovacao(A2)   [nova aprovação]
```
Troca de fornecedor **não** é transição: cancela e exige novo QC.

### Contrato
```
minuta →(fora do padrão)→ revisao_juridica → assinatura → cadastrado → em_aprovacao(A3)
                                                                          │
                                        ┌─────────────────────────────────┤
                                        ▼                                 ▼
                                 vigente ←── aditivo(A3)            reprovado
                                        │                          → aditivo ou distrato
                                  encerrado → garantia → retencao_liberada
```
O contrato é assinado **antes** do cadastro — por isso a reprovação em A3 não desfaz o contrato, ela
gera aditivo ou distrato. Isso é uma peculiaridade real do processo e o modelo precisa respeitá-la.

### Medição
```
em_apuracao → documentada → registrada → em_aprovacao(A4)
                   ↑                          │
                   └── devolvida ←────────────┤
                                              ▼
                                          aprovada → nota_liberada
```
Guarda: `valor_acumulado + valor_medido ≤ saldo_contratado` **por item**. Na obra, guarda adicional:
todos os comprovantes de encargos da competência presentes e verificados.

### Nota fiscal
```
recebida → conferida_responsavel → em_validacao_fiscal
                                          │
                        ┌─────────────────┼──────────────┐
                        ▼                 ▼              ▼
                    recusada         validada      pendente_cno
                   (fornecedor        │
                    reemite)          ▼
                              entrada_registrada → titulo_gerado
```

### Título
```
gerado → em_conferencia →┬→ divergente → corrigido → em_conferencia
                         └→ conferido →┬→ liberado_programacao
                                       └→ aguarda_A5 → prorrogado → liberado
```

### Solicitação de pagamento (despesa sem pedido)
```
rascunho → registrada →(valida prazo/duplicidade)→ em_aprovacao(D1)
                                                        │
                          ┌─────────────────────────────┼────────────┐
                          ▼                             ▼            ▼
                      reprovada                     aprovada     cancelada
                          │                             │
              ajusta e reenvia                 credor_pendente? ──► suprimentos
                                                        │
                                             titulo_em_criacao
                                                        │
                                        ┌───────────────┼───────────────┐
                                        ▼               ▼               ▼
                                  erro_integracao   titulo_criado   contingencia
                                        │               │               │
                                   reprocessa           └──► em_conferencia ──► conferido
```

## 4.5 Decisões de modelagem a tomar na Onda 0

1. **O pedido e o contrato nascem aqui ou no SIENGE?** Se o SIENGE já resolve A2/A3 bem, o sistema
   pode só anexar o QC e espelhar o status — escopo bem menor. Decisão de custo/benefício.
2. **Granularidade do espelho.** Espelhar o orçamento inteiro da obra ou consultar sob demanda?
3. **Reembolso de colaborador** é `Fornecedor` no SIENGE (credor CPF) ou entidade própria? Impacta
   LGPD e o portal.
4. **Multi-tenant por SPE ou coluna `empresa_id`?** A recomendação é coluna com *row-level security*
   — são dezenas de SPEs, não clientes distintos.
5. **Versionamento do QC**: revisão como novo registro ou como histórico do mesmo? (Recomendação:
   novo registro com `qc_origem_id`, porque o QC aprovado é documento anexado ao pedido.)
