# Base de Conhecimento

> [!TIP]
> **Prompt Sugerido para esta etapa:**
> Preciso organizar a base de conhecimento do meu agente financeiro educativo.
> Tenho esses arquivos de dados:[data].
> Me ajude a:
> (1) entender o que cada arquivo contém;
> (2) decidir como usar cada um;
> (3) criar um exemplo de contexto formatado para incluir no prompt.


## Dados Utilizados

| Arquivo | Formato | Para que serve no TOM? |
|---------|---------|------------------------|
| `historico_atendimento.csv` | CSV | Contextualizar interações anteriores e permitir continuidade no atendimento de forma mais eficiente. |
| `perfil_investidor.json` | JSON | Personalizar as explicações com base no perfil, nas metas e nas necessidades de aprendizado do cliente. |
| `produtos_financeiros.json` | JSON | Representar os produtos e recursos financeiros que o agente pode explicar ao cliente. |
| `transacoes.csv` | CSV | Analisar o padrão de gastos, entradas e saídas do cliente de forma educativa e contextualizada. |

> [!TIP]
> **Quer um dataset mais robusto?** Você pode utilizar datasets públicos do [Hugging Face](https://huggingface.co/datasets), desde que sejam compatíveis com o contexto do desafio e com a proposta educacional do agente.

---

## Adaptações nos Dados

Os dados mockados foram adaptados para a realidade do **Agente TOM**, com foco em **educação financeira para um cliente adolescente**.

As principais alterações realizadas foram:

- ajuste do perfil do cliente para **Luiz Augusto**, 17 anos, estudante, com perfil investidor **iniciante**;
- definição de renda mensal simulada de **R$ 1.250,00**, patrimônio total de **R$ 550,00** e reserva atual de **R$ 250,00**;
- inclusão de metas financeiras simples e compatíveis com o público-alvo, como:
  - montar uma pequena reserva;
  - guardar dinheiro para lazer e estudos;
- adaptação do arquivo de transações para refletir entradas e saídas comuns no cotidiano de um jovem, com categorias como:
  - alimentação;
  - transporte;
  - educação;
  - lazer;
  - vestuário;
  - investimento;
- inclusão de movimentações relacionadas a **Conta por Objetivos** e **CDB**, permitindo que o agente explique produtos simples e conservadores de forma educativa;
- adaptação do histórico de atendimento com temas coerentes com o cenário proposto, como:
  - problema no app;
  - metas financeiras;
  - bloqueio e desbloqueio de cartão;
  - Conta por Objetivos;
  - CDB;
- revisão dos produtos financeiros utilizados pelo agente, priorizando opções introdutórias, acessíveis e de baixo risco, como:
  - **CDB com liquidez diária**;
  - **Conta por Objetivos**;
  - **Planejador de gastos**.

Essas adaptações foram realizadas para manter coerência entre o perfil do cliente, o tom educativo do agente e os cenários de uso apresentados no projeto.

---

## Estratégia de Integração

### Como os dados são carregados?

Existem duas possibilidades, injetando os dados diretamente no prompt manualmente (Ctrl + C , Ctrl + V), ou carregar os arquivos via código.

Os arquivos da pasta `data/` são carregados no início da execução do agente em Python. Para isso, são utilizadas bibliotecas como `pandas`, para leitura dos arquivos `CSV`, e `json`, para leitura dos arquivos `JSON`.

Exemplo de carregamento:

```python
import pandas as pd
import json

# CSVs
historico = pd.read_csv("data/historico_atendimento.csv")
transacoes = pd.read_csv("data/transacoes.csv")

# JSONs
with open("data/perfil_investidor.json", "r", encoding="utf-8") as f:
    perfil = json.load(f)

with open("data/produtos_financeiros.json", "r", encoding="utf-8") as f:
    produtos = json.load(f)
```

Cada arquivo possui uma função específica no sistema:

- `perfil_investidor.json`: carrega os dados do cliente, perfil, objetivos e metas;
- `transacoes.csv`: fornece o histórico financeiro simplificado do usuário;
- `historico_atendimento.csv`: recupera interações anteriores e dúvidas já registradas;
- `produtos_financeiros.json`: representa a base de conhecimento sobre produtos financeiros e recursos educativos.

Após o carregamento, os dados ficam disponíveis para consulta durante a interação com o usuário.

### Como os dados são usados no prompt?

Os dados não são inseridos integralmente no *system prompt*. A estratégia adotada é fazer uma **consulta dinâmica**, de acordo com a intenção da pergunta feita pelo usuário.
Para simplificar, podemos injetar os dado em nosso prompt, guarantindo qu o Agente tenha o melhor contexto possível. Lembrando que, em soluções mais robustas, o ideal é que essas informações sejam carregadas dinamicamente para que possamos ganhar flexibilidade.

O fluxo funciona assim:

- o agente identifica a intenção da pergunta;
- consulta os arquivos mais relevantes;
- seleciona apenas os dados necessários para aquela resposta;
- monta um contexto resumido para enviar ao modelo.

Por exemplo:

- se o usuário perguntar sobre gastos, o agente consulta `transacoes.csv`;
- se perguntar sobre metas, objetivos ou perfil, consulta `perfil_investidor.json`;
- se a dúvida estiver relacionada a um tema já tratado anteriormente, consulta `historico_atendimento.csv`;
- se a pergunta for sobre produtos financeiros, consulta `produtos_financeiros.json`.

Essa abordagem reduz ruído no prompt, melhora a personalização da resposta e ajuda a evitar alucinações, já que o agente responde com base apenas nas informações disponíveis na base de conhecimento.

```text
DADOS E PERFIL DO CLIENTE (data/perfil_investidor.json): 
{
  "nome": "Luiz Augusto",
  "tipo_cliente": "Adolescente",
  "idade": 17,
  "ocupacao": "Estudante",
  "renda_mensal": 1250.0,
  "perfil_investidor": "iniciante",
  "objetivo_principal": "Aprender a organizar gastos, criar hábito de poupar e acompanhar metas simples",
  "patrimonio_total": 550.0,
  "reserva_emergencia_atual": 250.0,
  "aceita_risco": false,
  "metas": [
    {
      "meta": "Montar uma pequena reserva",
      "valor_necessario": 1000.0,
      "prazo": "2026-12"
    },
    {
      "meta": "Guardar dinheiro para lazer e estudos",
      "valor_necessario": 600.0,
      "prazo": "2026-08"
    }
  ],
  "observacoes": {
    "contexto": "Perfil adaptado para o Agente TOM, com foco educativo e linguagem acessível para adolescentes.",
    "recomendacao_geral": "Priorizar organização financeira, metas simples, cofrinho e produtos conservadores."
  }
}

TRANSAÇÕES DO CIENTE (data/transacoes.csv): 
data,descricao,categoria,valor,tipo
01.03.2026,Bolsa Estágio,receita,1000,entrada
02.03.2026,Cantina,alimentacao,20,saida
02.03.2026,Uber,transporte,12,saida
05.03.2026,Curso,educacao,100,saida
12.03.2026,Cantina,alimentacao,20,saida
15.03.2026,Mesada,receita,250,entrada
16.03.2026,Cinema,lazer,25,saida
22.03.2026,Uber,transporte,16,saida
25.03.2026,Loja de Calçados,vestuario,189,saida
27.03.2026,Supermercado,alimentacao,80,saida
28.03.2026,Reserva pessoal,investimento,250,saida
28.03.2026,CDB,investimento,300,saida

HISTÓRIO DE ATENDIMENTO DO CLIENTE (data/historico_atendimento.csv):
data,canal,tema,resumo,resolvido
23.02.2026,telefone,problema no app,Erro ao visualizar extrato foi corrigido,sim
01.03.2026,chat,Conta por objetivos,Cliente pediu orientações sobre como separar dinheiro por metas,sim
05.03.2026,chat,Metas financeiras,Cliente acompanhou o progresso da meta de viagem pelo aplicativo,sim
10.03.2026,chat,bloqueio de cartao,Solicitacao de bloqueio de cartao por perda e emissao de novo cartao,sim
22.03.2026,telefone,desbloqueio de cartao,Desbloqueio de cartao realizado com sucesso,sim
25.03.2026,chat,CDB,"Cliente perguntou sobre rentabilidade, prazo e seguranca do produto",sim

PRODUTOS DISPONÍVEIS PARA O ENSINO (data/produtos_financeiros.json):
[
  {
    "nome": "CDI",
    "categoria": "renda_fixa",
    "risco": "baixo",
    "rentabilidade": "varia conforme o percentual do CDI",
    "aporte_minimo": 50.0,
    "indicado_para": "Iniciantes que querem entender rendimento de forma simples"
  },
  {
    "nome": "Conta com objetivo",
    "categoria": "organizacao_financeira",
    "risco": "baixo",
    "rentabilidade": "varia conforme o percentual do CDI",
    "aporte_minimo": 10.0,
    "indicado_para": "Quem quer separar dinheiro por meta, como viagem ou curso e adolescentes que estão começando a guardar dinheiro"
  },
  {
    "nome": "Planilha de gastos",
    "categoria": "educacao_financeira",
    "risco": "baixo",
    "rentabilidade": "não se aplica",
    "aporte_minimo": 0.0,
    "indicado_para": "Usuários que precisam visualizar para onde o dinheiro está indo"
  }
]
```


---

## Exemplo de Contexto Montado

O exemplo de contexto montado abaixo, se baseia nos dados originais de base de conhecimento, mas os sintetiza deixando apenas as informações mais relevantes, otimizando assim o consumo de tokens. Entretando vale lembrar que o mais importantes do que economizar tokens, é ter todas as informações relevantes disponíveis em seu contexto.

```text
Dados do Cliente:
- Nome: Luiz Augusto
- Perfil investidor: Iniciante
- Renda mensal: R$ 1.250,00
- Reserva atual: R$ 250,00
- Objetivo principal: Aprender a organizar gastos, criar hábito de poupar e acompanhar metas simples

Metas atuais:
- Montar uma pequena reserva até 12/2026
- Guardar dinheiro para lazer e estudos até 08/2026

Últimas transações:
  Entrada
- 01.03.2026: Bolsa Estágio - R$ 1.000,00
- 15.03.2026: Mesada - R$ 250,00
- Total: R$ 1.250,00
  Saída
- 16.03.2026: Cinema - R$ 25,00
- 25.03.2026: Loja de Calçados - R$ 189,00
- 28.03.2026: Reserva pessoal - R$ 250,00
- 28.03.2026: CDB - R$ 300,00
- Total de saída: 764,00

Histórico de atendimento:
- 01.03.2026: Dúvida sobre Conta por Objetivos
- 05.03.2026: Acompanhamento de meta financeira
- 10.03.2026: Bloqueio de cartão
- 22.03.2026: Desbloqueio de cartão
- 25.03.2026: Dúvida sobre CDB
```
