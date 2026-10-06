Critérios de Qualidade e Gates do Pipeline — BFF

1. Objetivo

Esta página estabelece os critérios de qualidade esperados durante o ciclo de desenvolvimento e os gates recomendados para promoção do BFF entre as diferentes etapas.

O objetivo é transformar a estratégia de testes em um processo de qualidade contínuo.

O princípio é:

Quanto mais cedo um defeito puder ser detectado, menor deve ser sua chance de alcançar UAT.

⸻

2. Visão do pipeline

Desenvolvimento
      │
      ▼
Pull Request
      │
      ▼
┌─────────────────┐
│ Análise Estática│
└────────┬────────┘
         ▼
┌─────────────────┐
│    Unitários    │
└────────┬────────┘
         ▼
┌─────────────────┐
│    Integração   │
└────────┬────────┘
         ▼
┌─────────────────┐
│   Funcionais    │
└────────┬────────┘
         │
         │ QUALITY GATE
         ▼
       Build
         │
         ▼
     Deploy UAT
         │
         ▼
┌─────────────────┐
│ Smoke / Health  │
└────────┬────────┘
         ▼
┌─────────────────┐
│ Integração UAT  │
└────────┬────────┘
         │
         │ QUALITY GATE
         ▼
┌─────────────────┐
│       E2E       │
└────────┬────────┘
         ▼
 Avaliação Release

⸻

3. Gate de Pull Request

Antes da integração do código, executar automaticamente as validações rápidas.

Validações recomendadas

Build

O projeto deve compilar corretamente.

Análise estática

Quando disponível:

* qualidade de código;
* vulnerabilidades;
* code smells;
* dependências vulneráveis;
* padrões definidos pelo projeto.

Testes Unitários

Todos os testes unitários obrigatórios devem passar.

Testes de Integração Local

Os testes necessários para validar componentes integrados devem passar.

Testes Funcionais

Quando o tempo de execução permitir, a regressão funcional deve ser executada antes da promoção.

Caso a regressão completa seja extensa, pode existir separação entre:

PR
 │
 └──► suíte rápida/crítica
Merge/Build
 │
 └──► regressão funcional completa

⸻

4. Gate pré-deploy UAT

Antes do deploy em UAT, o BFF deve ter sido validado nas camadas locais.

Critério recomendado:

Build
            ✓
Unitários
            ✓
Integração Local
            ✓
Funcionais
            ✓
Análise de código
            ✓
Artefato gerado
            ✓

Um defeito conhecido exclusivamente relacionado ao comportamento do BFF não deveria ser descoberto pela primeira vez em UAT quando poderia ter sido coberto localmente.

⸻

5. Gate pós-deploy UAT

Após o deploy, executar inicialmente uma validação rápida da aplicação.

Smoke Test

Objetivo:

Confirmar que o BFF foi implantado e está minimamente operacional.

Validar, conforme aplicabilidade:

* aplicação disponível;
* health check;
* endpoints críticos acessíveis;
* configuração carregada;
* autenticação funcionando;
* dependências críticas acessíveis.

Fluxo:

Deploy
  │
  ▼
Smoke
  │
  ├── FAIL → interromper validações
  │
  └── PASS
        │
        ▼
    Integração UAT

Não existe benefício em executar uma regressão extensa quando a aplicação sequer está operacional.

⸻

6. Gate de integração UAT

Após o smoke, executar a suíte de integração em UAT.

Objetivo:

Comprovar que o BFF implantado consegue operar com suas dependências reais.

Validar:

* principais happy paths;
* comunicação com downstreams;
* autenticação entre serviços;
* configurações;
* serialização/deserialização real;
* infraestrutura;
* integrações relevantes.

A falha deve ser analisada antes da promoção.

⸻

7. Gate E2E

E2E deve ocorrer após as camadas anteriores apresentarem estabilidade.

Unitário
   ✓
   │
Integração
   ✓
   │
Funcional
   ✓
   │
Smoke UAT
   ✓
   │
Integração UAT
   ✓
   │
   ▼
 E2E

Não utilizar E2E como primeira linha de detecção de defeitos.

⸻

8. Critérios mínimos por camada

Camada	Critério
Build	Compilação concluída
Unitário	Testes obrigatórios aprovados
Integração Local	Integrações internas aprovadas
Funcional	Regressão do comportamento do BFF aprovada
Smoke UAT	BFF operacional
Integração UAT	Integrações críticas aprovadas
E2E	Jornadas críticas aprovadas

⸻

9. Política para testes instáveis

Um teste automatizado instável não deve ser tratado automaticamente como defeito do produto.

Entretanto, também não deve ser ignorado indefinidamente.

Testes flaky devem:

1. ser identificados;
2. ter sua causa investigada;
3. possuir responsável pela correção;
4. ser estabilizados;
5. voltar ao gate quando confiáveis.

A recorrência de testes instáveis deve ser acompanhada como indicador de saúde da suíte.

⸻

10. Falhas por camada

Unitário falhou

Principal investigação:

Código / regra interna

Integração local falhou

Principal investigação:

Integração entre componentes
Adapters
Clients
Serialização
Configuração local

Funcional local falhou

Principal investigação:

Comportamento do BFF
Regressão funcional
Transformação
Agregação
Tratamento de erros

Local passou e UAT falhou

Principal investigação:

Configuração
Autenticação
Conectividade
Compatibilidade real
Massa
Infraestrutura
Deploy
Dependência externa

Camadas anteriores passaram e E2E falhou

Principal investigação:

Interação entre sistemas
Estado da jornada
Integração entre domínios
Dependências adicionais

⸻

11. Critérios para bloqueio

Uma falha deve bloquear promoção quando representar risco incompatível com a próxima etapa.

Exemplos:

Bloqueadores

* build quebrado;
* testes críticos falhando;
* regressão funcional confirmada;
* BFF indisponível;
* integração crítica indisponível;
* jornada crítica quebrada;
* vulnerabilidade crítica conforme política de segurança.

Avaliação necessária

Situações como:

* teste flaky conhecido;
* dependência externa temporariamente indisponível;
* cenário não crítico;
* problema conhecido e formalmente aceito.

A decisão deve considerar impacto e risco, e não simplesmente a quantidade absoluta de testes que passaram.

⸻

12. Métricas recomendadas

A evolução da estratégia pode ser acompanhada por indicadores como:

Taxa de aprovação por camada
Tempo de execução
Quantidade de testes
Flakiness
Defeitos encontrados localmente
Defeitos encontrados em UAT
Defeitos encontrados em E2E
Defeitos escapados
Tempo médio de diagnóstico
Tempo médio de correção da suíte

Uma métrica particularmente útil é:

Em qual camada os defeitos estão sendo encontrados?

Se grande parte dos defeitos funcionais do BFF continuar sendo encontrada somente em UAT, isso pode indicar uma oportunidade de melhorar a cobertura local.

⸻

13. Evolução esperada

A maturidade desejada é sair de:

Desenvolve
    │
    ▼
Deploy UAT
    │
    ▼
Testa tudo
    │
    ▼
Descobre problemas

para:

Desenvolve
    │
    ▼
Valida localmente
    │
    ▼
Pipeline comprova comportamento
    │
    ▼
Deploy UAT
    │
    ▼
Comprova integração
    │
    ▼
E2E comprova jornada

⸻

14. Definition of Done — Qualidade do BFF

Como referência, uma entrega pode ser considerada pronta do ponto de vista de testes quando, conforme aplicabilidade:

* testes unitários foram implementados;
* testes de integração foram implementados;
* testes funcionais foram implementados;
* cenários negativos relevantes estão cobertos;
* tratamento de erros downstream está coberto;
* regressão local está aprovada;
* deploy em UAT está operacional;
* integrações críticas em UAT estão aprovadas;
* jornadas E2E impactadas estão aprovadas;
* evidências necessárias estão disponíveis;
* riscos residuais são conhecidos e aceitos.

⸻

15. Princípio final

O pipeline deve impedir que defeitos que podem ser identificados localmente avancem desnecessariamente para UAT. UAT deve funcionar como gate de integração, enquanto E2E deve funcionar como gate das jornadas críticas de negócio.

O resultado esperado é um processo com feedback mais rápido, menor custo de diagnóstico e maior confiança para promoção das entregas.