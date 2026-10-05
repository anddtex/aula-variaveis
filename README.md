# aula-variaveis
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
