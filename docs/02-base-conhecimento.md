# Base de Conhecimento

## Dados Utilizados

| Arquivo | Formato | Para que serve no Tom? |
|---------|---------|---------------------|
| `historico_atendimento.csv` | CSV | Contextualizar interações anteriores, ou seja, dar continuidade ao atendimento de forma mais eficiente. |
| `perfil_investidor.json` | JSON | Personalizar as explicações sobre as dúvidas e necessidades de aprendizado do cliente. |
| `produtos_financeiros.json` | JSON | Conhecer os produtos disponíveis para que eles possam ser ensinados ao cliente. |
| `transacoes.csv` | CSV | Analisar padrão de gastos do cliente e usar essas informações de forma didática. |

> [!TIP]
> **Quer um dataset mais robusto?** Você pode utilizar datasets públicos do [Hugging Face](https://huggingface.co/datasets) relacionados a finanças, desde que sejam adequados ao contexto do desafio.

---

## Adaptações nos Dados

Os dados mockados foram adaptados para a realidade do Agente TOM, com foco em educação financeira para um cliente do tipo **Adolescente**.

As principais alterações realizadas foram:

- Ajuste do perfil do cliente para **Luiz Augusto**, 17 anos, estudante, com perfil investidor **iniciante**.
- Definição de renda mensal simulada de **R$ 1.250,00**, patrimônio total de **R$ 550,00** e reserva atual de **R$ 250,00**.
- Inclusão de metas financeiras simples e compatíveis com o público-alvo, como:
  - montar uma pequena reserva;
  - guardar dinheiro para lazer e estudos.
- Adaptação do arquivo de transações para refletir entradas e saídas do cotidiano de um jovem, com categorias como:
  - alimentação,
  - transporte,
  - ensino,
  - lazer,
  - vestuário,
  - investimento.
- Inclusão de movimentações relacionadas a **Objetivos** e **CDB**, para permitir que o agente explique produtos simples e conservadores de forma educativa.
- Adaptação do histórico de atendimento com temas reais do contexto proposto, como:
  - CDB,
  - objetivos,
  - metas financeiras,
  - bloqueio e desbloqueio de cartão,
  - problema no app.
- Revisão dos produtos financeiros utilizados pelo agente, priorizando opções introdutórias, seguras e de linguagem acessível, como:
  - **CDB**,
  - **Ivest por objetivos**,
  - **Poupa Troco**,
  - **Planilha de gastos**.

Essas adaptações foram feitas para garantir coerência entre o perfil do usuário, o tom educativo do agente e os exemplos de uso apresentados no projeto.

---

## Estratégia de Integração

### Como os dados são carregados?

Os arquivos `JSON` e `CSV` são carregados no início da execução do agente em Python, utilizando bibliotecas como `pandas` para os arquivos tabulares e `json` para os arquivos estruturados.

Cada arquivo possui uma função específica no sistema:

- `perfil_investidor.json`: carrega os dados do cliente, perfil, objetivos e metas;
- `transacoes.csv`: fornece o histórico financeiro simplificado do usuário;
- `historico_atendimento.csv`: recupera interações anteriores e dúvidas já resolvidas;
- `produtos_financeiros.json`: oferece a base de conhecimento sobre produtos e recursos financeiros educativos.

Esses dados são organizados antes da geração da resposta, permitindo que o agente monte um contexto mais personalizado, seguro e coerente.

### Como os dados são usados no prompt?

Os dados não precisam ser totalmente inseridos no *system prompt*. A estratégia mais adequada é fazer uma **consulta dinâmica** com base na pergunta do usuário.

Funciona assim:

- o agente identifica a intenção da pergunta;
- consulta os arquivos relevantes;
- seleciona apenas os dados necessários para aquela resposta;
- monta um contexto resumido para enviar ao modelo.

Por exemplo:

- se o usuário perguntar sobre gastos, o agente consulta `transacoes.csv`;
- se perguntar sobre metas ou perfil, consulta `perfil_investidor.json`;
- se a dúvida estiver relacionada a um tema já tratado, consulta `historico_atendimento.csv`;
- se a pergunta for sobre produtos financeiros simples, consulta `produtos_financeiros.json`.

Essa abordagem reduz ruído no prompt, melhora a personalização da resposta e ajuda a evitar alucinações, já que o agente responde com base apenas nos dados disponíveis.

---

## Exemplo de Contexto Montado


```text
Dados do Cliente:
- Nome: Luiz Augusto
- Tipo de cliente: Adolescente
- Idade: 17 anos
- Ocupação: Estudante
- Perfil investidor: Iniciante
- Renda mensal: R$ 1.250,00
- Patrimônio total: R$ 550,00
- Reserva atual: R$ 250,00
- Objetivo principal: Aprender a organizar gastos, criar hábito de poupar e acompanhar metas simples

Metas atuais:
- Montar uma pequena reserva até 12/2026
- Guardar dinheiro para lazer e estudos até 08/2026

Últimas transações:
- 01.03.2026: Bolsa Estágio - R$ 1.000,00 (entrada)
- 15.03.2026: Mesada - R$ 250,00 (entrada)
- 16.03.2026: Cinema - R$ 25,00 (saída)
- 25.03.2026: ShoesTop - R$ 189,00 (saída)
- 28.03.2026: Cofrinho - R$ 250,00 (saída)
- 28.03.2026: CDI - R$ 300,00 (saída)

Histórico de atendimento:
- 01.03.2026: Dúvida sobre Conta Objetivos e Poupa Troco
- 05.03.2026: Acompanhamento de meta financeira
- 25.03.2026: Dúvida sobre CDB
- 10.03.2026: Bloqueio de cartão

Produtos disponíveis na base:
- CDB
- Conta Objetivos
- Poupa Troco
- Planilha de gastos

