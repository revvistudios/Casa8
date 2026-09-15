# 2. Diagnóstico — onde está a dor e onde está o valor

Os procedimentos da CASA8 são **bem escritos**. Isso é raro e é uma vantagem: as regras já estão
explícitas, os controles já estão nomeados, os papéis já estão definidos. O problema não é o
processo — é que ele está distribuído entre SIENGE, Excel, Word, e-mail e atenção humana.

Este diagnóstico separa o que dói do que apenas incomoda, porque isso define a ordem de construção.

## 2.1 As nove fraturas do processo atual

### F1 · A cotação vive fora de qualquer sistema
O QC é uma planilha Excel com fórmulas. A aprovação A1 — que na obra envolve **três empresas
diferentes** em cadeia ordenada — é registrada "por assinatura no QC ou e-mail anexado".

Consequências: não há trilha de auditoria confiável, não dá para saber onde um QC está parado, não
há controle de revisão (o próprio modelo tem campo "Revisão" preenchido à mão), e cada cotação
morre no arquivo — **o histórico de preços por insumo e fornecedor não se acumula em lugar nenhum**.

> Este é o maior valor não capturado do processo inteiro. A CASA8 cota, negocia em até duas rodadas,
> equaliza e escolhe — e joga fora o dado. Um sistema que guarde isso entrega, no segundo ano, uma
> base de preços própria que vale mais que a automação do fluxo.

### F2 · O cadastro de fornecedor é um formulário Word por e-mail
Ficha em Word preenchida pelo fornecedor, assinada, devolvida por e-mail com 4 a 6 documentos
anexos, conferida manualmente por suprimentos contra o cartão CNPJ.

Consequências: ciclo lento com fornecedor novo bloqueando cotação; conferência sem rastro; nenhum
controle de **validade** de certidões (o procedimento exige "certidões de regularidade atualizadas"
no contrato, mas nada avisa quando vencem); risco de a ficha assinada e os documentos se perderem
em caixas de e-mail pessoais.

### F3 · Alteração de dados bancários é o ponto de fraude clássico
O procedimento já acertou a regra ("alteração exige nova ficha assinada", "conta de titularidade do
CNPJ", "não aceitamos negociação de títulos com terceiros"). Mas a regra é aplicada por uma pessoa
lendo um e-mail. É exatamente o vetor da fraude do boleto/PIX no setor de construção.

### F4 · As aprovações estão em três mecânicas diferentes
A1 fora do SIENGE (papel/e-mail), A2/A3/A4/A5 dentro do SIENGE, D1 em ferramenta ainda inexistente.
A matriz de alçadas — que é **uma só** — acabaria parametrizada em dois sistemas.

Consequências: ninguém tem uma visão única de "o que está esperando minha aprovação"; não há SLA
por aprovação; e a divergência entre as duas cópias da matriz é questão de tempo.

### F5 · Nenhum prazo tem contagem regressiva automática
Seis prazos duros (dia 25, 15 dias, 2 dias úteis, encargos por medição, dias 5/14/21/28, 10 dias),
todos controlados por memória. A aprovação **A5 existe apenas porque o prazo de 15 dias é violado
com frequência** — é um controle criado para tratar uma falha recorrente a jusante, em vez de
evitá-la a montante.

### F6 · Comprovantes de encargos são conferidos a olho, a cada medição
Relação de funcionários + folha + DCTFWeb (INSS) + FGTS Digital, verificados pela gerenciadora antes
de cada A4. Isso é controle de **responsabilidade solidária previdenciária** — um dos maiores riscos
financeiros de uma construtora — sendo executado como conferência visual sem registro estruturado.

### F7 · A retenção contratual de 5% entra no sistema e some
Na obra, 5% de cada nota é retido, "registrado para liberação futura". A liberação está
explicitamente **fora de todos os procedimentos**. Não há, em lugar nenhum, um controle do saldo
retido por contrato, do termo de encerramento ou do fim do prazo de garantia.

Em uma obra de porte médio isso é um valor relevante parado, sem dono e sem relatório.

### F8 · Redigitação no contas a pagar
No fluxo de despesas sem pedido (cenário manual), o financeiro redigita ~40 títulos/mês a partir de
uma solicitação aprovada em outro sistema, e uma segunda pessoa confere a digitação. A conferência
existe para pegar o erro que a redigitação cria.

### F9 · Não existem indicadores
Nenhum dos procedimentos define métrica. Ninguém sabe hoje: lead time de cotação, % de compras com
3 propostas, % de escolhas fora do menor preço, aderência ao orçamento por obra, tempo médio por
aprovação, taxa de recusa fiscal de notas, quantas notas dependeram de A5.

## 2.2 A decisão que a empresa está tomando agora

O procedimento de despesas sem pedido está em v0.1 e apresenta dois cenários:

| | Cenário A · Microsoft 365 | Cenário B · BPM com API |
|---|---|---|
| Onde roda | SharePoint + Power Automate | Plataforma BPM contratada |
| Título no SIENGE | Manual, pelo financeiro | Automático, via API |
| Sustentação | Usuário-chave interno | Fornecedor da plataforma |
| Custo | Incluído no M365 (a confirmar plano) | Assinatura + implantação + API |
| Implantação | Rápida | Longa |

**Onde entra a sua proposta.** Se o escopo fosse só o fluxo de despesas sem pedido — 40 títulos/mês,
um formulário e uma aprovação — o cenário A resolveria, e propor um sistema próprio seria
sobre-engenharia. Você precisa dizer isso com todas as letras, porque é a objeção que vai receber.

O argumento a favor de um sistema próprio (chame de **cenário C**) não está nas despesas sem pedido.
Está em F1, F2, F3, F6 e F7 — cotação, cadastro de fornecedor, antifraude bancária, encargos
previdenciários e retenção contratual. Nenhum desses cabe bem em SharePoint, e num BPM genérico cada
um vira um projeto de customização pago à parte.

A proposta correta é: **o fluxo de despesas sem pedido é a primeira entrega, não o projeto.** Ele é
o menor pedaço que atravessa a arquitetura inteira (formulário → alçada → aprovação → integração
SIENGE → título) e por isso é o melhor teste da fundação. Entrega valor em ~2-3 meses e prova a
integração antes de a empresa apostar no resto.

## 2.3 Onde a automação realmente paga

Ordenado por retorno sobre esforço, não por ordem no fluxo:

| # | Oportunidade | Ganho | Esforço |
|---|--------------|-------|---------|
| 1 | **Motor único de alçadas + SoD** servindo A1–A5 e D1 | Uma fonte de verdade, fim da divergência, painel único de pendências | Médio |
| 2 | **QC nativo** com equalização, negociação e alertas automáticos | Substitui a planilha, gera trilha de A1 em cadeia, acumula base de preços | Médio-alto |
| 3 | **Portal do fornecedor** (ficha, documentos, propostas, NF, encargos) | Tira o e-mail do caminho crítico; validade de documentos controlada | Médio-alto |
| 4 | **Relógio de prazos** com contagem regressiva e escalonamento | Ataca a causa de A5 e do atraso em despesas | Baixo |
| 5 | **Cofre de documentos** vinculado ao registro | Fim de "pastas e e-mails pessoais"; auditoria em um clique | Baixo |
| 6 | **Integração de títulos com o SIENGE** | Elimina a redigitação e a conferência que existe por causa dela | Alto (depende da API) |
| 7 | **Controle de retenção contratual de 5%** | Recupera visibilidade de um passivo hoje sem dono | Baixo |
| 8 | **Checklist estruturado de encargos** por medição | Reduz risco de solidariedade previdenciária | Baixo |
| 9 | **Painel de indicadores** | Torna o processo gerenciável | Baixo (depois de 1–8) |

Note que quatro dos cinco itens mais baratos (4, 5, 7, 8) são de esforço baixo e **não dependem da
API do SIENGE**. Isso importa para o plano de risco: dá para entregar valor antes de saber se a
integração vai funcionar.
