---
name: refactoring-enterprise-patterns
description: >-
  Guides refactoring by applying enterprise layers (client, services, use cases,
  controllers, entities) and design patterns only when appropriate. Use when
  refactoring code, restructuring modules, or when the user asks for
  enterprise patterns, layered architecture, or design patterns in refactoring.
---

# Refactoring com Padrões Enterprise e de Design

## Quando aplicar

Use esta skill ao refatorar código quando o objetivo for ou envolver:
- Organização em camadas (client, services, use cases, controllers, entities)
- Regras de negócio ou integrações com APIs externas
- Nomenclatura de padrões (manter apenas nomes de padrões conhecidos)

---

## Ordem de prioridade das camadas enterprise

Ao refatorar, considerar **nesta ordem**:

### 1. Client / Connection

- **Quando**: Chamadas a APIs ou serviços externos (HTTP, gRPC, mensageria, etc.).
- **O que fazer**: Encapsular em um **client** ou **connection** dedicado. Nunca espalhar `fetch`, `axios`, `HttpClient` ou similares direto em controllers ou services.
- **Responsabilidade**: Abstrair URL, autenticação, serialização e erros da API externa; expor métodos de domínio (ex.: `getOrder(id)`, `createPayment(data)`).

### 2. Services

- **Quando**: Regras de negócio que cabem em poucas condições ou validações (até ~2 conditionais/validações simples).
- **O que fazer**: Manter a lógica em **services** (ex.: `OrderService`, `PaymentService`). Services orquestram clientes e entidades e aplicam regras diretas.

### 3. Use cases

- **Quando**: Regras de negócio com **mais de** duas conditionais ou validações, ou fluxos que coordenam vários services/entidades.
- **O que fazer**: Extrair para **use cases** (ex.: `PlaceOrderUseCase`, `RefundPaymentUseCase`). Use cases contêm o fluxo completo; services ficam para operações mais simples.

### 4. Controllers e Entities

- **Quando**: Só quando fizer sentido na arquitetura atual.
- **Controllers**: Recebem entrada (HTTP, CLI, etc.), delegam para use case ou service, devolvem resposta. Sem regras de negócio.
- **Entities**: Modelos de domínio com identidade e invariantes. Usar quando houver conceitos de domínio claros (Order, Payment, User, etc.).

---

## Design patterns

- **Regra**: Usar design patterns (Factory, Strategy, Repository, etc.) **apenas em cenários especiais** em que tragam benefício claro (extensibilidade, teste, redução de duplicação).
- **Não** introduzir padrões por nome só para “ficar mais enterprise”. Preferir código direto e legível quando um padrão não resolver um problema concreto.

---

## Nomenclatura de padrões

- **Regra**: Remover ou renomear qualquer padrão cujo nome **não corresponda** a um padrão conhecido (ex.: GoF, padrões enterprise amplamente documentados).
- **Manter** apenas nomes que refletem padrões famosos (ex.: Repository, Factory, Strategy, Adapter, Use Case, Service, Controller, Entity, Client).
- Ao encontrar classes/pastas com nomes genéricos ou inventados (ex.: “Manager”, “Handler” vago, “Processor” sem correspondência a padrão), refatorar para um nome que indique a responsabilidade real ou para um padrão reconhecido.

## Interfaces e Abstração

- Caso o projeto já possua padrões que implementam interfaces, criar interfaces separadas em pastas `interfaces` dentro de cada módulo e implementar estas interfaces.
- Caso a classe implemente uma interface, usar tokens criados com `Symbol` na injeção de dependência, já que o Nestjs não consegue usar interfaces no DI.

## Boas práticas

- Evitar criar classes grandes demais, sempre tentar manter o tamanho entre 50 a 150 linhas, mas não usar isso como regra absoluta.
- Evitar criar métodos públicos auxiliares.
- Evitar criar `Helpers` para isolar código, sempre tentar identificar a que se destina o código e criar um service específico para isso.
- Evitar fugir da estrutura já existente no projeto, e caso ela fira alguma das diretrizes definidas, sempre perguntar antes de alterar.

---

## Checklist de refatoração

Ao refatorar, percorrer:

1. **APIs externas** → estão em client/connection?
2. **Regras de negócio** → simples → service; complexas (3+ passos/validações) → use case?
3. **Controllers** → só entrada/saída, sem lógica de negócio?
4. **Entities** → usadas onde há conceitos de domínio com identidade?
5. **Design patterns** → só onde há cenário que justifique (extensibilidade, teste, clareza)?
6. **Nomes** → nenhum “padrão” com nome que não seja um padrão famoso; renomear ou remover.

---

## Resumo da prioridade

| Prioridade | Camada        | Uso principal                          |
|-----------|----------------|----------------------------------------|
| 1         | Client/Connection | Encapsular chamadas a APIs externas   |
| 2         | Services       | Regras de negócio simples (≤2 conditionais/validações) |
| 3         | Use cases      | Regras de negócio mais complexas ou fluxos compostos |
| Se necessário | Controllers | Entrada/saída, delegação              |
| Se necessário | Entities    | Modelos de domínio com identidade     |
| Especial  | Design patterns | Só quando o cenário justificar       |
