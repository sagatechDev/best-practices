---
layout: default
title: Rotas
description: A base de acesso da sua aplicação.
permalink: /laravel/routes
---

# Rotas

> Documentação oficial: [Routes](https://laravel.com/docs/routing)

- [Introdução](#introdução)
- [Resources](#resources)
  - [Only e Except](#only-except)
- [Rotas de Ação](#rotas-de-ação)
- [Nested Resources](#nested-resources)
- [Um pouco sobre RESTful](#um-pouco-sobre-restful)
- [Outras boas práticas para rotas](#outras-boas-práticas-para-rotas)

## Introdução

As rotas não devem ser criadas utilizando closures, mas sim, Controllers ou qualquer outra forma que o Laravel suporta como `Route::view('example', 'example-view')`.

## Resources

Ao pensar em rotas, tente pensar em _resources_ (recursos), não em endereços. Devemos buscar seguir o padrão RESTful, onde cada recurso e ações sobre este é representado por uma URL. Assim devemos fazer uso massivo de rotas _resources_ como mostrado [aqui](https://laravel.com/docs/controllers#resource-controllers).

As rotas resources trazem várias vantagens, como:

- Padronização.
- Facilidade para implementar as rotas RESTful.
- Automação, como nos nomes das rotas e nos métodos do Controller.

```php
// Ruim, sem nomes e sem padrão
Route::get('resources/index', [ResourceController::class, 'index']);
Route::get('resources/show/{id}', [ResourceController::class, 'show']);
Route::get('resources/create', [ResourceController::class, 'create']);
Route::post('resources/store', [ResourceController::class, 'store']);
Route::get('resources/edit/{id}', [ResourceController::class, 'edit']);
Route::put('resources/update/{id}', [ResourceController::class, 'update']);
Route::delete('resources/deletar/{id}', [ResourceController::class, 'destroy']);

// Bom
Route::resource('resources', ResourceController::class);
```

Apenas com a utilização de _resources_ já facilitamos bastante o processo de criação de rotas, manutenção e testes.

Os resources devem ser criados preferencialmente em inglês, mas em alguns casos pode se abrir exceções, sempre contendo com a aprovação de todo o time de desenvolvimento.

### Mas e se eu precisar de apenas um (ou alguns) método(s) do resource? {#only-except}

Nesse caso você deve utilizar o método `only` para definir os métodos que você deseja. Ou caso haja mais métodos que você não deseja utilizar, você pode utilizar o método `except`.

Deve-se escolher entre only ou except pensado no menor esforço. `->except('create', 'edit', 'store', 'update', 'destroy')` ou `->only('index', 'show')`? E vice-versa.

Deve se salientar que devemos expor apenas os métodos do resource que queremos utilizar, não todos. Por exemplo, no caso de API's devemos utilizar apenas `apiResource`, seguindo as mesmas regras dos resources.

```php
// Ruim
Route::resource('resources', ResourceController::class)->except('edit', 'create');

//Bom
Route::apiResource('resources', ResourceController::class);
```

## Rotas de Ação

Em alguns casos as rotas _resources_ não serão o suficiente para nossa aplicação, e nesses casos entram as rotas de ação.

As rotas de ação são utilizadas quando queremos que uma ação seja executada em um recurso que não seja o CRUD.

> Obs: Em alguns casos o que se acha que não é um CRUD acaba ainda sendo porém de outro recurso que não diretamente o da Model. Veja o que o Adam Wathan [diz aqui](https://www.youtube.com/watch?v=MF0jFKvS4SI). Cuidado para não utilizar as rotas de ação em excesso e não pensar em [OOP](https://en.wikipedia.org/wiki/Object-oriented_programming).

Esse tipo de rota deve sempre apontar para um [Single Action Controller](https://laravel.com/docs/controllers#single-action-controllers) e não para um método extra dentro de um [Resource Controller](https://laravel.com/docs/controllers#resource-controllers).

```php
// Ruim
Route::get('resources/{resource}/verify-something', [ResourceController::class, 'verifySomething']);

// Bom
Route::get('resources/{resource}/verify-something', VerifySomethingResourceController::class)->name('resources.verify-something');
```

Vale ressaltar que todas as rotas devem ter nome, seguindo o mesmo padrão dos nomes gerados automaticamente pelo `Route::resource()`.

## Nested Resources

> Documentação oficial: [Nested Resources](https://laravel.com/docs/controllers#restful-nested-resources)

Caso seja necessário retornar dados onde haja relacionamento entre os recursos, devemos utilizar os recursos aninhados.

> CRUD de medidores de água vinculados a um apartamento.

```php
Route::resource('apartments.meters', ApartmentMeterController::class);
```

## Um pouco sobre RESTful

Já que vamos seguir esse padrão devemos entender um pouco sobre ele. Aconselho a lerem esse [blog post](https://www.brunobrito.net.br/api-restful-boas-praticas/) sobre o assunto para começar suas pesquisas.

Devemos pensar nas rotas como uma forma de "pesquisar" dados pela nossa aplicação, onde cada nível é um filtro para o proximo parâmetro. Exemplo:

> **Buscar** no módulo de **configurações gerais**, os **webhooks** da **empresa** de ID **1**

Logo nossa URI seria: `GET /geral-configs/companies/1/webhooks`; ou em Laravel:

```php
Route::get('geral-configs/companies/{company}/webhooks', [CompanyWebhookController::class, 'index']);

// Ou melhor

Route::prefix('geral-configs')->name('geral-configs.')->group(function () {
  Route::resource('companies.webhooks', CompanyWebhookController::class);
});

```

Pensando sempre que o dado ou ação que queremos deve estar no final da rota.

## Outras boas práticas para rotas

- Uma rota não deve iniciar com /, exceto se o URL for vazio.

```php
// Ruim
Route::get('', [HomeController::class, 'index']);
Route::get('/open-source', [OpenSourceController::class, 'index']);

// Bom
Route::get('/', [HomeController::class, 'index']);
Route::get('open-source', [OpenSourceController::class, 'index']);
```

- Utilizar nomes de parâmetros como o singular do resource. Utilizar `snake_case`

```php
// Ruim
Route::get('user-profiles/{id}', [UserProfileController::class, 'show']);
Route::get('user-profiles/{userProfile}', [UserProfileController::class, 'show']);
Route::get('user-profiles/{user_profile_id}', [UserProfileController::class, 'show']);

// Bom
Route::get('user-profiles/{user_profile}', [UserProfileController::class, 'show']);
```

- Prefira utilizar a notação de tupla.

```php
// Ruim
Route::post('users/{user}/activate', 'ActivateUserController@store');

// Bom
Route::post('users/{user}/activate', [ActivateUserController::class, 'store']);
```

- Os resources devem ser escritos utilizando o plural do recurso.

```php
// Ruim
Route::resource('user', UserController::class);

// Bom
Route::resource('users', UserController::class);
```

- Em caso do recurso ser composto por mais de uma palavra, devemos utilizar `kebab-case`.

```php
// Ruim
Route::resource('userProfiles', UserProfileController::class);

// Bom
Route::resource('user-profiles', UserProfileController::class);
```

- Essas regras devem ser seguidas inclusive em todos os valores que compõem uma rota, como os prefixos, nomes e ações.

```php
// Ruim
Route::prefix('humanResource')->name('humanResource.')->group(function () {
  Route::resource('exams', ExamController::class);
});

// Bom
Route::prefix('human-resource')->name('human-resource.')->group(function () {
  Route::resource('exams', ExamController::class);
});
```
