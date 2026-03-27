---
name: writing-tests
description: Use when writing or reviewing unit tests, integration tests, e2e tests, or BDD scenarios; when the user mentions Jest, NestJS testing, Cucumber, test coverage, test performance or only test.
---

# Writing Tests

## Workflow

### Step 1 - Verificando a abordagem

- Se o teste for unitário
  - Garanta que apenas o alvo do teste está sendo testado, e não suas dependências.
  - Garanta que os mocks cobrem todos os casos de borda.
  - Garanta que os mocks estejam bem escritos e que sejam escritos unica e esclusivamente para aquele teste, e não compartilhado entre outros testes (exceto se for mock global).
- Se o teste for de integração
  - Garanta que realmente estão sendo utilizados os serviços, e não mocks deles
  - Garanta que existem testes para todos os casos de borda.

### Step 2 - Escrevendo os Testes

- Analise o arquivo alvo informado pelo usuário e crie uma lista de todos os casos de borda de cada função.
- Crie um bloco `describe` geral pro teste.
- Crie blocos `describe` para cada função a ser testada.
- Crie blocos `it` para todos os casos de borda. Estes inicialmente devem conter apenas a descrição e uma função vazia (sem testes).
- Adicione os testes nos blocos `it` seguindo os padrões determinados no arquivo `references/testing-patterns.md`.

### Step 3 - Executando Coverage

- Execute os testes coletando coverage com o comando definido no `package.json`, sendo geralmente `npm run test:cov` ou `npm run test:coverage`. Caso não saiba qual é, pode utilizar um comando montado na hora que use a biblioteca de testes do projeto.
- Caso por algum motivo algum teste quebre, execute as instruções do `Step 4` e depois retorne a este Step.
- Analise a saída do coverage e verifique quais arquivos não estão com cobertura completa e em quais linhas.
- Vá até as linhas que não possuem cobertura completa e verifique quais os casos de borda que estão presentes nessas linhas e ainda não foram mapeados.
- Com esta informação, volte ao `Step 2`, porém mantenha os testes existentes e aplique as instruções apenas para os novos casos de borda coletados, os adicionando nos blocos corretos.

### Step 4 - Verificando o Resultado

- Execute os testes no formato padrão com o comando definido no `package.json`, sendo geralmente `npm run test`. Caso não saiba qual é, pode utilizar um comando montado na hora que use a biblioteca de testes do projeto.
- Analise se todos os testes passaram corretamente, caso não, verifique na saída quais eram os resultados esperados e quais foram os recebidos e se estiverem diferentes, verifique mocks criados, o serviço testado e demais locais que poderiam impactar o teste.
- Caso seja um erro e não problema de resultados de testes, devolva o stack trace para o usuário e sugira uma solução, porém deixe que o usuário corrija o problema.

## Rules
- Sempre tente chegar ao máximo de coverage possível nos testes, escrevendo testes para todos os casos de borda possíveis.
- Siga os padrões determinados no arquivo `references/testing-patterns.md` SEMPRE, exceto se o usuário pedir para "ignorar padrões definidos".
- Quebre os testes de propósito para verificar se eles não estão retornando falso positivo e depois retorne para o teste correto.