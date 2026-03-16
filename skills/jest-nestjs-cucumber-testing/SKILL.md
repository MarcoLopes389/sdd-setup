---
name: jest-nestjs-cucumber-testing
description: Creates accurate and performant tests using Jest, @nestjs/testing, and Cucumber for NestJS applications. Use when writing or reviewing unit tests, integration tests, e2e tests, or BDD scenarios; when the user mentions Jest, NestJS testing, Cucumber, test coverage, or test performance.
---

# Testes com Jest, NestJS Testing e Cucumber

Especialista em testes precisos e performáticos em aplicações NestJS usando Jest, `@nestjs/testing` e Cucumber (BDD).

## Quando usar esta skill

- Escrever ou revisar testes unitários (`.spec.ts`, `.test.ts`) ou de integração
- Configurar ou otimizar Jest / NestJS Testing Module
- Introduzir ou manter cenários BDD com Cucumber
- Melhorar cobertura, precisão das asserções ou tempo de execução dos testes

## Princípios gerais

1. **Precisão**: testar comportamento e contratos, não implementação; mocks mínimos e explícitos.
2. **Performance**: compilar o módulo uma vez por `describe`, reutilizar instâncias, evitar I/O e timers desnecessários.
3. **Isolamento**: cada teste deve ser independente; não depender de ordem de execução ou estado global.

---

## Jest — precisão e performance

### Estrutura e nomenclatura

- Um arquivo de teste por unidade: `*.spec.ts` ao lado do código.
- `describe` para o componente; `describe` aninhados para método ou cenário.
- Nomes que descrevem o comportamento: "should return 404 when order is not found" em vez de "test findOrder".

### Asserções

- Preferir matchers específicos: `toEqual`, `toHaveBeenCalledWith`, `toThrow`, `resolves`/`rejects`.
- Evitar apenas `toBeTruthy()`; afirmar o valor exato quando importar.
- Para chamadas de função: `toHaveBeenCalledTimes(n)`, `toHaveBeenCalledWith(...)` e `toHaveBeenLastCalledWith(...)`.

```typescript
expect(service.findOrder(id)).resolves.toEqual(expectedOrder);
expect(mockRepo.save).toHaveBeenCalledTimes(1);
expect(mockRepo.save).toHaveBeenCalledWith(expect.objectContaining({ status: 'closed' }));
```

### Mocks

- Usar `jest.fn()` para funções e `jest.mock('module')` só quando necessário (ex.: módulos com side effects).
- Implementar apenas o que o teste usa; retornos explícitos por cenário.
- Preferir `jest.spyOn(instance, 'method')` quando precisar espiar um método real em parte do fluxo.

### Performance no Jest

- **maxWorkers**: limitar (ex.: 2–4) em CI para evitar sobrecarga; pode aumentar localmente.
- **cache**: manter `cacheDirectory` (ex.: `/tmp/jest-cache`) para reutilizar transformações.
- **testPathIgnorePatterns**: excluir pastas pesadas (ex.: integração/e2e) do run unitário.
- **isolatedModules** / **transform**: manter config enxuta; evitar transform de arquivos desnecessários.
- Evitar `--runInBand` em runs normais; usar só para debug ou testes que exigem serialização.

---

## NestJS Testing Module — testes unitários

### Setup mínimo

- Usar `Test.createTestingModule({ imports, providers, controllers })` e compilar uma vez por suíte quando possível.
- Substituir dependências por mocks com `overrideProvider` + `useValue` (ou `useFactory`) em vez de instanciar implementações pesadas.

```typescript
export interface RepoInterfaceMock {
  findOne: jest.Mock,
  save: jest.Mock
}

const sut = async () => {
  const mockRepoMock: RepoInterfaceMock = {
    findOne: jest.fn(),
    save: jest.fn(),
  };

  const moduleRef = await Test.createTestingModule({
    providers: [
      OrderService,
      { provide: OrderRepository, useValue: mockRepoMock },
    ],
  }).compile();

  const orderServiceMocked = moduleRef.get<OrderService>(OrderService);
  return { orderServiceMocked, mockRepoMock };
};

describe('OrderService', () => {
  let orderService: OrderService;
  let mockRepo: RepoInterfaceMock;

  beforeAll(async () => {
    const { orderServiceMocked, mockRepoMock } = await sut();

    orderService = orderServiceMocked;
    mockRepo = mockRepoMock;
  })

  it('should find order by id', async () => {
    mockRepo.findOne.mockResolvedValue({ id: '1', status: 'open' });

    const result = await orderService.findOrder('1');

    expect(result.status).toBe('open');
    expect(mockRepo.findOne).toHaveBeenCalledWith({ where: { id: '1' } });
  });
});
```

### Boas práticas

- **Compilar uma vez**: criar o módulo no `beforeAll` ou numa factory `sut()` reutilizada no `describe`; evitar `beforeEach` com `compile()` se não for estritamente necessário.
- **Tokens e injeção**: usar os tokens do projeto (ex.: `getModelToken`, `getConnectionToken`, constantes de provider) para substituir TypeORM/Mongoose/outros.
- **Mocks de módulos externos**: usar `@nestjs/testing` para providers; para módulos completos, `overrideModule` ou importar um módulo de teste que já use `useValue`/`useClass` fake.
- Não instanciar o serviço manualmente com `new Service(mockA, mockB)`; usar sempre o container de testes para refletir a árvore real de dependências.
- Manter arquivos de mocks e geração de valores isolados em uma pasta `__mocks__`.
- Criar testes para todos os casos de borda para cobrir o máximo do código com testes.
- Sempre limpar todos os mocks com o `afterEach` para que os testes não interfiram um no outro.

### Testes de controllers/resolvers (HTTP/GraphQL)

- Usar `NestApplication` + `supertest` para testes de integração de rota; para unitário, obter o controller com `moduleRef.get(Controller)` e chamar métodos diretamente, mockando serviços.
- Em testes de resolver GraphQL, mockar DataLoader e serviços; afirmar o formato do DTO retornado e as chamadas aos serviços.

---

## Cucumber (BDD)

### Papel no projeto

- Cucumber para cenários de aceite e fluxos de alto nível (e2e ou integração); Jest + NestJS Testing para unit e integração de módulos.
- Manter steps reutilizáveis e cenários legíveis para não-desenvolvedores.

### Estrutura sugerida

```
test/
  e2e/
    features/
      order-creation.feature
    steps/
      order.steps.ts
    support/
      world.ts
```

### Feature e steps

- Cenários em Gherkin: Given/When/Then; evitar passos que só repetem código (ex.: "When I call POST /orders" é ok; "When I set the variable X to 1" é ruído).
- Step definitions devem delegar para funções ou serviços de teste; não colocar lógica de negócio nos steps.

```gherkin
Feature: Order creation
  Scenario: Create order with valid payload
    Given an authenticated user
    And a valid cart with items
    When the user submits the order
    Then the order is created with status "pending"
    And the order appears in the user's order list
```

### Integração com NestJS

- Inicializar o app Nest (ou um módulo mínimo) no `BeforeAll`; usar o mesmo `TestingModule` ou `NestApplication` para todas as steps do run.
- Injetar serviços/repositórios no World ou em steps via container de testes para reutilizar mocks ou banco de teste.
- Para performance: reutilizar uma instância do app por suíte; limpar dados entre cenários (ex.: truncar tabelas) em vez de subir app por cenário.

### Performance e precisão

- Evitar cenários que dependem de tempo real (sleep); usar filas/events ou mocks de tempo.
- Dados de teste explícitos (ids, nomes) para asserções estáveis; evitar dados randômicos onde isso atrapalhe a leitura do cenário.


### Observação

Esperar sempre uma especificação por parte do usuário antes de criar testes usando o Cucumber (BDD).
---

## Checklist rápido

- [ ] Teste compila o módulo Nest uma vez por suíte quando possível
- [ ] Dependências externas substituídas por mocks com `useValue`/`useFactory`
- [ ] Asserções específicas (`toEqual`, `toHaveBeenCalledWith`) em vez de só truthy
- [ ] Sem I/O ou timers reais em testes unitários
- [ ] Jest com `maxWorkers` e `cacheDirectory` configurados
- [ ] Cenários Cucumber delegam para funções de teste; steps enxutos
- [ ] Nomes de testes descrevem o comportamento esperado
