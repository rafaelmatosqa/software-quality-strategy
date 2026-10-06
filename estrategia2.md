Padrão de Testes Funcionais — BFF

1. Objetivo

Este documento define o padrão para criação, automação e manutenção dos testes funcionais das APIs dos BFFs.

Os testes funcionais representam uma das principais camadas de regressão do BFF e devem validar seu comportamento através de sua interface HTTP, mantendo as dependências downstream controladas sempre que possível.

O objetivo principal é responder:

O BFF apresenta o comportamento esperado para as diferentes condições de entrada e de resposta das suas dependências?

⸻

2. Posicionamento na estratégia

Os testes funcionais fazem parte da camada local/pipeline:

Unitário
   │
   ▼
Integração
   │
   ▼
FUNCIONAL  ◄── Escopo desta página
   │
   ▼
Deploy UAT
   │
   ▼
Integração UAT
   │
   ▼
E2E

Essa camada deve possuir cobertura significativamente maior que os testes executados em UAT e E2E.

⸻

3. Arquitetura

A arquitetura recomendada é:

                   HTTP
Teste ─────────────────────────► BFF
                                 │
                      ┌──────────┼──────────┐
                      │          │          │
                      ▼          ▼          ▼
                    Stub       Stub       Stub
                   API A      API B      API C

O BFF deve ser executado utilizando sua implementação real.

As dependências externas devem ser substituídas por mocks/stubs sempre que possível.

Isso permite controlar completamente as condições necessárias para o cenário.

⸻

4. Princípio de cobertura

A regressão funcional deve validar:

Comportamentos que pertencem ou impactam diretamente a responsabilidade do BFF.

Não deve ser objetivo reproduzir integralmente os testes funcionais dos serviços downstream.

Exemplo:

API Cliente possui 150 cenários funcionais.
BFF utiliza:
GET /clientes/{id}

O BFF não precisa reproduzir os 150 testes da API Cliente.

O BFF precisa comprovar:

* que envia corretamente a requisição;
* que interpreta corretamente o response;
* que utiliza corretamente os campos necessários;
* que transforma os dados quando necessário;
* que trata adequadamente os erros relevantes;
* que apresenta o resultado esperado ao consumidor.

⸻

5. Categorias de testes funcionais

5.1 Happy Path

Validar o comportamento esperado quando todas as condições são válidas.

Exemplo:

Given
API Cliente retorna HTTP 200
And
API Conta retorna HTTP 200
When
o consumidor consulta o BFF
Then
BFF retorna HTTP 200
And
agrega corretamente as informações

⸻

5.2 Validação de entrada

Validar:

* campos obrigatórios;
* campos opcionais;
* formatos;
* tamanho mínimo/máximo;
* enumerações;
* valores inválidos;
* headers;
* path parameters;
* query parameters;
* request body.

⸻

5.3 Transformação de dados

Validar qualquer transformação realizada pelo BFF.

Exemplo downstream:

{
  "customer_id": "123",
  "full_name": "João Silva"
}

Response do BFF:

{
  "id": "123",
  "name": "João Silva"
}

O teste deve garantir que o mapeamento permaneça correto.

⸻

5.4 Agregação

Quando o BFF consulta diferentes serviços:

                 ┌────► Cliente
                 │
BFF ─────────────┼────► Conta
                 │
                 └────► Limites

o teste deve validar o resultado da composição.

Devem ser consideradas combinações relevantes entre os responses.

⸻

6. Testes de erro

As dependências controladas permitem simular erros de forma determinística.

Cobrir conforme aplicabilidade:

HTTP 400
HTTP 401
HTTP 403
HTTP 404
HTTP 409
HTTP 422
HTTP 429
HTTP 500
HTTP 502
HTTP 503
HTTP 504

O objetivo não é testar todos os status indiscriminadamente.

Devem ser testados aqueles que possam provocar comportamento relevante no BFF.

⸻

7. Resiliência

Quando o BFF possuir mecanismos de resiliência, testar:

* timeout;
* retry;
* fallback;
* circuit breaker;
* indisponibilidade;
* respostas parciais;
* lentidão downstream.

Exemplo:

API A → 200
API B → timeout
API C → 200
              ↓
Qual deve ser o comportamento do BFF?

O comportamento esperado deve estar explicitamente definido e automatizado.

⸻

8. Validações do response

Quando houver response, validar conforme aplicabilidade:

Status Code

Garantir que o status retornado corresponde ao comportamento esperado.

JSON Schema

Garantir a estrutura e os tipos do contrato exposto pelo BFF.

Conteúdo

Validar:

* valores;
* campos obrigatórios;
* campos opcionais;
* transformação;
* agregação;
* regras funcionais;
* ausência de informações indevidas.

A validação não deve se limitar ao HTTP Status Code.

⸻

9. Headers e contexto

Validar quando aplicável:

* Authorization;
* correlation-id;
* trace-id;
* headers obrigatórios;
* propagação de contexto;
* headers enviados aos downstreams;
* content-type;
* accept.

⸻

10. Segurança funcional

Quando a responsabilidade estiver no BFF, considerar:

Sem autenticação
        ↓
       401
Autenticado sem permissão
        ↓
       403

Também devem ser consideradas situações de acesso indevido entre usuários ou contextos diferentes quando aplicável.

Testes especializados de segurança podem complementar essa cobertura.

⸻

11. Cenários que devem permanecer fora desta camada

Não é responsabilidade dos testes funcionais locais comprovar:

* disponibilidade real de serviços em UAT;
* configuração real do ambiente;
* certificados de UAT;
* conectividade de rede entre ambientes;
* comportamento completo dos downstreams;
* jornada completa do negócio;
* integração real entre todos os sistemas.

Esses riscos pertencem às camadas superiores.

⸻

12. Massa de testes

Sempre que possível, os testes funcionais devem criar e controlar seus próprios dados.

Mocks/stubs devem permitir responses específicos para cada cenário.

Evitar dependência de:

* massa compartilhada;
* execução anterior;
* ordem dos testes;
* estado externo;
* dados alterados manualmente.

⸻

13. Independência dos testes

Cada teste deve poder executar:

sozinho
   +
em qualquer ordem
   +
repetidamente

sem depender de outro teste.

Isso aumenta a estabilidade da regressão e facilita execução paralela.

⸻

14. Automação

Para projetos Java, a implementação pode utilizar a stack definida pelo projeto, como:

Java
RestAssured
TestNG
Hamcrest
WireMock ou tecnologia equivalente

A arquitetura dos testes deve manter separação entre:

Test
 │
 ├── DataFactory
 ├── Clients
 ├── Models
 ├── Validators
 ├── Mocks/Stubs
 └── Utils

evitando concentrar toda a lógica diretamente nas classes de teste.

⸻

15. Execução

A suíte funcional deve ser adequada para execução automática no pipeline.

Exemplo:

Pull Request
     │
     ▼
Unitários
     │
     ▼
Integração
     │
     ▼
Funcionais
     │
     ▼
   Build
     │
     ▼
Deploy UAT

Falhas nessa camada devem impedir a promoção quando representarem regressão do comportamento esperado do BFF.

⸻

16. Critério de conclusão

Uma funcionalidade do BFF pode ser considerada adequadamente coberta localmente quando:

* principais happy paths estão automatizados;
* regras do BFF estão cobertas;
* transformações estão cobertas;
* agregações relevantes estão cobertas;
* erros relevantes dos downstreams estão simulados;
* mecanismos de resiliência estão cobertos;
* status code está validado;
* schema está validado;
* conteúdo relevante está validado.

⸻

17. Princípio final

A regressão funcional local deve ser a principal responsável por comprovar o comportamento do BFF. Dependências controladas devem ser utilizadas para aumentar cobertura, velocidade e determinismo, deixando para UAT os riscos que realmente dependem da integração entre componentes reais.