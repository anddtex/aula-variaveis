# Aula-variaveis
Repositorio do curso de DevOps PRO sobre variaveis de ambiente

# GitHub Actions — Variáveis de Ambiente

> Guia técnico sobre variáveis de ambiente no GitHub Actions, elaborado com base na transcrição da aula.

## 📌 Visão geral

Workflows podem conter valores fixos diretamente no código, mas essa abordagem reduz o reaproveitamento e dificulta a manutenção.

Exemplo de hardcode:

```yaml
run: echo "valor xpto"
```

Uma abordagem mais flexível é utilizar variáveis de ambiente:

```yaml
env:
  APP_ENV: "production"
```

e consumi-las no shell:

```yaml
run: echo "$APP_ENV"
```

A aula apresenta três níveis de escopo para variáveis de ambiente:

1. **Workflow**
2. **Job**
3. **Step/Action**

---

# 🧩 Variável no nível do Workflow

Quando declarada no nível do workflow, a variável pode ser utilizada pelos jobs e steps daquele workflow.

```yaml
name: Variáveis de Ambiente

on:
  workflow_dispatch:

env:
  ENV_WORKFLOW: "valor workflow"

jobs:
  teste:
    runs-on: ubuntu-latest

    steps:
      - name: Exibir variável
        run: echo "$ENV_WORKFLOW"
```

Estrutura:

```text
Workflow
│
├── Job 1
│   ├── Step 1
│   └── Step 2
│
└── Job 2
    ├── Step 1
    └── Step 2
```

A variável definida no workflow é a opção adequada quando o valor precisa ser compartilhado.

---

# 🏗️ Variável no nível do Job

Também é possível definir uma variável somente para determinado job:

```yaml
jobs:
  teste:
    runs-on: ubuntu-latest

    env:
      ENV_JOB: "valor job"

    steps:
      - name: Exibir variável
        run: echo "$ENV_JOB"
```

Um segundo job pode possuir outro valor:

```yaml
jobs:
  teste-1:
    runs-on: ubuntu-latest
    env:
      ENV_JOB: "valor job 1"

    steps:
      - run: echo "$ENV_JOB"

  teste-2:
    runs-on: ubuntu-latest
    env:
      ENV_JOB: "valor job 2"

    steps:
      - run: echo "$ENV_JOB"
```

O valor fica associado ao respectivo job.

---

# 🎯 Variável no nível do Step/Action

Também é possível limitar a variável a um step específico:

```yaml
jobs:
  teste:
    runs-on: ubuntu-latest

    steps:
      - name: Executar
        env:
          ENV_ACTION: "valor action"
        run: echo "$ENV_ACTION"
```

Nesse caso, o valor não deve ser considerado disponível nos demais steps.

Exemplo:

```yaml
steps:
  - name: Step 1
    env:
      TEMP_VALUE: "123"
    run: echo "$TEMP_VALUE"

  - name: Step 2
    run: echo "$TEMP_VALUE"
```

No `Step 2`, a variável definida exclusivamente no `Step 1` não estará disponível.

---

# 📊 Comparação dos escopos

| Escopo | Abrangência |
|---|---|
| Workflow | Workflow inteiro |
| Job | Steps daquele job |
| Step/Action | Apenas aquele step |

### Regra prática

> **Defina a variável no menor escopo que atenda à necessidade.**

Se vários jobs precisam dela, utilize workflow.

Se apenas um job precisa dela, utilize job.

Se apenas uma operação precisa dela, utilize step.

---

# 🔄 Sobrescrita por escopo

A aula demonstra que variáveis podem ter valores diferentes em escopos diferentes.

Exemplo:

```yaml
env:
  APP_ENV: "workflow"

jobs:
  teste:
    runs-on: ubuntu-latest

    env:
      APP_ENV: "job"

    steps:
      - name: Teste
        env:
          APP_ENV: "step"
        run: echo "$APP_ENV"
```

O valor definido no escopo mais específico prevalece para aquele contexto.

Conceitualmente:

```text
Workflow
APP_ENV = workflow
       │
       ▼
Job
APP_ENV = job
       │
       ▼
Step
APP_ENV = step
```

Isso permite que um valor geral seja definido no workflow e, quando necessário, seja substituído em um job ou step específico.

---

# 🖥️ Utilização no Shell

No ambiente Linux, a variável pode ser utilizada com `$`:

```bash
echo "$ENV_WORKFLOW"
```

Exemplo:

```yaml
env:
  APPLICATION_NAME: "api"

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Exibir aplicação
        run: echo "$APPLICATION_NAME"
```

A utilização segue o padrão de variáveis de ambiente de um ambiente Linux tradicional.

---

# ❌ Hardcode vs. Parametrização

### Hardcode

```yaml
run: echo "production"
```

O valor está diretamente na lógica do workflow.

### Parametrização

```yaml
env:
  ENVIRONMENT: "production"

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Ambiente
        run: echo "$ENVIRONMENT"
```

A segunda abordagem facilita manutenção e reutilização.

---

# 🔎 Variáveis predefinidas do GitHub

O GitHub Actions possui diversas variáveis de ambiente disponibilizadas automaticamente.

Antes de criar uma variável própria, é importante verificar a documentação oficial para descobrir se o GitHub já fornece a informação necessária.

Exemplos de utilização:

```yaml
steps:
  - name: Informações do workflow
    run: |
      echo "$GITHUB_RUN_NUMBER"
      echo "$GITHUB_REPOSITORY"
```

A recomendação da aula é consultar a lista oficial antes de criar novas variáveis.

> Isso evita recriar uma informação que já está disponível no próprio GitHub Actions.

---

# ⚠️ Não utilizar o prefixo `GITHUB_`

O prefixo `GITHUB_` é utilizado pelo próprio GitHub para suas variáveis.

Portanto, evite criar variáveis próprias como:

```yaml
env:
  GITHUB_MEUBANCO: "valor"
```

Prefira:

```yaml
env:
  DATABASE_HOST: "localhost"
```

ou:

```yaml
env:
  MY_DATABASE: "database"
```

---

# 📐 Convenção de nomenclatura

A prática recomendada na aula é utilizar:

- Letras maiúsculas.
- `_` para separar palavras.

### Exemplos

```yaml
env:
  APP_NAME: "api"
  APP_ENV: "production"
  DATABASE_HOST: "localhost"
  DATABASE_PORT: "5432"
  NODE_VERSION: "18"
```

Esse padrão facilita a leitura e mantém consistência entre os workflows.

---

# 🧪 Workflow completo

```yaml
name: Environment Variables

on:
  workflow_dispatch:

env:
  PROJECT_NAME: "devops-project"
  DEFAULT_ENVIRONMENT: "development"

jobs:
  build:
    runs-on: ubuntu-latest

    env:
      BUILD_TYPE: "release"

    steps:
      - name: Informações gerais
        run: |
          echo "Projeto: $PROJECT_NAME"
          echo "Ambiente: $DEFAULT_ENVIRONMENT"
          echo "Build: $BUILD_TYPE"

      - name: Configuração específica
        env:
          TOOL_VERSION: "1.0"
        run: |
          echo "Ferramenta: $TOOL_VERSION"

  test:
    runs-on: ubuntu-latest

    env:
      TEST_TYPE: "integration"

    steps:
      - name: Executar testes
        run: |
          echo "Projeto: $PROJECT_NAME"
          echo "Ambiente: $DEFAULT_ENVIRONMENT"
          echo "Teste: $TEST_TYPE"
```

Neste exemplo:

- `PROJECT_NAME` está no workflow.
- `DEFAULT_ENVIRONMENT` está no workflow.
- `BUILD_TYPE` pertence ao job `build`.
- `TEST_TYPE` pertence ao job `test`.
- `TOOL_VERSION` pertence somente ao step específico.

---

# 🔐 Variáveis de ambiente e Secrets

Variáveis de ambiente não devem ser utilizadas para armazenar diretamente informações sensíveis.

Evite:

```yaml
env:
  DATABASE_PASSWORD: "minha-senha"
```

Para credenciais, utilize o mecanismo de **Secrets** do GitHub.

Exemplo:

```yaml
steps:
  - name: Utilizar token
    env:
      API_TOKEN: ${{ secrets.API_TOKEN }}
    run: |
      ./deploy.sh
```

Assim, o valor sensível permanece armazenado como secret e é disponibilizado ao processo por meio da variável de ambiente.

---

# 🔄 Variáveis + Inputs

A aula também destaca a possibilidade de combinar variáveis de ambiente com **inputs**.

Essa combinação permite construir workflows mais parametrizados:

```text
Input
  │
  ▼
Workflow
  │
  ▼
Environment Variable
  │
  ▼
Job
  │
  ▼
Step
```

Isso é especialmente útil para reaproveitar o mesmo workflow com diferentes parâmetros de execução.

---

# 🧠 Boas práticas DevOps

## 1. Evite hardcode

Evite:

```yaml
run: echo "production"
```

Prefira:

```yaml
env:
  ENVIRONMENT: "production"
```

---

## 2. Utilize o menor escopo possível

Se somente um step precisa da variável:

```yaml
steps:
  - name: Processar
    env:
      TEMP_VALUE: "123"
    run: echo "$TEMP_VALUE"
```

Não há necessidade de promovê-la para o nível global do workflow.

---

## 3. Reutilize variáveis do GitHub

Antes de criar uma nova variável, consulte as variáveis predefinidas.

---

## 4. Não utilize `GITHUB_`

Mantenha esse prefixo reservado às variáveis do GitHub Actions.

---

## 5. Padronize nomes

Prefira:

```text
UPPER_CASE
```

Exemplos:

```text
DATABASE_HOST
DATABASE_PORT
APPLICATION_NAME
NODE_VERSION
DEPLOY_ENVIRONMENT
```

---

## 6. Separe configuração da lógica

Em vez de:

```yaml
run: |
  echo "api"
  echo "production"
  echo "18"
```

centralize:

```yaml
env:
  APPLICATION_NAME: "api"
  ENVIRONMENT: "production"
  NODE_VERSION: "18"
```

e utilize:

```yaml
run: |
  echo "$APPLICATION_NAME"
  echo "$ENVIRONMENT"
  echo "$NODE_VERSION"
```

---

# 🚨 Problemas comuns

## Variável não disponível

Verifique onde ela foi declarada.

Uma variável definida assim:

```yaml
- name: Step 1
  env:
    TEMP_VALUE: "123"
  run: echo "$TEMP_VALUE"
```

não deve ser considerada disponível no:

```yaml
- name: Step 2
  run: echo "$TEMP_VALUE"
```

porque seu escopo foi limitado ao primeiro step.

---

## Variável disponível em um job e não em outro

Exemplo:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest

    env:
      BUILD_VALUE: "123"
```

Outro job não deve assumir que `BUILD_VALUE` estará disponível automaticamente.

---

# 📋 Checklist

- [ ] Evitei hardcode desnecessário.
- [ ] Identifiquei os valores que precisam ser parametrizados.
- [ ] Escolhi corretamente o escopo.
- [ ] Usei workflow para valores compartilhados.
- [ ] Usei job para valores específicos do job.
- [ ] Usei step para valores pontuais.
- [ ] Consultei as variáveis predefinidas do GitHub.
- [ ] Não utilizei o prefixo `GITHUB_`.
- [ ] Utilizei nomes em caixa alta.
- [ ] Utilizei `_` para separar palavras.
- [ ] Não coloquei credenciais diretamente no `env`.
- [ ] Avaliei o uso de Secrets.
- [ ] Avaliei a combinação de `env` e inputs.
- [ ] Testei o comportamento em diferentes jobs e steps.

---

# 📌 Resumo

As variáveis de ambiente tornam os workflows do GitHub Actions mais **reutilizáveis, parametrizáveis e fáceis de manter**.

Os principais níveis são:

```text
Workflow
   │
   └── Job
        │
        └── Step/Action
```

Use:

```yaml
env:
```

no nível do workflow para valores compartilhados.

Use:

```yaml
jobs:
  job:
    env:
```

para valores específicos de um job.

Use:

```yaml
steps:
  - env:
```

para valores específicos de uma operação.

> **Regra de ouro:** defina a variável no menor escopo que atenda à necessidade do workflow.

---

# 🎯 Conceitos-chave

```text
Hardcode
   │
   ▼
Evitar quando o valor precisa ser reutilizado
   │
   ▼
Environment Variables
   │
   ├── Workflow
   ├── Job
   └── Step/Action
```

E, antes de criar uma variável nova:

```text
Preciso de uma variável?
       │
       ▼
O GitHub já fornece essa informação?
       │
   ┌───┴───┐
  SIM     NÃO
   │       │
   ▼       ▼
Usar     Criar
existente variável
```

A documentação da aula reforça justamente a importância de verificar as variáveis já disponibilizadas pelo GitHub e escolher conscientemente o escopo de cada variável. fileciteturn1file0L343-L385

# GitHub Actions — Contextos

> Guia técnico sobre **Contextos (Contexts) no GitHub Actions**, elaborado com base na transcrição da aula. O objetivo é documentar como os contextos disponibilizam informações sobre o workflow, jobs, steps, runners, variáveis, secrets, estratégias e inputs.

---

## 📌 Visão geral

Os **contextos** são estruturas utilizadas pelo GitHub Actions para disponibilizar informações sobre a execução do workflow.

Entre as principais categorias apresentadas estão:

- `github` — informações relacionadas ao GitHub e à execução.
- `env` — variáveis de ambiente.
- `vars` — variáveis disponibilizadas pelo mecanismo de variáveis.
- `job` / `jobs` — informações dos jobs.
- `steps` — informações dos steps.
- `runner` — informações do ambiente de execução.
- `secrets` — informações relacionadas a secrets.
- `strategy` — informações da estratégia de execução.
- `matrix` — informações da matriz.
- `inputs` — informações recebidas como entradas.

A aula apresenta esses contextos como fontes estruturadas de informações que podem ser consultadas pelo workflow.

---

# 🧠 O que são Contextos?

Contextos são informações armazenadas/disponibilizadas pelo GitHub Actions para que o workflow consiga obter dados relacionados à própria execução.

Conceitualmente:

```text
Workflow
   │
   ▼
Contextos
   │
   ▼
Informações da execução
```

Em vez de criar manualmente informações que o GitHub já possui, o workflow pode consultar o contexto correspondente.

Exemplos de informações disponíveis:

- Repositório.
- Branch/ref.
- Evento que disparou o workflow.
- Run ID.
- Run number.
- Job.
- Runner.
- Variáveis.
- Steps.

---

# 📚 Principais Contextos

| Contexto | Finalidade |
|---|---|
| `github` | Informações relacionadas ao GitHub e à execução |
| `env` | Variáveis de ambiente |
| `vars` | Variáveis disponibilizadas pelo GitHub |
| `job` | Informações do job atual |
| `jobs` | Informações relacionadas aos jobs |
| `steps` | Informações dos steps |
| `runner` | Informações do runner |
| `secrets` | Informações relacionadas a secrets |
| `strategy` | Informações da estratégia de execução |
| `matrix` | Informações da matriz |
| `inputs` | Informações recebidas como entradas |

> A aula informa que `inputs`, `matrix` e `strategy` serão aprofundados posteriormente.

---

# 🔎 Contexto `github`

O contexto `github` disponibiliza informações relacionadas ao GitHub e à execução do workflow.

A aula demonstra informações como:

- Repository owner.
- Repository owner ID.
- Job.
- Branch.
- Run ID.
- Run number.
- Repository ID.
- Evento que disparou a execução.

Exemplos:

```yaml
- name: Repository
  run: echo "${{ github.repository }}"
```

```yaml
- name: Run Number
  run: echo "${{ github.run_number }}"
```

```yaml
- name: Run ID
  run: echo "${{ github.run_id }}"
```

Esses valores são fornecidos pelo contexto `github`.

---

# 🔄 Contexto `env`

O contexto `env` está relacionado às variáveis de ambiente definidas no workflow.

Exemplo:

```yaml
env:
  ENV_WORKFLOW: "valor workflow"
```

Pode ser consultado por expressão:

```yaml
${{ env.ENV_WORKFLOW }}
```

Também pode ser utilizado diretamente no shell:

```bash
echo "$ENV_WORKFLOW"
```

Portanto, existe uma diferença entre acessar a variável no shell e acessar a mesma informação através do contexto do GitHub Actions.

---

# 🏗️ `env` e seus escopos

As variáveis de ambiente podem existir em diferentes níveis:

```text
Workflow
   │
   ├── env
   │
   └── Job
        │
        ├── env
        │
        └── Step
             │
             └── env
```

Exemplo:

```yaml
env:
  ENV_WORKFLOW: "workflow"

jobs:
  teste:
    runs-on: ubuntu-latest

    env:
      ENV_JOB: "job"

    steps:
      - name: Contextos
        env:
          ENV_STEP: "step"
        run: |
          echo "${{ env.ENV_WORKFLOW }}"
          echo "${{ env.ENV_JOB }}"
          echo "${{ env.ENV_STEP }}"
```

---

# 🧩 Contexto `job`

O contexto `job` contém informações relacionadas ao job que está sendo executado.

A aula demonstra informações relacionadas ao estado da execução e aos steps.

Exemplo:

```yaml
- name: Status do Job
  run: echo "${{ job.status }}"
```

Esse tipo de informação pode ser utilizado para controlar o fluxo do workflow.

---

# 🪜 Contexto `steps`

O contexto `steps` disponibiliza informações relacionadas aos steps executados.

Conceitualmente:

```text
steps
 │
 ├── step-1
 ├── step-2
 ├── step-3
 └── ...
```

Um exemplo prático de uso é compartilhar um output entre steps:

```yaml
- name: Gerar informação
  id: build
  run: |
    echo "resultado=sucesso" >> "$GITHUB_OUTPUT"

- name: Consumir informação
  run: |
    echo "${{ steps.build.outputs.resultado }}"
```

> A transcrição apresenta `steps` como contexto, mas não aprofunda outputs. O exemplo acima é uma extensão prática para demonstrar seu uso.

---

# 🖥️ Contexto `runner`

O contexto `runner` disponibiliza informações sobre o ambiente onde o job está sendo executado.

A aula demonstra informações como:

- Sistema operacional.
- Arquitetura.
- Nome.
- Environment.

Exemplos:

```yaml
- name: Sistema operacional
  run: echo "${{ runner.os }}"
```

```yaml
- name: Arquitetura
  run: echo "${{ runner.arch }}"
```

```yaml
- name: Nome do runner
  run: echo "${{ runner.name }}"
```

---

# 🔐 Contexto `secrets`

O contexto `secrets` está relacionado às informações armazenadas como secrets no GitHub.

Exemplo:

```yaml
steps:
  - name: Utilizar token
    env:
      API_TOKEN: ${{ secrets.API_TOKEN }}
    run: |
      ./deploy.sh
```

> Nunca coloque senhas, tokens ou chaves diretamente no YAML quando o valor deve ser protegido.

---

# 📦 Contexto `vars`

O contexto `vars` está relacionado às variáveis disponibilizadas pelo mecanismo de variáveis do GitHub.

Exemplo conceitual:

```yaml
${{ vars.NOME_DA_VARIAVEL }}
```

A aula apresenta o contexto, mas não detalha sua configuração ou seus escopos.

---

# 🎯 Contexto `inputs`

O contexto `inputs` disponibiliza valores recebidos como entradas.

A aula apresenta `inputs` entre os contextos disponíveis e informa que o assunto será aprofundado posteriormente.

Conceitualmente:

```text
Input
  │
  ▼
Workflow
  │
  ▼
Contexto inputs
  │
  ▼
Execução
```

---

# 🔀 Contextos `strategy` e `matrix`

A aula também apresenta os contextos:

```text
strategy
matrix
```

Eles estão relacionados à estratégia de execução e à matriz de jobs.

Conceitualmente:

```text
Strategy
   │
   ▼
Matrix
   │
   ├── Configuração 1
   ├── Configuração 2
   ├── Configuração 3
   └── ...
```

O conteúdo fornecido apenas introduz esses contextos; seus detalhes não são desenvolvidos na aula transcrita.

---

# 🧰 Função `toJSON`

A aula apresenta a função:

```text
toJSON
```

Ela pode ser utilizada para transformar um contexto em JSON, facilitando sua visualização durante a execução.

Fluxo:

```text
Contexto
   │
   ▼
toJSON()
   │
   ▼
JSON
   │
   ▼
Output do Job
```

Isso é particularmente útil para diagnóstico e estudo da estrutura dos contextos.

---

# 🔬 Inspecionando Contextos

Uma técnica demonstrada consiste em converter um contexto para JSON e armazená-lo em uma variável de ambiente para depois imprimir seu conteúdo.

Exemplo:

```yaml
name: Contextos

on:
  workflow_dispatch:

jobs:
  context:
    runs-on: ubuntu-latest

    steps:
      - name: Contexto GitHub
        env:
          GITHUB_CONTEXT: ${{ toJSON(github) }}
        run: |
          echo "===== GITHUB ====="
          echo "$GITHUB_CONTEXT"

      - name: Contexto ENV
        env:
          ENV_CONTEXT: ${{ toJSON(env) }}
        run: |
          echo "===== ENV ====="
          echo "$ENV_CONTEXT"

      - name: Contexto JOB
        env:
          JOB_CONTEXT: ${{ toJSON(job) }}
        run: |
          echo "===== JOB ====="
          echo "$JOB_CONTEXT"

      - name: Contexto STEPS
        env:
          STEPS_CONTEXT: ${{ toJSON(steps) }}
        run: |
          echo "===== STEPS ====="
          echo "$STEPS_CONTEXT"

      - name: Contexto RUNNER
        env:
          RUNNER_CONTEXT: ${{ toJSON(runner) }}
        run: |
          echo "===== RUNNER ====="
          echo "$RUNNER_CONTEXT"
```

---

# 🧪 Workflow de diagnóstico

Um laboratório simples para estudar os contextos:

```yaml
name: Context Diagnostics

on:
  workflow_dispatch:

env:
  ENV_WORKFLOW: "valor workflow"

jobs:
  context:
    runs-on: ubuntu-latest

    env:
      ENV_JOB: "valor job"

    steps:
      - name: GitHub Context
        env:
          GITHUB_CONTEXT: ${{ toJSON(github) }}
        run: |
          echo "===== GITHUB ====="
          echo "$GITHUB_CONTEXT"

      - name: ENV Context
        env:
          ENV_CONTEXT: ${{ toJSON(env) }}
        run: |
          echo "===== ENV ====="
          echo "$ENV_CONTEXT"

      - name: Job Context
        env:
          JOB_CONTEXT: ${{ toJSON(job) }}
        run: |
          echo "===== JOB ====="
          echo "$JOB_CONTEXT"

      - name: Steps Context
        env:
          STEPS_CONTEXT: ${{ toJSON(steps) }}
        run: |
          echo "===== STEPS ====="
          echo "$STEPS_CONTEXT"

      - name: Runner Context
        env:
          RUNNER_CONTEXT: ${{ toJSON(runner) }}
        run: |
          echo "===== RUNNER ====="
          echo "$RUNNER_CONTEXT"
```

Esse workflow serve como laboratório para observar os dados disponibilizados pelo GitHub Actions.

---

# 🔎 Informações observadas na aula

No contexto `github`, a aula mostra informações como:

```text
repository owner
repository owner ID
job
branch
run ID
run number
repository ID
event
```

No contexto `env`, aparecem as variáveis especificadas no workflow, job e step.

No contexto `job`, aparecem informações relacionadas ao job e à sua execução.

No contexto `runner`, aparecem informações como:

```text
OS
Architecture
Name
Environment
```

---

# 🧭 Contextos como mecanismo de controle

Os contextos não servem apenas para visualizar informações. Eles podem ser utilizados para que o workflow tome decisões com base na execução atual.

Fluxo conceitual:

```text
Informação do GitHub
        │
        ▼
     Contexto
        │
        ▼
 Expressão/Condição
        │
        ▼
Decisão do Workflow
        │
        ▼
 Próxima etapa
```

Exemplo:

```yaml
- name: Executar na branch principal
  if: ${{ github.ref == 'refs/heads/main' }}
  run: |
    echo "Executando na branch principal"
```

Outro exemplo:

```yaml
- name: Verificar status
  if: ${{ job.status == 'success' }}
  run: |
    echo "Job executado com sucesso"
```

Esses exemplos mostram como informações dos contextos podem participar do controle de execução.

---

# 🧠 Contextos x Variáveis de Ambiente

É importante diferenciar os conceitos.

### Variável de ambiente

```yaml
env:
  APP_ENV: "production"
```

No shell:

```bash
echo "$APP_ENV"
```

Como contexto:

```yaml
${{ env.APP_ENV }}
```

### Contexto GitHub

```yaml
${{ github.repository }}
```

A informação é fornecida pelo próprio GitHub através do contexto `github`.

Resumo:

```text
Variáveis configuradas
        │
        ▼
       env
        │
        ▼
Contextos do GitHub Actions
        │
        ├── github
        ├── job
        ├── runner
        ├── steps
        ├── secrets
        ├── vars
        ├── inputs
        ├── strategy
        └── matrix
```

---

# 🛠️ Estratégia de diagnóstico

Quando houver dúvida sobre uma informação disponível durante a execução:

### 1. Identifique a informação

```text
Qual informação preciso?
```

### 2. Identifique o contexto

Verifique qual contexto possui essa informação.

### 3. Consulte a documentação

Cada contexto possui documentação específica sobre suas propriedades.

### 4. Inspecione com `toJSON`

Quando necessário:

```yaml
${{ toJSON(github) }}
```

### 5. Analise os logs

Execute o workflow e observe o output.

### 6. Utilize a propriedade correta

Depois de identificar a propriedade, utilize-a no workflow.

---

# 📋 Checklist

- [ ] Entendi o conceito de contexto.
- [ ] Sei diferenciar contexto de variável de ambiente.
- [ ] Conheço `github`.
- [ ] Conheço `env`.
- [ ] Conheço `job`.
- [ ] Conheço `steps`.
- [ ] Conheço `runner`.
- [ ] Conheço `secrets`.
- [ ] Conheço `vars`.
- [ ] Sei que existem `inputs`, `strategy` e `matrix`.
- [ ] Sei consultar a documentação específica de cada contexto.
- [ ] Sei utilizar `toJSON` para diagnóstico.
- [ ] Sei visualizar contextos nos logs.
- [ ] Sei utilizar informações de contexto para controlar o workflow.

---

# 🚨 Cuidados importantes

## Não exponha secrets

Embora `toJSON` seja útil para diagnóstico, evite imprimir informações sensíveis nos logs.

Especialmente:

```text
secrets
```

não deve ser tratado como um contexto apropriado para impressão deliberada.

> **Regra prática:** use `toJSON` para estudar estruturas e diagnosticar workflows, mas nunca transforme informações sensíveis em logs deliberadamente.

---

# 📚 Documentação

A aula recomenda consultar a documentação oficial do GitHub Actions para verificar:

- Contextos disponíveis.
- Propriedades de cada contexto.
- Tipos de valores.
- Condições de utilização.
- Informações disponibilizadas por cada objeto.

Como o GitHub Actions evolui continuamente, a documentação oficial deve ser utilizada como referência atual para propriedades e comportamentos.

---

# 📌 Resumo

Os **contextos do GitHub Actions** são mecanismos para acessar informações relacionadas ao workflow e à sua execução.

Principais contextos apresentados:

```text
github
env
vars
job
jobs
steps
runner
secrets
strategy
matrix
inputs
```

A utilização desses contextos permite que os workflows tenham conhecimento do próprio ambiente de execução e utilizem essas informações para controlar seu comportamento.

---

# 🎯 Conceito-chave

> **Contextos são fontes estruturadas de informações que o GitHub Actions disponibiliza para que o workflow possa consultar dados e controlar seu fluxo de execução.**

Visualização simplificada:

```text
                GitHub Actions
                      │
                      ▼
                  Contextos
                      │
       ┌──────────────┼──────────────┐
       │              │              │
       ▼              ▼              ▼
    github           env          runner
       │              │              │
       ▼              ▼              ▼
   GitHub         Variáveis      Ambiente
   metadata       do workflow    de execução
       │
       └──────────────┬──────────────┘
                      ▼
               Controle do fluxo
                      │
                      ▼
                  Workflow
```

A principal mensagem da aula é que os contextos podem parecer pouco importantes inicialmente, mas tornam-se fundamentais conforme os workflows ficam mais complexos e precisam tomar decisões com base nas informações da própria execução.

---

## 📝 Nota sobre a transcrição

Este documento mantém o escopo e a abordagem apresentados na aula. Alguns exemplos de sintaxe foram organizados para tornar a documentação mais prática, sem alterar o conceito central apresentado: **consultar informações disponibilizadas pelos contextos para compreender e controlar o fluxo de execução do GitHub Actions.**


# GitHub Actions — Variables

> Guia prático sobre **GitHub Actions Variables**, suas diferenças em relação a **Environment Variables**, **Contexts** e **Secrets**, além da configuração em nível de repositório e organização.

---

## 📌 Visão geral

No GitHub Actions existem diferentes mecanismos para disponibilizar informações durante a execução dos workflows.

Nesta aula, o foco está nas **Variables do GitHub Actions**.

É importante não confundir:

- **Contexts** → informações estruturadas disponibilizadas pelo próprio GitHub;
- **Environment Variables (`env`)** → variáveis definidas dentro do workflow;
- **GitHub Actions Variables (`vars`)** → valores de configuração definidos no GitHub, podendo ser reutilizados por workflows;
- **Secrets (`secrets`)** → informações sensíveis que não devem ser tratadas como variáveis de configuração comuns.

A principal diferença apresentada na aula é que uma variável criada no GitHub pode ser reutilizada por diferentes workflows, sem precisar declarar o valor diretamente dentro de cada arquivo YAML.

---

# 🔎 Contexts x Environment Variables x Variables

Antes de trabalhar com `vars`, é importante entender a diferença entre os conceitos.

| Recurso | Característica | Exemplo |
|---|---|---|
| **Contexts** | Informações estruturadas fornecidas pelo GitHub | `${{ github.repository }}` |
| **Environment Variables** | Definidas dentro do workflow, job ou step | `${{ env.MEU_VALOR }}` |
| **Variables** | Configurações cadastradas no GitHub e reutilizáveis | `${{ vars.VAR_CONTEXT }}` |
| **Secrets** | Dados sensíveis protegidos | `${{ secrets.MEU_SECRET }}` |

---

# 🧩 1. Environment Variables

As Environment Variables são definidas diretamente no arquivo do workflow.

Elas podem possuir diferentes níveis de escopo:

```yaml
env:
  VARIAVEL_GLOBAL: "valor"
```

Também podem ser definidas em um job:

```yaml
jobs:
  build:
    env:
      VARIAVEL_JOB: "valor"
```

Ou em um step:

```yaml
steps:
  - name: Executar
    env:
      VARIAVEL_STEP: "valor"
    run: echo "$VARIAVEL_STEP"
```

Essas variáveis fazem parte da configuração do próprio workflow.

---

# 🧠 2. GitHub Actions Variables

As **GitHub Actions Variables** são diferentes das Environment Variables.

Em vez de declarar o valor diretamente no arquivo YAML, podemos cadastrá-lo na configuração do GitHub.

Isso permite que uma configuração seja reutilizada por diferentes workflows.

As Variables apresentadas na aula podem ser configuradas principalmente em:

- **nível de repositório**;
- **nível de organização**.

> O conceito de Variables em nível de Environment é mencionado na aula, mas será tratado posteriormente.

---

# 📦 3. Variables em nível de Repository

Uma variável pode ser criada diretamente dentro de um repositório.

O caminho apresentado é:

```text
Repository
  └── Settings
      └── Secrets and variables
          └── Actions
              └── Variables
```

Na tela de Variables é possível criar uma nova variável.

---

## 📝 Exemplo

Podemos criar uma variável chamada:

```text
VAR_CONTEXT
```

Com o valor:

```text
valor XPTO
```

Depois de criada, essa variável fica disponível para os workflows daquele repositório.

---

# 🔧 4. Utilizando uma Repository Variable

As GitHub Actions Variables são acessadas através do contexto:

```text
vars
```

Por exemplo:

```yaml
${{ vars.VAR_CONTEXT }}
```

Um workflow simples pode ser:

```yaml
name: Variables

on:
  workflow_dispatch:

jobs:
  exemplo:
    runs-on: ubuntu-latest

    steps:
      - name: Exibir variável
        run: echo "Valor da variável: ${{ vars.VAR_CONTEXT }}"
```

Nesse caso:

```text
vars
```

representa o contexto das Variables.

E:

```text
VAR_CONTEXT
```

é o nome da variável criada no GitHub.

---

# 🔄 5. Fluxo de utilização

O fluxo básico fica assim:

```text
GitHub Repository
       │
       ▼
Settings
       │
       ▼
Secrets and variables
       │
       ▼
Actions
       │
       ▼
Variables
       │
       ▼
VAR_CONTEXT = "valor XPTO"
       │
       ▼
Workflow
       │
       ▼
${{ vars.VAR_CONTEXT }}
```

Dessa forma, o valor não precisa ficar escrito diretamente no arquivo YAML.

---

# 🏢 6. Variables em nível de Organization

Além de criar Variables dentro de um repositório, também é possível trabalhar com Variables em nível de **Organization**.

Nesse caso, a configuração é feita na organização.

O caminho apresentado é:

```text
Organization
  └── Settings
      └── Secrets and variables
          └── Actions
              └── Variables
```

A partir dessa tela, uma variável pode ser criada para ser utilizada pelos repositórios autorizados.

---

# 🌐 7. Exemplo de Organization Variable

Podemos criar uma variável:

```text
TESTE_VAR
```

Com o valor:

```text
valor XPTO
```

Essa variável poderá ser utilizada pelos workflows dos repositórios que tiverem acesso a ela.

No workflow, o acesso continua utilizando o contexto:

```yaml
${{ vars.TESTE_VAR }}
```

Ou seja, o mecanismo de acesso permanece:

```text
vars.NOME_DA_VARIAVEL
```

---

# 🔐 8. Controle de acesso das Organization Variables

Uma característica importante das Variables de organização é o controle de quais repositórios poderão utilizá-las.

Durante a criação/configuração da variável, o acesso pode ser direcionado para diferentes conjuntos de repositórios.

Entre as opções apresentadas estão:

### 🌍 Public repositories

A variável poderá ser utilizada pelos repositórios públicos abrangidos pela configuração.

### 🔒 Private repositories

A variável poderá ser utilizada pelos repositórios privados abrangidos pela configuração.

### 🎯 Selected repositories

É possível selecionar especificamente quais repositórios poderão acessar a variável.

Esse modelo é especialmente útil quando uma mesma configuração precisa ser compartilhada por vários projetos, mas não deve ficar disponível para todos os repositórios da organização.

---

# 🆚 9. Repository Variable x Organization Variable

| Característica | Repository Variable | Organization Variable |
|---|---|---|
| Local de criação | Repository | Organization |
| Escopo | Um repositório | Organização |
| Reutilização | Workflows do repositório | Repositórios autorizados |
| Contexto | `vars` | `vars` |
| Controle de acesso | Associado ao repositório | Pode restringir repositórios |
| Uso principal | Configuração específica do projeto | Configuração compartilhada |

---

# 🧪 10. Exemplo prático completo

Imagine que o repositório tenha a seguinte variável:

```text
VAR_CONTEXT = valor XPTO
```

Podemos criar um workflow:

```yaml
name: Test Variables

on:
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Mostrar Variable
        run: |
          echo "Minha variável é: ${{ vars.VAR_CONTEXT }}"
```

Ao executar o workflow, o GitHub disponibilizará o valor configurado na Variable.

---

# 🔀 11. Variables e Environment Variables juntas

Também é possível utilizar uma GitHub Actions Variable para preencher uma Environment Variable dentro de um workflow.

Exemplo:

```yaml
name: Variables

on:
  workflow_dispatch:

jobs:
  exemplo:
    runs-on: ubuntu-latest

    env:
      MINHA_VARIAVEL: ${{ vars.VAR_CONTEXT }}

    steps:
      - name: Exibir variável
        run: echo "$MINHA_VARIAVEL"
```

Nesse cenário:

```text
GitHub Variable
       │
       ▼
vars.VAR_CONTEXT
       │
       ▼
env.MINHA_VARIAVEL
       │
       ▼
Shell
```

Isso permite separar a configuração armazenada no GitHub da forma como ela será consumida pelo processo dentro do workflow.

---

# ⚠️ 12. Variables NÃO são Secrets

Este é um dos pontos mais importantes da aula.

Uma GitHub Actions Variable deve ser utilizada para **configuração e informações que não sejam sensíveis**.

Não devemos utilizar Variables para armazenar informações confidenciais.

### ❌ Não utilizar Variables para:

```text
Senhas
Tokens
API Keys
Chaves privadas
Credenciais
Segredos de autenticação
```

Para esse tipo de informação, devem ser utilizados **GitHub Actions Secrets**.

---

# 🔐 13. Variables x Secrets

A diferença conceitual pode ser representada assim:

```text
                    GitHub Actions
                          │
             ┌────────────┴────────────┐
             │                         │
             ▼                         ▼
         Variables                  Secrets
             │                         │
             ▼                         ▼
      Configuração                 Informação
      não sensível                 sensível
```

Exemplo de uma configuração:

```text
APP_ENVIRONMENT = production
```

Pode ser uma Variable.

Já uma senha:

```text
DB_PASSWORD
```

deve ser tratada como Secret.

Exemplo:

```yaml
env:
  DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
```

---

# 🚨 14. Exemplo do que NÃO fazer

Evite utilizar uma Variable para armazenar uma senha:

```yaml
vars:
  DB_PASSWORD: "minha-senha"
```

ou criar uma GitHub Variable contendo uma credencial.

O problema é que **Variables não são o mecanismo destinado ao armazenamento de informações sensíveis**.

Para dados confidenciais, utilize:

```yaml
${{ secrets.NOME_DO_SECRET }}
```

---

# 🏗️ 15. Quando utilizar Repository Variables?

Repository Variables são adequadas quando uma configuração:

- pertence especificamente a um projeto;
- precisa ser reutilizada por vários workflows daquele repositório;
- não é uma informação sensível;
- não precisa ser duplicada em cada arquivo YAML.

Exemplo:

```text
Repository
└── VAR_CONTEXT
      └── valor XPTO
```

Vários workflows podem consumir:

```yaml
${{ vars.VAR_CONTEXT }}
```

---

# 🏢 16. Quando utilizar Organization Variables?

Organization Variables são adequadas quando uma configuração precisa ser compartilhada entre diferentes repositórios.

Exemplo:

```text
Organization
│
├── Repository A
├── Repository B
├── Repository C
└── Repository D
```

Uma variável de organização pode ser disponibilizada somente aos repositórios autorizados.

Isso evita a necessidade de cadastrar manualmente a mesma configuração em cada projeto.

---

# 🎯 17. Vantagem da centralização

Sem uma Organization Variable, poderíamos ter:

```text
Repository A
└── API_URL = valor

Repository B
└── API_URL = valor

Repository C
└── API_URL = valor
```

Com uma variável centralizada:

```text
Organization
└── API_URL = valor
       │
       ├── Repository A
       ├── Repository B
       └── Repository C
```

A configuração passa a ser administrada em um único ponto, respeitando as permissões definidas para os repositórios.

---

# 📋 18. Comparativo geral

| Recurso | Onde é definido | Contexto | Uso |
|---|---|---|---|
| Context | GitHub | `github`, `job`, `steps`, etc. | Informações da execução |
| `env` | Workflow | `env` | Variáveis do workflow |
| Repository Variable | Repository Settings | `vars` | Configuração do projeto |
| Organization Variable | Organization Settings | `vars` | Configuração compartilhada |
| Secret | Repository/Organization | `secrets` | Dados sensíveis |

---

# 🧠 19. Regra mental

Uma forma simples de memorizar:

```text
Context
   ↓
Informações fornecidas pelo GitHub

env
   ↓
Variáveis definidas no workflow

vars
   ↓
Configurações cadastradas no GitHub

secrets
   ↓
Informações sensíveis
```

---

# 🛠️ 20. Boas práticas

## ✅ Centralize configurações reutilizáveis

Se o mesmo valor é utilizado por vários workflows, considere armazená-lo como Variable.

---

## ✅ Utilize Organization Variables para configurações compartilhadas

Quando diversos repositórios precisam do mesmo valor, uma Organization Variable pode evitar duplicação.

---

## ✅ Restrinja o acesso

Para Organization Variables, utilize o controle de acesso adequado.

Quando possível, utilize:

```text
Selected repositories
```

para limitar a utilização aos projetos que realmente precisam da variável.

---

## ✅ Não armazene informações sensíveis em Variables

Para informações confidenciais:

```text
Secrets
```

é o mecanismo apropriado.

---

## ✅ Utilize nomes claros

Exemplos:

```text
API_URL
APP_ENV
APP_NAME
DOCKER_REGISTRY
DEPLOY_NAMESPACE
VAR_CONTEXT
```

Prefira nomes:

- claros;
- objetivos;
- consistentes;
- em letras maiúsculas;
- separados por `_`.

---

# ⚠️ 21. Erros comuns

### ❌ Confundir `env` com `vars`

```yaml
${{ env.MINHA_VARIAVEL }}
```

é diferente de:

```yaml
${{ vars.MINHA_VARIAVEL }}
```

O primeiro representa uma Environment Variable.

O segundo representa uma GitHub Actions Variable.

---

### ❌ Confundir `vars` com `secrets`

```yaml
${{ vars.API_KEY }}
```

não deve ser utilizado para informações sensíveis.

Para um segredo:

```yaml
${{ secrets.API_KEY }}
```

---

### ❌ Criar a variável no nível errado

Uma variável criada no repositório possui um escopo diferente de uma variável criada na organização.

Sempre verifique onde a configuração foi cadastrada.

---

### ❌ Organization Variable sem acesso ao repositório

Uma Organization Variable pode existir corretamente, mas não estar disponível para determinado repositório devido às regras de acesso configuradas.

Verifique:

```text
Organization
→ Settings
→ Secrets and variables
→ Actions
→ Variables
```

e revise o escopo de acesso.

---

# 🔍 22. Checklist

Antes de utilizar uma Variable em um workflow, confirme:

- [ ] A informação é realmente uma configuração?
- [ ] O dado não é sensível?
- [ ] A variável deveria existir no Repository ou Organization?
- [ ] O nome da variável está claro?
- [ ] O workflow está utilizando `${{ vars.NOME }}`?
- [ ] Caso seja uma Organization Variable, o repositório possui acesso?
- [ ] Para dados sensíveis, foi utilizado `secrets`?

---

# 🚀 23. Exemplo de arquitetura

Uma organização pode estruturar suas configurações desta maneira:

```text
GitHub Organization
│
├── Variables
│   ├── DOCKER_REGISTRY
│   ├── APP_ENV
│   └── DEFAULT_REGION
│
├── Repository A
│   └── Variables
│       └── APP_NAME
│
├── Repository B
│   └── Variables
│       └── APP_NAME
│
└── Repository C
    └── Variables
        └── APP_NAME
```

E os workflows podem consumir essas informações através de:

```yaml
${{ vars.DOCKER_REGISTRY }}
```

ou:

```yaml
${{ vars.APP_NAME }}
```

Enquanto informações sensíveis devem permanecer em:

```yaml
${{ secrets.NOME_DO_SECRET }}
```

---

# 📚 24. Resumo

As **GitHub Actions Variables** permitem armazenar configurações no próprio GitHub e reutilizá-las em workflows.

Os principais conceitos apresentados são:

### Contexts

São informações estruturadas disponibilizadas pelo GitHub.

```yaml
${{ github.repository }}
```

### Environment Variables

São definidas no workflow e podem possuir escopo de workflow, job ou step.

```yaml
${{ env.MINHA_VARIAVEL }}
```

### GitHub Actions Variables

São configurações cadastradas no GitHub e acessadas através do contexto `vars`.

```yaml
${{ vars.VAR_CONTEXT }}
```

Podem ser configuradas em nível de:

```text
Repository
Organization
```

### Secrets

Devem ser utilizados para informações sensíveis.

```yaml
${{ secrets.MEU_SECRET }}
```

---

# 🎯 Conceito principal

> **`env` é uma variável definida no workflow. `vars` representa uma configuração cadastrada no GitHub. `secrets` representa uma informação sensível.**

A separação correta desses mecanismos ajuda a manter os workflows mais organizados, reutilizáveis e fáceis de administrar.

---

## 📖 Referência da aula

Conteúdo estruturado a partir da transcrição fornecida, preservando os conceitos, exemplos e terminologia apresentados na aula sobre **GitHub Actions Variables**.
