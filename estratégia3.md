Matriz de Cobertura — Local x UAT x E2E

1. Objetivo

Esta página define onde cada tipo de risco e comportamento do BFF deve ser validado.

Seu objetivo é evitar:

* duplicação desnecessária;
* regressões excessivamente grandes em UAT;
* dependência excessiva do ambiente integrado;
* sobreposição entre testes funcionais e E2E;
* cenários sem responsabilidade claramente definida.

⸻

2. Princípio

A decisão deve ser baseada no risco que o teste pretende comprovar, e não apenas na possibilidade técnica de executá-lo em determinado ambiente.

A pergunta principal é:

Qual é a camada mais baixa capaz de comprovar este risco com confiança?

Sempre que possível, utilizar essa camada.

⸻

3. Visão geral

LOCAL
──────────────────────────────
"O BFF funciona?"
Unitário
Integração
Funcional
Maior cobertura
Maior isolamento
Maior velocidade
UAT
──────────────────────────────
"O BFF funciona integrado?"
Serviços reais
Configuração real
Infraestrutura real
Cobertura seletiva
E2E
──────────────────────────────
"A jornada funciona?"
Múltiplos sistemas
Fluxos críticos
Menor quantidade de testes

⸻

4. Matriz de cobertura

Risco / Comportamento	Unitário	Integração Local	Funcional Local	UAT	E2E
Regra interna do BFF	✓				
Validação de entrada	✓		✓		
Transformação de payload	✓	✓	✓		
Serialização/deserialização		✓	✓	✓	
Agregação de APIs		✓	✓	✓*	
Construção do request downstream		✓	✓	✓*	
Headers downstream		✓	✓	✓*	
Downstream retorna 400			✓		
Downstream retorna 404			✓		
Downstream retorna 500			✓		
Downstream retorna 503			✓		
Downstream timeout			✓		
Response downstream inválido			✓		
Response parcial			✓		
Retry	✓	✓	✓		
Fallback	✓	✓	✓		
Circuit breaker		✓	✓		
Status code do BFF			✓	✓	
JSON Schema do BFF			✓	✓*	
Conteúdo do response			✓	✓*	
Comunicação real BFF → API				✓	
Endpoint configurado em UAT				✓	
Autenticação entre serviços				✓	
Certificados / secrets				✓	
Integração real Kafka				✓	✓*
Integração real banco				✓	✓*
Correlation/trace entre serviços				✓	✓*
Happy path principal	✓*	✓*	✓	✓	✓*
Jornada completa de negócio					✓
Integração entre múltiplos sistemas				✓*	✓
Comportamento do frontend/canal					✓*

✓ = cobertura principal.

✓* = cobertura complementar quando aplicável.

⸻

5. Sobreposição intencional

A existência de um mesmo fluxo em diferentes camadas não significa necessariamente duplicação.

Exemplo:

             HAPPY PATH
Funcional Local
      │
      └──► O comportamento do BFF está correto?
UAT
      │
      └──► O BFF consegue executar esse fluxo
           utilizando dependências reais?
E2E
      │
      └──► A jornada de negócio completa funciona?

A entrada pode ser semelhante, mas o risco comprovado é diferente.

⸻

6. Exemplo prático

Considere:

GET /bff/clientes/{id}/resumo

O BFF consulta:

GET API Cliente
GET API Conta
GET API Limites

Funcional Local

Cobrir:

Cliente 200 + Conta 200 + Limites 200
Cliente 404
Conta 500
Limites 500
Limites timeout
Conta sem determinado campo
Transformação do Cliente
Agregação do response
Validação do schema final

Essa camada pode possuir dezenas de combinações.

⸻

UAT

Não é necessário reproduzir todas essas combinações.

Cobrir, por exemplo:

BFF
 │
 ├──► Cliente UAT
 ├──► Conta UAT
 └──► Limites UAT
Todos respondem utilizando integrações reais.

O objetivo é comprovar:

* conectividade;
* autenticação;
* configuração;
* compatibilidade dos payloads;
* interpretação dos responses reais;
* integração.

⸻

E2E

O mesmo BFF pode participar de algo maior:

Usuário
   │
   ▼
Canal
   │
   ▼
BFF
   │
   ├──► Cliente
   ├──► Conta
   └──► Limites
           │
           ▼
       outros sistemas

Aqui o teste comprova a jornada do usuário.

⸻

7. Anti-pattern: regressão espelhada

Evitar:

Funcional Local
150 testes
     │
     ▼
UAT
150 testes
     │
     ▼
E2E
150 testes

Esse modelo aumenta:

* custo;
* tempo;
* manutenção;
* flakiness;
* dependência de massa;
* dificuldade de diagnóstico.

Sem necessariamente aumentar proporcionalmente a confiança.

⸻

8. Modelo desejado

               E2E
              █████
             UAT
          ███████████
      FUNCIONAL LOCAL
██████████████████████████
       INTEGRAÇÃO
████████████████████████████
        UNITÁRIO
████████████████████████████████

Os números exatos dependem da complexidade de cada BFF.

O importante é a distribuição da responsabilidade.

⸻

9. Checklist para novos cenários

Antes de adicionar um cenário em UAT ou E2E, perguntar:

O comportamento já está coberto localmente?

Se não estiver, avaliar primeiro a cobertura local.

O cenário depende realmente de uma integração real?

Se não depender, priorizar local.

UAT adiciona algum risco novo que o teste local não consegue comprovar?

Se não adicionar, avaliar se o teste em UAT é realmente necessário.

O cenário representa uma jornada de negócio completa?

Se não representar, provavelmente não é E2E.

Uma falha nesse teste permitiria identificar claramente qual risco não foi coberto pelas camadas anteriores?

Essa pergunta ajuda a identificar testes redundantes.

⸻

10. Regra final

Local comprova comportamento. UAT comprova integração. E2E comprova jornada.

Essa separação deve orientar a criação e manutenção da regressão dos BFFs.