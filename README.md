# Pedro — Chatbot de Atendimento da Loja do Fluminense

Chatbot textual em **Python** que simula um atendente virtual de uma loja de produtos do Fluminense. O Pedro conversa com o cliente pelo terminal, reconhece a **intenção** de cada mensagem com um classificador **Naive Bayes multinomial** (scikit-learn) e responde usando regras que consultam um **catálogo de produtos**.

> Projeto da **Avaliação Oficial 01 – 2026.2** da disciplina **Desenvolvimento de Chatbot**
> Centro Universitário Anhanguera de Niterói

## Autores

| Nome | RA |
|---|---|
| David Capulot Corrêa | |
| Gabriel do Almo Silveira Vicente  |
| Peter Emmerich Mulim e Silva |  |
| Welington Carlos Silva De Barros | |

---

## Sumário

- [Funcionalidades](#funcionalidades)
- [Como funciona](#como-funciona)
- [Estrutura do repositório](#estrutura-do-repositório)
- [Requisitos](#requisitos)
- [Como executar](#como-executar)
- [Exemplo de conversa](#exemplo-de-conversa)
- [Catálogo da loja](#catálogo-da-loja)
- [Estruturas de dados utilizadas](#estruturas-de-dados-utilizadas)
- [Limitações conhecidas](#limitações-conhecidas)
- [Melhorias futuras](#melhorias-futuras)
- [Declaração de uso de IA](#declaração-de-uso-de-ia)

---

## Funcionalidades

- Conversa em **loop** até o cliente se despedir.
- **Tratamento de texto**: minúsculas, remoção de pontuação e acentos, remoção de palavras irrelevantes (stopwords).
- **Reconhecimento de 9 intenções** com Machine Learning: `saudacao`, `ajuda`, `informacao`, `produto`, `preco`, `pagamento`, `entrega`, `despedida` e `desconhecida`.
- **Respostas variadas**: cada intenção tem uma lista de respostas e o Pedro sorteia uma delas.
- **Consulta ao catálogo**: ao citar um produto, o cliente recebe a descrição e os preços; ao citar um modelo (ex.: "infantil"), recebe apenas o preço dele.
- **Contexto de conversa**: o Pedro lembra do último produto mencionado e entende perguntas curtas como *"e a infantil?"*.
- **Apelidos de produtos**: "bermuda" e "calcao" levam a *short*, "pelota" leva a *bola*.

## Como funciona

O problema é dividido em duas perguntas:

1. **O que o cliente quer?** → respondida pelo classificador de ML (intenção).
2. **Sobre o que ele está falando?** → respondida por regras que procuram produto/modelo no catálogo.

### Fluxo do laço principal

```
ler entrada → prever intenção → gerar resposta (intenção + contexto) → exibir → é despedida? → encerra
```

### Geração da resposta (ordem de verificação)

1. Se a mensagem (ou o contexto) indica um **modelo específico** do produto → responde só com o preço dele.
2. Se o cliente cita um **produto** → mostra a descrição e todos os preços.
3. Caso contrário → sorteia uma resposta da lista da intenção prevista.

Em despedidas o contexto é ignorado.

### Pipeline de processamento de texto

```
texto → normalizar() → remover_palavras_ignoradas() → CountVectorizer → MultinomialNB → intenção
```

- `normalizar()`: minúsculas, troca pontuação por espaço, remove espaços duplicados e acentos (Unicode NFD).
- `remover_palavras_ignoradas()`: tira artigos, preposições e pronomes ("a", "de", "para", "com"...). Se sobrar texto vazio, usa o original.
- `CountVectorizer`: transforma o texto em contagem de palavras (*bag of words*).
- `MultinomialNB`: escolhe a intenção mais provável. O treino ocorre **a cada execução** do programa.
- Se nenhuma palavra da mensagem existe no vocabulário, a intenção é tratada direto como `desconhecida`.

## Estrutura do repositório

| Arquivo | Descrição |
|---|---|
| `prova_de_pedro.py` | Código-fonte completo do chatbot (exportado do Colab). |
| `Prova_de_pedro.ipynb` | Notebook (Google Colab) com o mesmo código. |
| `Avaliação_Oficial_01_-_2026_2_Desenvolvimento_de_Chatbot.pdf` | Relatório do projeto (decisões, exemplos, limitações). |
| `saidaIA.txt` | Registro das conversas com IA usadas como apoio (ver declaração de uso de IA). |
| `README.md` | Este arquivo. |

## Requisitos

- Python **3.8+**
- Bibliotecas:
  - `pandas`
  - `scikit-learn`
- Módulos da biblioteca padrão (não precisam de instalação): `unicodedata`, `time`, `random`

## Como executar

### Opção 1 — Terminal local

```bash
# 1. (opcional) criar e ativar um ambiente virtual
python -m venv venv
source venv/bin/activate        # Linux/macOS
venv\Scripts\activate           # Windows

# 2. instalar as dependências
pip install pandas scikit-learn

# 3. executar
python prova_de_pedro.py
```

### Opção 2 — Google Colab

1. Abra o arquivo `Prova_de_pedro.ipynb` no [Google Colab](https://colab.research.google.com/).
2. Execute todas as células (**Ambiente de execução → Executar tudo**).
3. Converse com o Pedro na saída da última célula.

Para encerrar a conversa, basta se despedir (ex.: "tchau").

## Exemplo de conversa

```
...: Olá! Eu me chamo Pedro e eu estarei te atendendo hoje.

Pedro: Qual é o seu nome?

Usuário Tricolor: David

Pedro: Prazer, David! Como posso ajudar você?

David: quanto é a camisa

Pedro: Temos camisa titular, reserva, infantil, feminina e de goleiro. Titular: R$ 249,90.
       Reserva: R$ 229,90. Infantil: R$ 149,90. Feminina: R$ 199,90.
       Masculina: a partir de R$ 199,90. Goleiro: R$ 259,90. Posso ajudar em mais alguma coisa?

David: e a titular?

Pedro: Titular: R$ 249,90. Posso ajudar em mais alguma coisa?

David: vocês entregam em casa?

Pedro: A gente não faz entrega, mas você pode vir buscar seu produto na loja.
```

Neste exemplo o Pedro guarda "camisa" como último produto, por isso entende "e a titular?" sem o produto ser citado. Na loja fictícia, **pagamento e retirada são presenciais e não há entrega**.

## Catálogo da loja

> Os preços são **valores de exemplo**, não de uma loja real.

| Produto | Modelos / Preço |
|---|---|
| Camisa | Titular R$ 249,90 · Reserva R$ 229,90 · Infantil R$ 149,90 · Feminina R$ 199,90 · Masculina a partir de R$ 199,90 · Goleiro R$ 259,90 |
| Short | Masculino R$ 89,90 · Feminino R$ 79,90 |
| Boné | a partir de R$ 59,90 |
| Bandeira | a partir de R$ 49,90 |
| Cachecol | a partir de R$ 39,90 |
| Bola | a partir de R$ 89,90 |
| Chuteira | a partir de R$ 199,90 |

Para **adicionar um produto**, basta incluir uma nova entrada no dicionário `catalogo` (e, se quiser, em `apelidos`), sem alterar as regras.

## Estruturas de dados utilizadas

| Estrutura | Para que serve |
|---|---|
| Dicionário `exemplos` | Cada intenção aponta para uma lista de frases de treino (frase sempre com seu rótulo). |
| Listas `respostas_*` | Várias respostas prontas por intenção; o Pedro sorteia uma com `random.choice`. |
| Dicionário aninhado `catalogo` | Descrição e preço (ou dicionário de modelos) de cada produto. |
| Dicionário `apelidos` | Liga palavras alternativas ao nome oficial do produto. |
| Conjunto `palavras_ignorar` | Stopwords, com consulta de pertencimento rápida. |
| Tupla `contexto` | Guarda o último produto mencionado. |
| `DataFrame` (pandas) | Organiza frases de treino e intenções antes da vetorização. |

O conjunto de treino tem **232 frases** distribuídas entre as 9 intenções.

## Limitações conhecidas

- **Vocabulário limitado** às frases de treino: gírias, erros de digitação (ex.: "camiza") e produtos não cadastrados passam despercebidos.
- **Bag of words** ignora a ordem das palavras e não distingue pergunta de negação.
- O classificador **sempre escolhe uma intenção** se reconhecer ao menos uma palavra, mesmo com baixa confiança.
- O **nome do usuário** aceita qualquer texto, inclusive frases inteiras; símbolos especiais podem não ser exibidos no terminal. Se ficar vazio, usa "Torcedor".
- O Pedro **não responde sobre si mesmo** (ex.: "qual seu nome") nem **repete respostas** ("repete o preço").
- O contexto guarda **um produto por vez**; se dois forem citados, responde só sobre o primeiro encontrado.
- Não consulta estoque, não registra pedidos e não fala de nada fora do catálogo e das intenções cadastradas.

## Melhorias futuras

- Ampliar o conjunto de treino com mensagens escritas por pessoas de fora do grupo.
- Avaliar o modelo com métricas (acurácia, matriz de confusão) em vez de apenas testes manuais.
- Substituir a sequência de `if/elif` da função `gerar_resposta` por um dicionário que liga cada intenção à sua lista de respostas.
- Usar um limiar de confiança do classificador para cair em `desconhecida` quando houver dúvida.
- Validar o nome informado pelo usuário e guardar a última resposta para permitir "repete".

## Declaração de uso de IA

Foram usados **ChatGPT** e **Claude** como apoio. A ideia do projeto, as intenções, o catálogo, o fluxo de decisão e a lógica de contexto foram definidos pelo grupo, e o código foi testado e ajustado pelos integrantes.

- **Código:** versão completa da função `normalizar` (remoção de pontuação com `replace()`) e criação do conjunto `palavras_ignorar`.
- **Listas de respostas:** primeira versão das listas de saudação, ajuda, informação, despedida e desconhecida. As listas de produto, preço, pagamento e entrega foram escritas pelo grupo.
- **Dados de treino:** estruturação em textos e intenções, ampliação para vinte frases por intenção e refinamento entre informação e produto.
- **Adaptação final:** uso de `CountVectorizer` no lugar do TF-IDF sugerido, dicionário de frases por intenção, intenção de preço e função de contexto (criada pelo grupo, sem sugestão de IA).

O registro das conversas com a IA está em `saidaIA.txt`.
