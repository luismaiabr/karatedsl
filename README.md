# Tutorial: Instalação e Configuração do Karate DSL para Testes da API Brownie

## Índice
1. [Introdução](#introdução)
2. [Pré-requisitos](#pré-requisitos)
3. [Instalação do Karate DSL](#instalação-do-karate-dsl)
4. [Estrutura do Projeto de Testes](#estrutura-do-projeto-de-testes)
5. [Configuração dos Testes](#configuração-dos-testes)
6. [Exemplos de Testes](#exemplos-de-testes)
7. [Executando os Testes](#executando-os-testes)
8. [Boas Práticas](#boas-práticas)

---

## Introdução

O Karate DSL é um framework de testes de API que combina automação de testes, mocks e relatórios de performance em uma única ferramenta. Este tutorial mostrará como configurar o Karate para testar a API Brownie, que é construída com FastAPI e usa Supabase como banco de dados.

### Endpoints da API Brownie

A API possui os seguintes endpoints principais:

- **Estoque**: `/api/v1/estoque/*`
  - GET `/estoque` - Listar estoque por categoria
  - POST `/estoque_atual` - Obter estoque atual de uma categoria
  - POST `/atualizar_estoque` - Atualizar estoque (UPSERT)
  - POST `/adicionar_ao_estoque` - Adicionar ao estoque
  - GET `/categorias_estoque` - Listar categorias
  - POST `/preco_unitario` - Obter preço unitário

- **Produtos**: `/api/v1/produtos/*`

- **Vendas**: `/api/v1/vendas/*`
  - POST `/vender` - Registrar nova venda

- **Histórico**: `/api/v1/historico/*`

- **Cobrança**: `/api/v1/cobranca/*`

- **Clientes**: `/api/v1/clientes/*`

---

## Pré-requisitos

### 1. Java Development Kit (JDK)

O Karate requer Java 8 ou superior.

**Verificar instalação:**
```bash
java -version
```

**Instalar Java (se necessário):**

**Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install openjdk-17-jdk
```

**macOS (com Homebrew):**
```bash
brew install openjdk@17
```

**Windows:**
Baixe o instalador do [Oracle](https://www.oracle.com/java/technologies/downloads/) ou [AdoptOpenJDK](https://adoptium.net/).

### 2. Maven

O Maven será usado para gerenciar dependências do projeto Karate.

**Verificar instalação:**
```bash
mvn -version
```

**Instalar Maven:**

**Ubuntu/Debian:**
```bash
sudo apt install maven
```

**macOS:**
```bash
brew install maven
```

**Windows:**
Baixe de [Apache Maven](https://maven.apache.org/download.cgi) e adicione ao PATH.

### 3. Python e FastAPI (sua API)

Certifique-se de que a API Brownie está funcionando:

```bash
# No diretório do projeto bonobrownie
poetry install
poetry run uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

Acesse: http://localhost:8000/docs para verificar a documentação Swagger.

---

## Instalação do Karate DSL

### Opção 1: Projeto Maven Standalone

#### Passo 1: Criar estrutura do projeto de testes

No diretório raiz do projeto `bonobrownie`, crie uma pasta para os testes:

```bash
mkdir -p tests-karate
cd tests-karate
```

#### Passo 2: Criar o arquivo `pom.xml`

Crie o arquivo `pom.xml` com o seguinte conteúdo:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.bonobrownie</groupId>
    <artifactId>brownie-api-tests</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>17</java.version>
        <maven.compiler.version>3.11.0</maven.compiler.version>
        <karate.version>1.4.1</karate.version>
    </properties>

    <dependencies>
        <!-- Karate Core -->
        <dependency>
            <groupId>com.intuit.karate</groupId>
            <artifactId>karate-junit5</artifactId>
            <version>${karate.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <testResources>
            <testResource>
                <directory>src/test/java</directory>
                <excludes>
                    <exclude>**/*.java</exclude>
                </excludes>
            </testResource>
        </testResources>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>${maven.compiler.version}</version>
                <configuration>
                    <encoding>UTF-8</encoding>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.0.0</version>
                <configuration>
                    <argLine>-Dfile.encoding=UTF-8</argLine>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

#### Passo 3: Criar estrutura de diretórios

```bash
mkdir -p src/test/java/brownie
mkdir -p src/test/java/brownie/features
```

---

## Estrutura do Projeto de Testes

A estrutura final ficará assim:

```
bonobrownie/
├── app/                          # API FastAPI
├── tests-karate/                 # Projeto Karate
│   ├── pom.xml
│   └── src/
│       └── test/
│           └── java/
│               └── brownie/
│                   ├── BrownieApiTest.java           # Runner principal
│                   ├── karate-config.js              # Configuração global
│                   └── features/
│                       ├── estoque.feature           # Testes de estoque
│                       ├── vendas.feature            # Testes de vendas
│                       ├── clientes.feature          # Testes de clientes
│                       └── cobranca.feature          # Testes de cobrança
└── pyproject.toml
```

---

## Configuração dos Testes

### 1. Arquivo de Configuração Global: `karate-config.js`

Crie o arquivo `src/test/java/brownie/karate-config.js`:

```javascript
function fn() {
  var env = karate.env; // obtém o ambiente da variável -Dkarate.env
  karate.log('karate.env system property was:', env);

  if (!env) {
    env = 'dev'; // ambiente padrão
  }

  var config = {
    baseUrl: 'http://localhost:8000/api/v1',
    apiTimeout: 10000
  };

  // Configurações específicas por ambiente
  if (env == 'dev') {
    config.baseUrl = 'http://localhost:8000/api/v1';
  } else if (env == 'staging') {
    config.baseUrl = 'https://staging-api.bonobrownie.com/api/v1';
  } else if (env == 'prod') {
    config.baseUrl = 'https://api.bonobrownie.com/api/v1';
  }

  karate.configure('connectTimeout', config.apiTimeout);
  karate.configure('readTimeout', config.apiTimeout);

  return config;
}
```

### 2. Test Runner: `BrownieApiTest.java`

Crie o arquivo `src/test/java/brownie/BrownieApiTest.java`:

```java
package brownie;

import com.intuit.karate.junit5.Karate;

class BrownieApiTest {

    @Karate.Test
    Karate testAll() {
        return Karate.run().relativeTo(getClass());
    }

    @Karate.Test
    Karate testEstoque() {
        return Karate.run("features/estoque").relativeTo(getClass());
    }

    @Karate.Test
    Karate testVendas() {
        return Karate.run("features/vendas").relativeTo(getClass());
    }

    @Karate.Test
    Karate testClientes() {
        return Karate.run("features/clientes").relativeTo(getClass());
    }
}
```

---

## Exemplos de Testes

### 1. Testes de Estoque: `estoque.feature`

Crie o arquivo `src/test/java/brownie/features/estoque.feature`:

```gherkin
Feature: Testes de Gestão de Estoque

  Background:
    * url baseUrl
    * def categoriaTeste = 'Brownie Tradicional'

  Scenario: Listar todas as categorias de estoque
    Given path '/estoque/categorias_estoque'
    When method get
    Then status 200
    And match response == '#array'
    And match each response == '#string'

  Scenario: Obter estoque completo
    Given path '/estoque/estoque'
    When method get
    Then status 200
    And match response == '#array'
    And match each response contains { categoria: '#string', quantidade: '#number' }

  Scenario: Obter estoque de uma categoria específica
    Given path '/estoque/estoque_atual'
    And request { categoria: '#(categoriaTeste)' }
    When method post
    Then status 200
    And match response == '#number'

  Scenario: Atualizar estoque de uma categoria
    Given path '/estoque/atualizar_estoque'
    And request
      """
      {
        "categoria": "#(categoriaTeste)",
        "quantidade": 100
      }
      """
    When method post
    Then status 200
    And match response.message contains 'atualizado com sucesso'

  Scenario: Adicionar ao estoque existente
    # Primeiro, obter o estoque atual
    Given path '/estoque/estoque_atual'
    And request { categoria: '#(categoriaTeste)' }
    When method post
    Then status 200
    * def estoqueAntes = response

    # Adicionar 10 unidades
    Given path '/estoque/adicionar_ao_estoque'
    And request
      """
      {
        "categoria": "#(categoriaTeste)",
        "quantidade": 10
      }
      """
    When method post
    Then status 200
    And match response.message contains 'incrementado com sucesso'

    # Verificar que o estoque aumentou
    Given path '/estoque/estoque_atual'
    And request { categoria: '#(categoriaTeste)' }
    When method post
    Then status 200
    And assert response == estoqueAntes + 10

  Scenario: Obter preço unitário de uma categoria
    Given path '/estoque/preco_unitario'
    And request { categoria: '#(categoriaTeste)' }
    When method post
    Then status 200
    And match response == '#number'
    And assert response >= 0

  Scenario: Erro ao buscar categoria inexistente
    Given path '/estoque/estoque_atual'
    And request { categoria: 'Categoria_Inexistente_XYZ123' }
    When method post
    Then status 404
```

### 2. Testes de Vendas: `vendas.feature`

Crie o arquivo `src/test/java/brownie/features/vendas.feature`:

```gherkin
Feature: Testes de Vendas

  Background:
    * url baseUrl
    * def vendaValida =
      """
      {
        "categoria_produto": "Brownie Tradicional",
        "cliente_id": 1,
        "qtd_unidades": 5,
        "prazo_dias": 7,
        "valor_unitario": 10.50
      }
      """

  Scenario: Registrar uma nova venda com sucesso
    # Obter estoque antes da venda
    Given path '/estoque/estoque_atual'
    And request { categoria: 'Brownie Tradicional' }
    When method post
    Then status 200
    * def estoqueAntes = response

    # Registrar venda
    Given path '/vendas/vender'
    And request vendaValida
    When method post
    Then status 201

    # Verificar que o estoque foi decrementado
    Given path '/estoque/estoque_atual'
    And request { categoria: 'Brownie Tradicional' }
    When method post
    Then status 200
    And assert response == estoqueAntes - 5

  Scenario: Venda com valor unitário omitido (deve usar preço do estoque)
    Given path '/vendas/vender'
    And request
      """
      {
        "categoria_produto": "Brownie Tradicional",
        "cliente_id": 1,
        "qtd_unidades": 2,
        "prazo_dias": 7
      }
      """
    When method post
    Then status 201

  Scenario: Tentar vender quantidade negativa
    Given path '/vendas/vender'
    And request
      """
      {
        "categoria_produto": "Brownie Tradicional",
        "cliente_id": 1,
        "qtd_unidades": -5,
        "prazo_dias": 7,
        "valor_unitario": 10.50
      }
      """
    When method post
    Then status 422
```

### 3. Testes de Clientes: `clientes.feature`

Crie o arquivo `src/test/java/brownie/features/clientes.feature`:

```gherkin
Feature: Testes de Gestão de Clientes

  Background:
    * url baseUrl

  Scenario: Listar todos os clientes
    Given path '/clientes'
    When method get
    Then status 200
    And match response == '#array'

  Scenario: Obter cliente específico
    Given path '/clientes/1'
    When method get
    Then status 200
    And match response contains { id: 1 }

  Scenario: Cliente inexistente retorna 404
    Given path '/clientes/99999'
    When method get
    Then status 404
```

### 4. Testes de Integração Completa: `integracao.feature`

Crie o arquivo `src/test/java/brownie/features/integracao.feature`:

```gherkin
Feature: Testes de Integração Completa - Fluxo de Venda

  Background:
    * url baseUrl

  Scenario: Fluxo completo: adicionar estoque, realizar venda e verificar cobrança
    # 1. Adicionar estoque
    Given path '/estoque/adicionar_ao_estoque'
    And request
      """
      {
        "categoria": "Brownie Premium",
        "quantidade": 50
      }
      """
    When method post
    Then status 200
    * def estoqueInicial = response

    # 2. Verificar estoque adicionado
    Given path '/estoque/estoque_atual'
    And request { categoria: 'Brownie Premium' }
    When method post
    Then status 200
    * def estoqueAtual = response
    * print 'Estoque atual:', estoqueAtual

    # 3. Realizar venda
    Given path '/vendas/vender'
    And request
      """
      {
        "categoria_produto": "Brownie Premium",
        "cliente_id": 1,
        "qtd_unidades": 10,
        "prazo_dias": 15,
        "valor_unitario": 12.50
      }
      """
    When method post
    Then status 201

    # 4. Verificar estoque após venda
    Given path '/estoque/estoque_atual'
    And request { categoria: 'Brownie Premium' }
    When method post
    Then status 200
    And assert response == estoqueAtual - 10

    # 5. Verificar que a cobrança foi criada (se houver endpoint)
    # Given path '/cobranca/cliente/1'
    # When method get
    # Then status 200
    # And match response contains { cliente_id: 1 }
```

---

## Executando os Testes

### 1. Iniciar a API FastAPI

Em um terminal, inicie a API:

```bash
cd /home/luis/Documents/PROBONO/BACK/bonobrownie
poetry run uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

### 2. Executar todos os testes

Em outro terminal:

```bash
cd tests-karate
mvn test
```

### 3. Executar testes específicos

**Apenas testes de estoque:**
```bash
mvn test -Dtest=BrownieApiTest#testEstoque
```

**Apenas testes de vendas:**
```bash
mvn test -Dtest=BrownieApiTest#testVendas
```

### 4. Executar com ambiente específico

```bash
mvn test -Dkarate.env=staging
```

### 5. Gerar relatórios HTML

Após executar os testes, os relatórios serão gerados em:
```
tests-karate/target/karate-reports/
```

Abra o arquivo `karate-summary.html` no navegador para visualizar os resultados.

---

## Boas Práticas

### 1. Organização de Features

- **Uma feature por módulo**: Crie features separadas para cada funcionalidade (estoque, vendas, clientes)
- **Use Background**: Coloque configurações comuns no bloco `Background`
- **Reutilize variáveis**: Use `* def` para definir dados reutilizáveis

### 2. Validações Robustas

```gherkin
# Validar estrutura de resposta
And match response == { id: '#number', name: '#string', active: '#boolean' }

# Validar arrays
And match response == '#array'
And match each response contains { id: '#number' }

# Validar com regex
And match response.email == '#regex .+@.+\\..+'

# Asserções customizadas
And assert response.price > 0
```

### 3. Dados Dinâmicos

```gherkin
# Gerar dados aleatórios
* def randomId = Math.floor(Math.random() * 1000)
* def timestamp = new Date().getTime()

# Usar JavaScript
* def precoComDesconto = function(preco) { return preco * 0.9 }
```

### 4. Tratamento de Erros

```gherkin
# Validar códigos de erro
Then status 400
And match response.detail contains 'erro'

# Validar estrutura de erro
And match response == { detail: '#string' }
```

### 5. Setup e Teardown

```gherkin
# Criar dados antes do teste
* def criarCliente = call read('helpers/criar-cliente.feature')

# Limpar após o teste
* def limparDados = karate.call('helpers/limpar-dados.feature')
```

---

## Comandos Úteis

```bash
# Limpar e recompilar
mvn clean test

# Executar em paralelo (mais rápido)
mvn test -Dkarate.options="--threads 5"

# Modo debug (mais verboso)
mvn test -Dkarate.options="--tags @debug"

# Executar apenas scenarios com tag específica
mvn test -Dkarate.options="--tags @smoke"
```

---

## Troubleshooting

### Problema: "Connection refused"

**Solução**: Verifique se a API está rodando em `http://localhost:8000`

```bash
curl http://localhost:8000/
```

### Problema: "Compilation failure"

**Solução**: Verifique a versão do Java

```bash
java -version
# Deve ser Java 8 ou superior
```

### Problema: Testes falham com timeout

**Solução**: Aumente o timeout no `karate-config.js`:

```javascript
karate.configure('connectTimeout', 30000);
karate.configure('readTimeout', 30000);
```

### Problema: Encoding de caracteres

**Solução**: Certifique-se de usar UTF-8:

```bash
mvn test -Dfile.encoding=UTF-8
```

---

## Recursos Adicionais

- **Documentação oficial**: https://karatelabs.github.io/karate/
- **Exemplos**: https://github.com/karatelabs/karate/tree/master/karate-demo
- **Comunidade**: https://stackoverflow.com/questions/tagged/karate

---

## Conclusão

Agora você tem um ambiente completo de testes automatizados para a API Brownie usando Karate DSL. Os testes cobrem:

- Gestão de estoque
- Registro de vendas
- Gestão de clientes
- Integrações completas

Para adicionar novos testes, basta criar novas features seguindo os padrões apresentados neste tutorial.
