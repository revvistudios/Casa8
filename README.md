# CASA8 · Centralização e automação do processo de suprimentos e despesas

Análise do processo atual do Grupo CASA8 e proposta de sistema para centralizá-lo, a partir dos
procedimentos, modelos e fluxos fornecidos pela controladoria.

## Resumo em uma página

**O que existe hoje.** Dois procedimentos bem escritos — Suprimentos (v1.0) e Despesas sem Pedido
(v0.1, ainda em minuta) — executados sobre SIENGE + Excel + Word + e-mail. Seis pontos de aprovação
(A1 a A5 e D1) distribuídos por três mecânicas diferentes. Seis prazos duros controlados por
memória. O quadro de concorrência e o cadastro de fornecedor vivem inteiramente fora de qualquer
sistema.

**A tese.** O SIENGE continua sendo o sistema de registro. O sistema novo é a camada de processo:
cotação, aprovações, documentos, prazos, portal do fornecedor e indicadores — escrevendo o resultado
no SIENGE ao final de cada etapa. Não se constrói um ERP paralelo.

**Onde está o valor.** Não está nas despesas sem pedido (~40 títulos/mês), que o Microsoft 365
resolveria. Está na cotação, no cadastro de fornecedor, na antifraude bancária, nos encargos
previdenciários por medição e na retenção contratual de 5% — nada disso cabe em SharePoint, e num
BPM genérico cada item vira uma customização paga.

**O caminho.** Cinco ondas, ~11 a 13 meses para um desenvolvedor em tempo integral, começando pelo
fluxo de despesas sem pedido — o menor pedaço que atravessa a arquitetura inteira e prova a
integração antes de a empresa apostar no resto.

**O risco número 1.** A API do SIENGE não está disponível para clientes com servidor local, e a
inserção de título documentada publicamente é a baseada em NF-e — enquanto o caso de uso das
despesas é justamente o **título avulso sem nota**. Há um spike técnico de uma semana proposto antes
de qualquer compromisso de prazo.

## Documentos

| Arquivo | Conteúdo |
|---------|----------|
| [`docs/01-entendimento-do-dominio.md`](docs/01-entendimento-do-dominio.md) | Como o processo funciona hoje: fluxos, aprovações, prazos, entidades e as regras de negócio duras |
| [`docs/02-diagnostico.md`](docs/02-diagnostico.md) | As nove fraturas do processo atual e onde a automação realmente paga |
| [`docs/03-proposta.md`](docs/03-proposta.md) | **A proposta**: tese, escopo, arquitetura, stack, plano em ondas, comparação com os cenários A e B, riscos e indicadores |
| [`docs/04-modelo-de-dados.md`](docs/04-modelo-de-dados.md) | Entidades, o modelo que substitui a planilha de cotação e as máquinas de estado |
| [`docs/05-integracao-sienge.md`](docs/05-integracao-sienge.md) | O que se sabe da API, o checklist do spike técnico e o plano B |
| [`docs/06-controles.md`](docs/06-controles.md) | Os 28 controles-chave dos procedimentos convertidos em mecanismo de sistema |
| [`docs/07-perguntas-abertas.md`](docs/07-perguntas-abertas.md) | O que perguntar na empresa, incluindo o que perguntar sobre o seu próprio papel |
| [`docs/08-proposta-comercial.md`](docs/08-proposta-comercial.md) | **A proposta comercial** para enviar ao cliente: checkpoints do sistema, implementação em cinco fases a partir de um mapeamento contratado à parte, investimento e condições |
| [`docs/prompts/portal-cotacao-fornecedor.md`](docs/prompts/portal-cotacao-fornecedor.md) | Prompt para o Claude Design gerar as telas do portal de cotação do fornecedor |

## Como usar isto

1. Comece pelo **`03-proposta.md`** — é o documento que vai para a diretoria.
2. Leve o **`07-perguntas-abertas.md`** para as primeiras reuniões. As oito perguntas bloqueantes
   valem mais que qualquer estimativa feita sem elas.
3. Faça o spike do **`05-integracao-sienge.md`** antes de prometer prazo.
4. Use o **`06-controles.md`** para mostrar à controladoria que o sistema não enfraquece os
   controles que ela escreveu — ele os torna executáveis.

## Fontes

Procedimento de Suprimentos CASA8 (v1.0, set/2026) · Procedimento de Custos e Despesas sem Pedido ou
Contrato CASA8 (v0.1, set/2026) · Quadro de Concorrência CASA8 (XLSX) · Ficha Cadastral de Fornecedor
CASA8 (DOCX) · Fluxos em raias de suprimentos e dos cenários A e B de despesas.
