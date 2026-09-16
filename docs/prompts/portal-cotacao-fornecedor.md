# Prompt · Portal de cotação do fornecedor

Prompt pronto para colar no **Claude Design** e gerar as telas do portal onde o
fornecedor vê o que está sendo cotado e devolve a proposta.

**Por que estas telas primeiro:** a cotação é a etapa que hoje vaza inteiramente
para planilha e e-mail (fratura F1 do diagnóstico), e o portal do fornecedor é o
que tira o e-mail do caminho crítico (F2). É também a parte do sistema que nenhum
dos cenários A ou B da empresa entrega bem.

**Dados usados:** os itens e valores vêm da aba `Exemplo` do
`Quadro_de_Concorrencia_CASA8.xlsx` — o QC-2026-015 de concreto usinado. Usar
dados reais evita que o Claude Design invente conteúdo genérico, e as telas já
saem prontas para mostrar a suprimentos.

**Decisão de produto embutida:** o campo de observação/divergência **por item**.
A planilha só tem um "Atende à especificação técnica? Sim / Parcial / Não" global
— no exemplo, a Gama está marcada como "Parcial" e não há onde registrar em qual
item. Sem isso, a premissa de que todos cotam a mesma especificação quebra em
silêncio, e a comparação do QC deixa de valer.

---

```text
Crie um canvas de design para o portal de cotação de fornecedores do Grupo CASA8,
uma incorporadora/construtora brasileira. É a tela que o fornecedor recebe por
e-mail para ver o que está sendo cotado e devolver sua proposta.

## CONTEXTO DO PROCESSO

Suprimentos envia a mesma especificação para no mínimo 3 fornecedores cadastrados.
Cada um responde com preço unitário por item e as condições comerciais. Depois,
suprimentos equaliza as propostas (soma frete FOB, impostos não inclusos, aplica
descontos), negocia em até 2 rodadas e monta o Quadro de Concorrência.

O fornecedor é externo: não tem conta corporativa, acessa por link do e-mail,
sem senha. Pode ser o comercial abrindo no celular.

## REGRAS QUE MOLDAM A INTERFACE

1. O fornecedor NUNCA vê preço de concorrente nem o custo orçado da obra. Cotação
   fechada — isso é inegociável e deve ficar visualmente evidente ("sua proposta
   é confidencial").
2. Os itens (descrição, unidade, quantidade) são SOMENTE LEITURA. Todos cotam a
   mesma especificação; se cada um alterar, a comparação não vale.
3. Upload da proposta formal assinada é obrigatório — nenhum fornecedor é
   escolhido sem proposta formal arquivada.
4. Prazo de resposta sempre visível, com contagem regressiva.
5. Deve dar para salvar rascunho e voltar depois.
6. Depois de enviado, trava — mas suprimentos pode reabrir numa rodada de negociação.
7. Português do Brasil. Valores em R$ com vírgula decimal e milhar com ponto.
   Números alinhados à direita com tabular-nums.

## ARTBOARDS

### 1 — E-mail do convite
Assunto: "Cotação CASA8 · QC-2026-015 · resposta até 22/09". Remetente Suprimentos.
Corpo curto e objetivo: o que está sendo cotado, para qual obra, o prazo, e um
botão grande "Responder a cotação". Listar os 3 itens em resumo (sem campo de
preço — o preenchimento é no portal). Rodapé com contato do comprador.
Mostre como card de e-mail realista, com cabeçalho De/Para/Assunto.

### 2 — Itens e preços (tela principal)
Cabeçalho fixo com: QC-2026-015 · Revisão 00 · prazo "faltam 4 dias" ·
"Concreteira Beta · respondendo para a SPE do empreendimento".
Bloco de identificação (somente leitura, compacto): escopo, obra/centro de custo,
CNO, local de entrega, solicitação SC 1520.

Tabela de itens — colunas: Nº, Insumo, Descrição, Unid., Qtd. (todas somente
leitura) e depois "Preço unit. (R$)" (campo editável) e "Total (R$)" (calculado):

  1 · INS-0101 · Concreto usinado fck 30 MPa, slump 10±2 · m³ · 120
  2 · INS-0145 · Serviço de bombeamento de concreto · m³ · 120
  3 · INS-0150 · Taxa mínima de bomba por mobilização · un · 3

Mostre a tela PREENCHIDA com: 479,50 · 38,00 · 700,00
Totais por item: 57.540,00 · 4.560,00 · 2.100,00 — Subtotal 64.200,00

Cada item tem também um campo discreto de observação/divergência, para o caso de
o fornecedor não atender exatamente à especificação daquele item.

Abaixo, bloco "Equalização" — explique em uma linha por que é pedido:
  Frete: CIF (incluso) / FOB (por conta da obra) → selecionado CIF
  Impostos inclusos no preço? Sim / Não → Sim
  Outros custos (+) ou descontos (−): −1.500,00 (rotulado "Desconto comercial")

Painel de total fixo no rodapé, sempre visível: Subtotal 64.200,00 ·
Desconto −1.500,00 · TOTAL DA PROPOSTA 62.700,00

### 3 — Condições e documentos
Formulário em grupos:
  Comerciais: forma e condição de pagamento (21 dias), prazo de entrega
  (conforme programação), prazo de início, prazo de conclusão, validade da
  proposta (10 dias), garantia (laudo de resistência)
  Técnicas: atende à especificação técnica? Sim / Parcial / Não → Sim.
  Composição de custo (MAT + MO) → Material. Limite de faturamento direto.
  Para serviço: código LC 116 e local de execução. Retenção contratual de 5%
  aceita? (só aparece em serviço de obra)
  Documentação de obra (checkboxes): fornecimento de EPIs, manutenção preventiva
  e corretiva de equipamentos, ART ou RRT, PCMSO e PGR, checklist de documentações
  Anexos: área de upload com "Proposta formal assinada" marcada como obrigatória
  (mostre um PDF já anexado) e espaço para documentos complementares
  Campo de observações

### 4 — Revisão antes de enviar
Resumo de tudo em leitura, com link "editar" por bloco. Aviso claro: "Depois de
enviar, sua proposta não poderá ser alterada, exceto se a CASA8 abrir uma rodada
de negociação." Total em destaque: R$ 62.700,00. Botão primário "Enviar proposta".

### 5 — Proposta enviada
Confirmação com número de protocolo, data e hora, total enviado, link para baixar
o comprovante em PDF. Explicar o próximo passo em uma frase: a CASA8 vai comparar
as propostas e pode abrir uma rodada de negociação. Nada de confete.

### 6 — Rodada de negociação
Mesma identidade visual, com tarja no topo: "A CASA8 solicitou uma revisão da sua
proposta · prazo até 25/09". Mostra a proposta anterior (62.700,00) em leitura e
um campo para o novo total, com histórico das rodadas ao lado:
  Proposta original 62.700,00 · 1ª negociação 62.100,00 · 2ª negociação (em aberto)
Campo de justificativa opcional. Botão "Enviar nova proposta" e link secundário
"Manter o valor anterior".

### 7 — Recusar a cotação
Estado alternativo, acessível desde a tela principal por link discreto
"Não vou participar desta cotação". Motivo em lista (sem capacidade no prazo,
fora do escopo de atuação, material indisponível, outro) + campo livre.
Mantenha leve — recusar rápido é melhor para os dois lados do que silêncio.

### 8 — Celular (tela 2 em ~390px)
A tabela de itens vira cards empilhados, um por item, com o preço unitário como
campo grande e fácil de tocar. O painel de total continua fixo no rodapé.

## DIREÇÃO VISUAL

Papel técnico frio, sóbrio, de engenharia — sem cara de startup.

  fundo      #EDF0F1     superfície #FFFFFF     superfície 2 #F4F7F8
  tinta      #15181B     secundária #4E5A63     terciária    #7C8892
  linhas     #D3DADE
  laranja    #DD5420  — reservado para AÇÃO e DECISÃO (botão primário, prazo,
                        campos obrigatórios). Vem dos diagramas de processo da
                        própria empresa, onde marca aprovações. Use com parcimônia.
  teal       #0E6B62  — confirmado, enviado, valor consolidado

Tipografia: Archivo (títulos, rótulos e UI), IBM Plex Mono (códigos de insumo,
números de QC, quantidades e valores — tabular). No corpo do e-mail pode usar
Source Serif 4, que é a face dos documentos institucionais da empresa.

Nada de cards arredondados por toda parte: bordas de 2px de raio, hairlines,
hierarquia por peso e espaçamento. Tabelas densas e legíveis, como planilha bem
feita — é o que essas pessoas usam o dia inteiro. Funciona em tema claro e escuro.
```

---

## Variações úteis

**Para gerar a contraparte de suprimentos**, troque a seção de artboards por:
painel de acompanhamento da cotação (quem respondeu, quem não, prazo por
fornecedor), tela de equalização lado a lado com as 3 propostas, tela de abertura
de rodada de negociação, e a tela de escolha com os três testes de justificativa
obrigatória (menos de 3 propostas · escolha diferente do menor valor · acima do
orçamento).

**Para um QC de serviço em vez de material**, troque os itens por serviços com
planilha de quantidades, e ative os campos que só aparecem em serviço: retenção
contratual de 5%, código LC 116, local de execução, cessão de mão de obra e a
documentação de segurança do trabalho (ART/RRT, PCMSO, PGR).

## Antes de levar para a empresa

Confirme com suprimentos:

- A negociação é registrada só como **total** (é como a planilha faz) ou o
  fornecedor deve revisar preço por item? Isso muda a tela 6.
- Quantas rodadas de negociação na prática — a planilha prevê 2.
- O fornecedor pode ver o QC depois de decidido, ou só o resultado?
- Quem é o remetente do e-mail: o comprador nominalmente ou um endereço da área?
