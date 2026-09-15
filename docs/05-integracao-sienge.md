# 5. Integração com o SIENGE — o que precisa ser verificado antes de qualquer promessa

> **Este é o documento mais importante do conjunto para o risco do projeto.** Todas as estimativas
> das Ondas 1 a 4 dependem do que está aqui. Trate como pré-requisito, não como detalhe técnico.

## 5.1 O que foi possível confirmar na documentação pública

| Item | O que se sabe |
|------|---------------|
| Documentação | Publicada em `https://api.sienge.com.br/docs/`, gerada dinamicamente e atualizada diariamente |
| Autenticação | **Basic Auth**. Usuário e senha **de API**, criados no **Portal de Integrações** — são credenciais distintas do login do Sienge Plataforma |
| Endereçamento | Por **tenant/subdomínio**. Se o acesso da empresa é `casa8.sienge.com.br`, o tenant é `casa8` e compõe a URL da chamada |
| Credores | Existe API REST de credores, incluindo inserção de dados bancários (`POST /creditors/{creditorId}/bank-informations`) |
| Contas a pagar | Existem APIs de **consulta** de títulos, de **alteração de unidades** em títulos e **Bulk Data** para consulta em massa de parcelas |
| Inserção de título | A API de inserção documentada é a **baseada em Nota Fiscal Eletrônica recebida** (`eletronic-invoice-bills`) |
| **Restrição de acesso** | **Clientes com servidor local (on-premise) não têm acesso às APIs.** Os recursos de API são disponibilizados para clientes em datacenter/nuvem |

## 5.2 Gate zero — a pergunta que precede todas as outras

> ### O SIENGE da CASA8 está em nuvem (DC) ou em servidor local?

Se estiver em **servidor local**, a API não está disponível e **todo o cenário B da empresa e a
integração desta proposta ficam inviáveis como desenhados**. O projeto continua fazendo sentido (a
maior parte do valor — QC, portal do fornecedor, alçadas, encargos, retenção — não depende da API),
mas a gravação no contas a pagar passa a exigir o Plano B da §5.5.

**Esta é a primeira pergunta a fazer na empresa. Antes de estimar qualquer coisa.**

Perguntas que vêm junto:
- Qual é o tenant/subdomínio?
- Existe usuário de API criado no Portal de Integrações? Quem administra?
- O contrato atual inclui uso de API, ou é módulo/custo à parte?
- Há limite de requisições contratado?

## 5.3 A segunda pergunta: existe criação de título *avulso* via API?

O caso de uso central da Onda 1 é criar um **título avulso** — sem vínculo com pedido, contrato,
medição **ou nota fiscal**. É exatamente como o procedimento define:

> *Título avulso — título lançado direto no contas a pagar do SIENGE, sem vínculo com pedido,
> contrato, medição ou nota de compra.*

Mas a API de inserção de títulos que aparece na documentação pública é a de **`eletronic-invoice-bills`**,
ou seja, ancorada em NF-e recebida. Guias de tributo, ITBI, taxas de prefeitura e reembolso de
colaborador **não têm NF-e**.

Há três desfechos possíveis, e é o spike que decide qual é:

1. **Existe endpoint de título avulso** não coberto pela busca pública → Onda 1 corre como planejada.
2. **Existe, mas exige campos que o processo não tem** (ex.: número/chave de documento fiscal) →
   ajustar o de-para e possivelmente o formulário.
3. **Não existe** → Plano B da §5.5 para o fluxo de despesas, mantendo o restante do projeto.

## 5.4 Checklist do spike técnico — Onda 0, ~1 semana

Cada linha é uma chamada real contra um ambiente de homologação, com resultado registrado.

**Bloco 1 · Acesso (bloqueante)**
- [ ] Confirmar nuvem vs. on-premise
- [ ] Obter credencial de API no Portal de Integrações e autenticar com sucesso
- [ ] Identificar o tenant e a URL base efetiva
- [ ] Verificar se existe ambiente de homologação separado do de produção
- [ ] Medir rate limit e tempo de resposta típico

**Bloco 2 · Leitura de cadastros (alimenta o espelho)**
- [ ] Empresas/SPEs, com CNPJ
- [ ] Obras, com CNO
- [ ] Centros de custo e departamentos
- [ ] Plano financeiro (contas)
- [ ] Insumos
- [ ] Credores/fornecedores, incluindo dados bancários
- [ ] Orçamento da obra por item — *existe leitura? é o insumo do alerta orçamentário*
- [ ] Verificar se há paginação e Bulk Data para as cargas grandes

**Bloco 3 · Escrita (o coração do risco)**
- [ ] **Criar título avulso sem NF-e** ← a pergunta da §5.3
- [ ] Anexar arquivo a um título (documento de cobrança + comprovante da aprovação)
- [ ] Criar título a partir de NF-e recebida (caminho do Processo A, material)
- [ ] Criar/atualizar credor — ou confirmar que o cadastro permanece manual em suprimentos
- [ ] Registrar medição contra itens de contrato
- [ ] Lançar retenções tributárias no título
- [ ] Registrar a retenção contratual de 5%

**Bloco 4 · Comportamento sob erro**
- [ ] O que retorna em campo obrigatório ausente? A mensagem é acionável?
- [ ] Reenvio da mesma requisição duplica o título? (**idempotência** — determina se precisamos de
      chave própria de deduplicação)
- [ ] Comportamento em indisponibilidade: timeout, 5xx, manutenção programada
- [ ] É possível consultar um título criado para confirmar gravação?

**Bloco 5 · Permissões**
- [ ] O usuário de API respeita as restrições por ação do SIENGE (o procedimento cita as ações
      **1604, 1605, 1607 e 1608** para cadastro de título avulso)?
- [ ] É possível restringir o usuário de API só ao que ele precisa?

**Saída do spike:** um documento de uma página por bloco, com `curl` que funcionou, resposta real e
veredito. Isso vira a base da estimativa firme.

## 5.5 Plano B — se a escrita não for possível

Não é o fim do projeto. É uma mudança de desenho que preserva a maior parte do valor.

```
  Sistema CASA8                                          SIENGE
 ┌─────────────────┐                                   ┌────────────┐
 │ Aprovação D1    │                                   │ Contas     │
 │ concluída       │──► "Ficha de lançamento" ─────────►│ a pagar    │
 └─────────────────┘    (tela pronta para digitar:      │ (digitação │
                         todos os campos, na ordem       │  manual)   │
                         da tela do SIENGE, com          └─────┬──────┘
                         copiar-em-um-clique)                  │
                                                               │ nº do título
         ┌─────────────────────────────────────────────────────┘
         ▼
 ┌────────────────────────────────────────┐
 │ Conferência automática por Bulk Data:  │  o sistema LÊ o título criado e
 │ compara o título lido com o aprovado.  │  compara com o que foi aprovado.
 │ Divergência → alerta.                  │  A conferência humana deixa de ser
 └────────────────────────────────────────┘  digitação-contra-digitação.
```

Isso aproveita que **a leitura (Bulk Data) tem muito mais chance de estar disponível que a escrita**.
Mesmo sem gravar, o sistema elimina a segunda conferência manual: em vez de uma pessoa reconferir o
que a outra digitou, a máquina compara o título real com a solicitação aprovada.

Ganho preservado: aprovação, rastreabilidade, prazos, anexos, indicadores.
Ganho perdido: a digitação no contas a pagar continua existindo.

## 5.6 Princípios da integração, independentemente do resultado

1. **Assíncrona sempre.** O usuário nunca espera o SIENGE responder. Aprovou → evento → fila.
2. **Idempotência do nosso lado.** Chave de deduplicação própria por solicitação, porque não se pode
   assumir que o SIENGE a tem.
3. **Fila de erro é tela, não log.** O financeiro precisa ver "3 títulos falharam, este é o motivo,
   este é o botão de reprocessar".
4. **Contingência é estado de primeira classe.** `titulo.origem = integracao | contingencia`, para o
   controle "título lançado em contingência é conferido com a mesma regra e identificado como exceção".
5. **O SIENGE sempre vence nas divergências de cadastro.** O espelho é cópia de leitura, com carimbo
   de sincronização visível na tela.
6. **Credencial de API é segredo gerenciado**, nunca em código, com rotação prevista.

---

**Fontes consultadas:**
[Como entender a documentação das APIs — Sienge](https://ajuda.sienge.com.br/support/solutions/articles/153000200931-como-entender-a-documenta%C3%A7%C3%A3o-das-apis-) ·
[API REST Credores](https://ajuda.sienge.com.br/support/solutions/articles/153000200200-api-rest-credores) ·
[API de inserção de títulos com base em NF-e recebida](https://ajuda.sienge.com.br/support/solutions/articles/153000200156-alterac%C3%A3o-de-api-de-inserc%C3%A3o-post-de-t%C3%ADtulos-do-contas-a-pagar-com-base-na-nota-fiscal-eletr%C3%B4nica-r) ·
[Bulk Data de parcelas do contas a pagar](https://ajuda.sienge.com.br/support/solutions/articles/153000200215-contas-a-pagar-melhorias-na-api-bulk-data-consulta-em-massa-de-parcelas-do-contas-a-pagar) ·
[Documentação técnica](https://api.sienge.com.br/docs/)

*Informação de documentação pública, sujeita a mudança e ao contrato específico da CASA8.
Nada aqui substitui a verificação do spike.*
