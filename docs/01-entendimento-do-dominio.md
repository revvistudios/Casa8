# 1. Entendimento do domínio — Grupo CASA8

> Base: `Procedimento_Suprimentos_CASA8.docx` (v1.0), `Procedimento_Despesas_sem_Pedido_CASA8.docx` (v0.1),
> `Quadro_de_Concorrencia_CASA8.xlsx`, `Ficha_Cadastral_Fornecedor_CASA8.docx` e os três fluxos em raias.

## 1.1 O que a empresa é, do ponto de vista de processo

A CASA8 é um grupo de incorporação/construção que opera por **SPEs** (uma sociedade por
empreendimento). Isso tem três consequências que moldam qualquer sistema:

1. **Multi-empresa é requisito, não configuração.** Todo documento — solicitação, QC, pedido,
   contrato, medição, nota, título — pertence a uma SPE específica, com CNPJ próprio, endereço de
   faturamento próprio e **CNO** próprio. Errar a SPE não é erro de digitação: é nota fiscal
   emitida contra o CNPJ errado, com retenção previdenciária no CNO errado.
2. **Há terceiros dentro do fluxo de aprovação.** Gerenciadora de obra e incorporadora aprovam
   (A1 e A4). Não são funcionários. Precisam de acesso ao sistema sem estarem no diretório interno.
3. **Obra e administrativo têm regras diferentes na mesma etapa.** A mesma etapa (cotação, medição)
   muda de aprovador, de documento obrigatório e de retenção conforme a aplicação. Isso é
   parametrização, não dois processos paralelos.

O **SIENGE** é o ERP. Ele é a fonte de verdade de: cadastros (SPEs, obras, centros de custo, plano
financeiro, insumos, fornecedores), pedidos, contratos, medições, notas e contas a pagar. Tudo o que
este projeto construir precisa conviver com isso, não substituí-lo.

## 1.2 Os dois processos existentes

### Processo A — Suprimentos (com pedido ou contrato)

Cobre compra de material e contratação de serviço, da necessidade até o título a pagar.

```
Solicitação ──► Análise de suprimentos ──► Cotação (QC) ──► [A1]
                                                             │
                         ┌───────────────────────────────────┴──────────────────┐
                         │ MATERIAL                                   SERVIÇO   │
                         ▼                                                      ▼
                  Pedido de compra ──► [A2]                        Contrato ──► [A3]
                         │                                                      │
                         │                                              Medição ──► [A4]
                         │                                                      │
                         └──────────────► Nota fiscal do fornecedor ◄───────────┘
                                                   │
                                    Recepção e validação fiscal (fiscal, 2 d.ú.)
                                                   │
                                    Entrada da nota e geração do título
                                                   │
                                    Conferência do título (financeiro) ──► [A5 se fora do prazo]
                                                   │
                                          Programação de pagamento (fora do escopo)
```

Divide-se após a cotação em dois trilhos, reúne-se na nota fiscal, divide-se de novo na geração do
título (material entra pelo pedido; serviço entra pelo caminho da medição) e termina na conferência.

### Processo B — Custos e despesas sem pedido nem contrato

Cobre o que não passa por suprimentos: tributos, ITBI, taxas de prefeitura, contas de consumo e
reembolso de colaborador. Hoje **não existe ferramenta definida** — o procedimento está em versão
0.1 e apresenta dois cenários concorrentes (Microsoft 365 vs. plataforma BPM com API) justamente
para a empresa escolher.

```
4 entradas (fatura de consumo | taxa/ITBI | tributo | reembolso)
        │
        ▼
Solicitação de pagamento ──► verificação automática do prazo de 10 dias ──► [D1]
                                                                            │
                                                     Credor existe no SIENGE? ──não──► Suprimentos cadastra
                                                                            │
                                                            Título no SIENGE (manual ou via API)
                                                                            │
                                                             Conferência do título ──► programação
```

Volume declarado: **~40 títulos por mês**, hoje ocupando duas pessoas do financeiro no cenário
manual (uma lança, outra confere).

## 1.3 O quadro de aprovações consolidado

Este é o coração do sistema. Hoje as aprovações estão em três lugares diferentes.

| Cód. | Etapa | Quem aprova | Natureza | Onde é registrada hoje |
|------|-------|-------------|----------|------------------------|
| **A1** | Cotação (QC) | **Obra:** construtora → gerenciadora → incorporadora (cadeia de 3, nesta ordem). **Adm.:** diretor da área | Critério e escolha do fornecedor. **Sem alçada de valor** | **Fora do SIENGE** — assinatura no XLSX ou e-mail anexado |
| **A2** | Pedido de compra | Matriz de alçadas do departamento | Alçada por valor. Reaprova se o valor sobe | SIENGE |
| **A3** | Contrato e **todo** aditivo | Matriz de alçadas do departamento | Alçada por valor | SIENGE |
| **A4** | Medição | **Obra:** gerenciadora. **Adm.:** diretor da área | Técnica, sem alçada. Na obra exige comprovantes de encargos | SIENGE |
| **A5** | Pagamento fora do prazo | Financeiro | Nota chegou a <15 dias do vencimento; o financeiro define a prorrogação | SIENGE |
| **D1** | Despesa sem pedido | Matriz de alçadas do departamento, 1+ níveis | Alçada por valor | **Não existe ainda** |

**Observação crítica:** a matriz de alçadas é uma só (por departamento), mas seria parametrizada em
dois lugares — no SIENGE para A2/A3 e na ferramenta nova para D1. Duas cópias da mesma regra é uma
divergência esperando para acontecer.

## 1.4 Os prazos e o calendário do processo

| Prazo | Regra | Consequência de violar |
|-------|-------|------------------------|
| NF emitida e enviada **até o dia 25** | Permite conferir e recolher retenções no prazo legal | Retenção recolhida fora do prazo |
| NF chega com **≥15 dias** do vencimento | Abaixo disso depende de **A5** | Aprovação extra + renegociação com o fornecedor |
| Validação fiscal em **≤2 dias úteis** | Contados do recebimento pelo fiscal | Consome a folga dos 15 dias |
| Comprovantes de encargos **a cada medição** | Obrigatórios para A4 na obra | Medição não aprovada |
| Pagamentos nos dias **5, 14, 21 e 28** | Exceto impostos | Título perde a janela e espera a próxima |
| Despesa sem pedido: enviar com **≥10 dias** do vencimento | Abaixo disso, justificativa obrigatória visível ao aprovador | Multa/juros por atraso |

Todos esses prazos hoje são controlados por atenção humana. Nenhum tem alerta automático nem
contagem regressiva. É a origem provável da maior parte dos retrabalhos.

## 1.5 Entidades do domínio (primeira leitura)

**Cadastros espelhados do SIENGE (o sistema lê, não é dono):**
Empresa/SPE · Obra · CentroDeCusto · Departamento · PlanoFinanceiro (conta) · Insumo ·
Orçamento da obra · Fornecedor/Credor · CondiçãoDePagamento

**Cadastros do sistema novo (o sistema é dono):**
MatrizDeAlçadas (departamento × faixa de valor × nível × aprovador) ·
ListaDeDocumentosAutorizados (para despesas sem pedido) ·
De-para lista autorizada ↔ tipo de documento e conta financeira do SIENGE ·
PolíticaDeReembolso · Papéis e permissões · Calendário de pagamento

**Documentos de processo:**
Solicitação · QuadroDeConcorrência (+ Proposta, ItemDeProposta, Equalização, RodadaDeNegociação,
Escolha, Justificativa) · PedidoDeCompra · Contrato · Aditivo · Medição (+ Atesto, BoletimDeMedição,
ComprovanteDeEncargo) · NotaFiscal (+ ValidaçãoFiscal, Retenções) · Título · RetençãoContratual ·
SolicitaçãoDePagamento (despesa sem pedido) · FichaCadastralDeFornecedor

**Transversais:**
Aprovação (código, nível, decisão, motivo, data, aprovador) · Anexo · EventoDeAuditoria · SLA/Prazo ·
Notificação

## 1.6 Regras de negócio que o sistema precisa fazer valer

Recolhidas dos dois procedimentos, do QC e da ficha cadastral. São regras duras — não sugestões.

**Segregação de funções (SoD)**
- Quem registra a solicitação de despesa não a aprova (D1).
- Quem emite o pedido não o aprova (A2).
- Quem elabora o contrato não o aprova (A3).
- Quem mede não aprova a medição (A4).
- Quem lança o título não o confere (cenário manual de despesas).
- Reembolso do próprio aprovador sobe para o nível seguinte da matriz.

**Cotação**
- Mínimo de 3 propostas formais, todas sobre a **mesma especificação**.
- Só cotam fornecedores já cadastrados no SIENGE.
- Justificativa obrigatória em três gatilhos: <3 propostas, escolha ≠ menor valor final, valor acima
  do orçamento. (O XLSX já calcula esses três alertas automaticamente — é lógica pronta para migrar.)
- Na obra, A1 só se completa com as três aprovações **na ordem**: construtora → gerenciadora →
  incorporadora. Reprovação em qualquer nível devolve o QC a suprimentos.
- Nenhum pedido ou contrato vai para A2/A3 sem o QC aprovado anexado.
- Troca de fornecedor não é alteração: exige nova cotação e novo QC.

**Fornecedor**
- Cadastro só conclui com ficha **assinada** + todos os documentos obrigatórios aplicáveis.
- **MEI não é contratado.**
- Conta bancária obrigatoriamente de titularidade do **CNPJ do fornecedor**. Sem negociação de
  títulos com terceiros.
- **Alteração de dados bancários exige nova ficha cadastral assinada.**
- Dados de pagamento do título vêm exclusivamente do cadastro — nunca do corpo da nota ou de e-mail.

**Pedido e contrato**
- Nenhum pedido chega ao fornecedor antes de aprovado no SIENGE.
- Aumento de valor no pedido → nova A2.
- Todo aditivo (valor, prazo ou escopo) → A3.
- Fugiu do modelo padrão de contrato → revisão do jurídico obrigatória.
- Serviço não começa sem contrato vigente e, na obra, sem a documentação de segurança entregue.

**Medição**
- Não ultrapassa o saldo contratado **por item**; excedente exige aditivo aprovado em A3.
- Boletim assinado pelo prestador.
- Na obra: relação de funcionários, folha, DCTFWeb (INSS) e FGTS Digital verificados **antes** de A4.
- Nota é emitida só depois da medição aprovada e **no mesmo valor**.

**Nota fiscal**
- Autenticidade consultada na SEFAZ ou na prefeitura **antes** do registro.
- Nota divergente é **recusada**, nunca corrigida internamente.
- Material só é aceito com pedido aprovado + confirmação de entrega (canhoto/DANFE assinado).
- Serviço só com medição ou atesto aprovado, no mesmo valor.
- Retenção de INSS em obra sem CNO → nota recusada.
- Retenções lançadas no título = exatamente as validadas pelo fiscal.

**Título**
- Nota sem vínculo a pedido ou medição não gera título.
- Documentos ficam anexados no SIENGE, **não em pastas ou e-mails pessoais**.
- Líquido = bruto − retenções tributárias − 5% de retenção contratual (obra).
- Boleto: beneficiário com o CNPJ do próprio fornecedor.
- Despesa sem pedido: nenhum título sem D1 registrada; cadastro de título avulso restrito.

**Regra geral que define a cultura do processo**
> "Não há fluxo de urgência: todas as compras e contratações seguem este procedimento."

Isso é importante para o desenho: **não construa um botão de bypass.** Construa velocidade no
caminho normal.
