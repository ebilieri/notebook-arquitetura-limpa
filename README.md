# 🏛️ Arquitetura Limpa: O que é e como implementar

## 🎯 Contexto e Objetivos
Este caderno temático explora o conceito de **Arquitetura Limpa (Clean Architecture)** — um padrão de desenvolvimento de software que visa separar as regras de negócio das tecnologias e frameworks externos.

**Objetivos de Estudo:**
- Entender os princípios básicos da restrição de dependências (regra de dependência de fora para dentro).
- Visualizar a implementação prática de uma API em Java (Spring Boot) focada no gerenciamento de Pedidos, Clientes e Produtos.
- Aprender a estruturar física e logicamente projetos utilizando a abordagem Maven Multi-modules para forçar o respeito arquitetural pelo compilador.

---

## 📚 Curadoria de Fontes
Durante o estudo e alimentação do NotebookLM, os seguintes materiais sobre padrões arquiteturais garantiram o contexto base:

1. **[The Clean Architecture - Robert C. Martin (Uncle Bob)](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)** - O artigo original (aberto) que define os anéis, a essência do domínio independente e a regra de dependência.
2. **[Hexagonal Architecture - Alistair Cockburn](https://alistair.cockburn.us/hexagonal-architecture/)** - Base conceitual (Ports and Adapters), pilar fundamental usado na Arquitetura Limpa para isolamento.
3. **[Documentação do Spring e Padrões de Design](https://spring.io/projects/spring-boot)** - Material oficial usado como parâmetro de como aplicar e delegar propriedades em frameworks invasivos para a camada "Infraestrutura".
4. **Artigos Práticos de DDD e Arquitetura Limpa em Java (Ex: Baeldung/InfoQ)** - Utilizados para compor o escopo do ecossistema moderno na conversão de domínios em microsserviços.

---

## 🛠️ Engenharia de Prompts e "Cicatrizes"

O raciocínio da exploração via IA foi de abstraí-lo do conceito teórico para a real barreira técnica que o código impõe.

### 📍 Teste 1: A Estrutura Lógica em Java
> **Prompt:** *"mostre um exemplo de como implementar uma arquitetura limpa para uma api de pedidos em java. A api precisa gerenciar pedidos, clientes e produtos"*

**Resultados Obtidos (IA):** 
A resposta desenhou as 4 camadas concêntricas (Domain, Use Cases, Interface Adapters, Infrastructure), criando a independência do framework.
A IA gerou as classes de forma isolada, como:
- `Pedido`: Uma entidade Java pura, sem anotações de dependência de banco de dados (`JPA`).
- `PedidoRepositoryGateway`: Uma port/interface residindo na lógica.
- `CriarPedidoInteractor`: O caso de uso (Use Case) que orquestra tudo.
- `PedidoJpaAdapter`: Detalhe da infraestrutura que sabe persistir usando os componentes do Spring Data.

**Troubleshooting (A Cicatriz):** 
*O insight do processo:* Analisando a primeira resposta, entendi que o código gerado em suas pastas (packages) parece perfeito. No entanto, qual a garantia que eu tenho que o próximo desenvolvedor não vai importar, por impulso, uma anotação `@Entity` dentro da classe de domínio? Uma camada lógica de packages não barra dependências indesejadas, era necessário proteger fisicamente. Daí surgiu a segunda pergunta.

### 📍 Teste 2: Evitando violações através da arquitetura de build
> **Prompt:** *"Gostaria de ver como estruturar as pastas e módulos do Maven para garantir que essas regras sejam respeitadas pelo compilador"*

**Resultados Obtidos (IA):**
Evoluímos de pastas para um **Projeto Multi-módulo no Maven**.
A orientação correta devolvida pela Inteligência Artificial definiu barreiras através de projetos acoplados:
1. `core` (ou `domain`): Camada mais central (só entidades POJOs). Sem dependências externas.
2. `usecase`: Depende apenas do `core`.
3. `adapters`: Tradutores. Dependem de `usecase` e `core`.
4. `infra`: Camada do Spring Boot e JDBC/JPA, que dependerá dos demais para injetar e fazer as conexões.

**O Valor Tirado:** Isso assegura que se houver uma distração e alguém importar classes de banco num `usecase` ou `core`, o próprio Maven falhará a compilação, atuando como fiscal das regras.

---

## 📖 Miniguia de Estudo (Consolidação)

### 📌 Resumo Estruturado do Assunto
A **Arquitetura Limpa** consiste numa série de práticas para construir softwares sustentáveis dividindo-o em camadas como "uma cebola". A diretriz vital do sistema é a **Regra de Dependência**: "As dependências do código-fonte só podem apontar em direção ao centro".
- O **Domínio** rege o coração (independente de web, bancos ou tecnologias).
- Os **Casos de Uso** representam a camada da aplicação em si, coordenando o negócio.
- Os **Adaptadores e a Infraestrutura** garantem a "cola" (o banco de dados, os endpoints REST JSON, as views Web). Se decidir mudar o banco do MySQL para um MongoDB, nada no núcleo lógico precisará ser reescrito, apenas adaptadores externos.

### 📌 Glossário Clássico do Padrão
*   **Entity / Entidade Pura:** As verdadeiras regras corporativas. Um objeto/modelo focado apenas no negócio e matemática.
*   **Interactor (Caso de Uso):** As regras da aplicação. Coordena transações de fluxo da entidade sem saber o destino dessas saídas no banco.
*   **Gateway / Port:** Contratos (Interfaces) definidos nos casos de uso que as interfaces externas e a infraestrutura devem cumprir (Inversão da Dependência).
*   **Adapter:** São classes na periferia do projeto que traduzem algo de uso restrito do seu sistema (Entities/Use Cases) num formato externo apropriado (Controllers Web ou Repositórios JPA).

### 📌 Conjunto de Prompts Reutilizáveis
Use esses prompts em suas ferramentas de IA ou no próprio NotebookLM para praticar e revisar contextos dentro da Arquitetura Limpa:
1. *"Analise este trecho de código em Java da minha camada de [NOME DA CAMADA] e identifique se estou infringindo alguma Regra de Dependência estrutural da Arquitetura Limpa."*
2. *"Dada a proposta do isolamento Clean Architecture, como você escreveria um teste unitário em JUnit para o Use Case 'CriarPedido' sem injetar instâncias reais de banco de dados nem levantar os containers externos do Spring?"*
3. *"Quero criar um novo Adaptador para comunicar meu projeto com um serviço Amazon S3. Atuando como senior, crie o código de Interface/Port do meu Use Case e a Implementação respectiva gerada na camada de adapters/infraestrutura separada."*
