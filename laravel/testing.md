---
layout: default
title: Testes
permalink: /laravel/testing
---

# Testes

[:arrow_backward: Voltar](../laravel)

> Documentação oficial: [Testing](https://laravel.com/docs/testing)

- [**Introdução**](#introdução)
- [**Organização das Pastas**](#organização-das-pastas)
- [**Organização dos Testes**](#organização-dos-testes)

## Introdução

Testes automatizados são muito importantes em nossas aplicações. Então todo código produzido, que tenha a intenção de ser mantido pelo time de desenvolvimento, deve ser testado.

Os testes devem ser escritos por quem criou o código, e não por apenas um integrante do time. Uma boa metodologia de testes é o TDD, porém não é uma obrigatoriedade.

Por padrão o Laravel traz um setup pronto para testes usando o [**PHPUnit**](https://phpunit.de/). Este framework é muito fácil de usar e é muito flexível, permitindo que você teste todos os módulos de sua aplicação.

Existe também o [Pest](https://pestphp.com/) que traz todas as funcionalidades do PHPUnit, porém mais prático e mais moderno.

Por padrão vamos segui-lo, porém cada time tem a liberdade de escolher o framework que melhor se adapte ao seu projeto. Porém sempre teste o que você cria.

## Organização das Pastas

Devemos seguir algum padrão em como organizar nosso testes. Uma sugestão é a seguinte:

```bash
tests
├── Feature
│   ├── Commands
│   ├── Endpoints
│   ├── Middleware
│   └── Observers
└── Unit
    ├── Entities
    └── Services
```

Por padrão a pasta `Unit` deveria ter apenas testes que não dependam diretamente do Laravel, porém como os casos que se aplicam a isso são muito escassos, vamos deixar a pasta `Unit` como sendo para testar as partes que não dependem diretamente de uma chamada HTTP ou de um Comando para serem executadas, ou seja, as regras de negócio da aplicação.

- `Entities`: Testes das Entidades.
- `Services`: Testes dos Serviços.

Na pasta `Feature` devemos colocar todos os testes que dependem diretamente do Laravel ou que testem um fluxo completo de uma ação.

- `Commands`: Testes dos Comandos.
- `Endpoints`: Testes que chamam diretamente as rotas da nossa aplicação.
- `Middleware`: Testes dos Middleware.
- `Observers`: Testes dos Observers.

## Organização dos Testes

Os testes devem ser feitos para todos os casos possíveis de uma ação ou fluxo. Por isso, uma simples rota terá vários testes, sendo que alguns deles vão testar o fluxo de sucesso, e outros as exceções que podem acontecer.

Para isso podemos separar da seguinte maneira:

```bash
Endpoints
└── Resource
    ├── ResourcePermissionTest.php
    ├── ResourceTest.php
    └── ResourceValidationTest.php
```

- `ResourceTest.php`: Testes que testam o fluxo padrão relacionado aos Endpoints de `Resource`
- `ResourceValidationTest.php`: Testes que testam as validações relacionado aos Endpoints de `Resource`, onde as chamadas HTTP devem retornar erros de validação, garantindo assim que existem validações no código.
- `ResourcePermissionTest.php`: Testes que testam as permissões de acesso relacionado aos Endpoints de `Resource`, onde as chamadas HTTP devem retornar erros de permissão (403).

E assim por diante. Case precisamos agrupar outros testes que não testem o fluxo de sucesso, podemos criar outro arquivo que agrupem esses testes.

No caso dos testes dentro da pasta `Unit` não haverá essa necessidade de separar os testes em mais de um arquivo já que cada arquivo estará testando uma única ação ou entidade. Mas caso a equipe ache necessário, também pode ser feita essa separação.

[Voltar para o topo](#validação)
