# Padrões de Estrutura de Teste

Todos os testes devem possuir um bloco `describe` geral para o alvo testado e blocos `describe` menores para cada função e seus casos de borda.
Além disso devem sempre ser incluídos os blocos `beforeAll` e `afterEach` para criação de sut e limpeza de mocks, respectivamente. 
Sendo assim, o template básico de um arquivo de testes é o seguinte:

```javascript

describe('TargetService', () => {
    let sutService: SutType
    let dependency: DependencyMock

    beforeAll(() => {
        // create sut
    })

    afterEach(() => {
        jest.clearAllMocks()
    })

    describe('exampleFunction', () => {
        it('should do something important', () => {
            // Tests
        })
    })
})

```

# Padrões de escrita de mocks

Sempre isole a criação de mocks do arquivo de testes para melhor leitura do teste e organização. 
A criação de mocks deve ser realizada em um arquivo dentro da pasta `__mocks__` que deverá se encontrar dentro do mesmo módulo em que os testes estão sendo escritos.
Por exemplo, se existe um módulo chamado `test` e dentro dele existem pastas chamadas `services`, `controllers` e `helpers` e em cada uma delas existem testes, os mocks deles devem ser guardados dentro de `test/__mocks__`.
Sempre crie mocks do zero, exceto se o projeto possuir mocks globais a serem usados. Além dos mocks, crie interfaces que vão criar tipos para estes mocks, assim elas podem ser utilizados no teste principal.

Segue abaixo um exemplo de criação de sut:

```javascript

export interface DependencyMock {
    testingMethod: jest.Mock
}

export const sut = async () => {
  const dependencyMock: DependencyMock = {
    testingMethod: jest.fn()
  }

  const moduleRef = await Test.createTestingModule({
    providers: [
      {
        provide: Dependency,
        useValue: dependencyMock,
      },
      TestService,
    ],
  }).compile();

  const testServiceMocked = moduleRef.get<TestService>(TestService);

  return {
    testServiceMocked,
    dependencyMock,
  };
};

```

# Padrões de escrita de testes

Sempre crie testes isolados uns dos outros, sendo isolados em sua definição: que não compartilham mocks, então sempre prefira criar mocks de uso único; que não dependem da instanciação do sut pra funcionar; que não dependem da ordem que os testes são executados para funcionar.
Sempre verifique a saída da função testada, as chamadas internas de suas dependências e com quais valores estão sendo chamadas.
Segue abaixo um exemplo de implementação de teste:

```javascript

it('should do something important', () => {
    dependencyMock.testingMethod.mockResolvedValueOnce({ ok: true })

    const result = testServiceMocked.doSomething()

    expect(dependencyMock.testingMethod).toBeCalledWith({
        user: 'mats',
        cancel: true
    })
    expect(result).toBe(true)
})

```