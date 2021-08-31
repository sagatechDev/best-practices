---
layout: default
title: Models
description: Tabelas do BD em classes PHP.
permalink: /laravel/models
---

# Models

[:arrow_backward: Voltar](../laravel)

- [**Introdução**](#introdução)

## Introdução

As models são a representação de tabelas do BD em classes PHP, então nelas só devem ter funções relacionadas ao banco de dados.

Não devemos utilizar a Model em outros lugares do projeto como uma construtora de queries complexas. Devemos utiliza-la como uma proxy das interações com o BD, utilizando de funções que abstraem o que desejamos fazer.

Exemplo:

```php
// Dentro de uma service

// Ruim
public function handle() {
  $modelsFiltered = Model::query()
    ->where()
    ->whereIn()
    ->where(fn($query) => $query->where()->orWhere())
    ->whereHas()
    // ...
    ->get();
}

/**
 * Melhor, se essa query for utilizada apenas nessa service,
 * pois isola a query do resto da service.
 */

public function handle()
{
  $modelsFiltered = $this->getModelsFiltered();
}

private function getFilteredModels()
{
  return Model::query()
    ->where()
    ->whereIn()
    ->where(fn($query) => $query->where()->orWhere())
    ->whereHas()
    // ...
    ->get();
}

// Bom

// Dentro da Model

public static function getFilteredModels()
{
  return Model::query()
    ->where()
    ->whereIn()
    ->where(fn($query) => $query->where()->orWhere())
    ->whereHas()
    // ...
    ->get();
}

// Dentro de uma service

public function handle()
{
  $modelsFiltered = Model::getModelsFiltered();
}

```

Apesar do exemplo acima melhorar a manutenibilidade de código, ainda há maneiras melhores que o Eloquent fornece, e vamos mostrar algumas delas nessa documentação.

Começando pelo que podemos ter nas models:

- Relacionamentos
- Escopos
- Funções que grupam ações no DB

## Relacionamentos

> Documentação oficial: [Relationships](https://laravel.com/docs/eloquent-relationships)

Essas funções devem seguir o seguinte padrão:

- Caso o relacionamento retorne uma instancia, deve-se chamar o método com o nome da outra Model no singular. Exemplo:

```php
public function profile()
{
	return $this->belongsTo(Profile::class);
}
```

- Caso o relacionamento retorne uma coleção, deve-se chamar o método com o nome da outra Model no plural. Exemplo:

```php
public function users()
{
	return $this->hasMany(User::class);
}
```

> **Obs:** Não é necessário criar todos os relacionamentos se eles não estão sendo utilizados de ambos os lados. Criar relacionamentos sem necessidade apenas aumenta o tamanho da Model.

## Escopos

Escopos devem ser usados para criar filtros padrões da Model que contenham um nome que explique o que o filtro faz. Exemplo:

```php
// Ruim quando usado fora da Model
Model::query()->where('active', 1)->get();

// Bom
Model::whereActive()->get();
```

Esse caso é simples, mas mostra como os escopos podem ajudar a organizar os filtros na Model de maneira mais descritível e menos pensando em queries no banco de dados, o que ajuda a novos desenvolvedores a entenderem o que uma query complexa faz.

[Voltar para o topo](#services)
