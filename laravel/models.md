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
  $modelsFiltered = $this->getFilteredModels();
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

Escopos devem ser usados para criar filtros padrões na Model que contenham um nome que explique o que o filtro faz. Exemplo:

```php
// Ruim quando usado fora da Model
Model::query()->where('active', 1)->get();

// Bom
Model::whereActive()->get();
```

Esse caso é simples, mas mostra como os escopos podem ajudar a organizar os filtros na Model de maneira mais descritível e menos pensando em queries no banco de dados, o que ajuda a novos desenvolvedores a entenderem o que uma query complexa faz.

## Funções gerais

Essas funções servem para agrupar queries que são utilizadas em vários locais para evitar repetição de código ou queries complexas e confusas dentro das services ou controllers. Elas podem ser estáticas ou não.

As funções estáticas não dependem de uma instância da model, então toda query é feita diretamente na model.

```php
public static function getReportFormattedModels()
{
	return Model::query()
		->select()
		->join()
		->join()
		->whereIn()
		->get();
}
```

Já as funções não estáticas dependem de uma instância da model, então a query é feita a partir de uma model já carregada do banco.

```php
public function getReportFormattedModels()
{
	return $this->relationship()
		->with()
		->withCount()
		->where()
		->whereHas()
		->first();
}
```

Prefira usar o segundo caso quando a query for filtrada por **um único registro**, como por exemplo trazer todos os post filtrados de **um usuário**. Nesse caso, a função deveria estar na model do usuário com uma função não estática e não na model de post como estática.

## Query Builder

Em alguns casos, nossas models podem ficar grandes demais por terem muitos escopos ou quando sabemos de ante mão que essa model irá crescer em tamanho e funcionalidades. Nesses casos podemos utilizar um Query Builder customizado para essa model.

Devemos primeiramente criar uma pasta no diretório `app/Eloquent/Builders` e um arquivo `FooBuilder.php`, onde `Foo` é o nome da Model. O conteúdo desse arquivo deve seguir o padrão abaixo:

```php
<?php

namespace App\Eloquent\Builders;

use Illuminate\Database\Eloquent\Builder;

class FooBuilder extends Builder
{
	public function whereActive()
	{
		// ..
	}

	public function whereValidStatus()
	{
		// ..
	}
}
```

E na Model:

```php
use Illuminate\Database\Query\Builder as QueryBuilder;
use Illuminate\Database\Eloquent\Builder;
use App\Eloquent\Builders\FooBuilder;

class Foo extends Model
{
	public function newEloquentBuilder(QueryBuilder $query): Builder
	{
		return new FooBuilder($query);
	}

	// Removemos os escopos abaixo
	public function scopeWhereActive() // remover
	{
		// ..
	}

	public function scopeWhereValidStatus() // remover
	{
		// ..
	}
}
```

E podemos utilizar normalmente como era usado antes com os escopos diretamente na Model. Utilizar um Query Builder customizado traz, além da separação de responsabilidades, uma vantagem nas IDE's que passam a informar os escopos no IntelliSense, o que não acontecia anteriormente.

## Eloquent Collections

Seguindo o query builders, em alguns casos podemos ter vários ações padrões que podem ser feitas em uma coleção de models. Por exemplo, podemos filtrar uma coleção de models por um atributo, ordenar por um atributo, mapear para um formato diferente do padrão que vem da model, etc. Algo como:

```php
public function handle(Collection $models)
{
	$models
		->mapWithKeys()
		->filter()
		->sort()
		->values();
}

// Ou

public function handle()
{
	Model::query()
		->where()
		->get()
		->mapWithKeys()
		->filter()
		->sort()
		->values();
}
```

Caso essas ações feitas nas collections sejam utilizadas em mais de um local ou sejam complexas, podemos criar uma Eloquent Collection customizada para a Model.

Devemos primeiramente criar uma pasta no diretório `app/Eloquent/Collections` e um arquivo `FooCollection.php`, onde `Foo` é o nome da Model. O conteúdo desse aquivo deve seguir o padrão abaixo:

```php
<?php

namespace App\Eloquent\Builders;

use Illuminate\Database\Eloquent\Collection;

class FooCollection extends Collection
{
	public function onlyWithCode()
	{
		// ..
	}
}
```

E na Model:

```php
use App\Eloquent\Collections\FooCollection;
use Illuminate\Database\Eloquent\Collection;

class Foo extends Model
{
	public function newCollection(array $models = []): Collection
	{
		return new FooCollection($models);
	}
}
```

[Voltar para o topo](#models)
