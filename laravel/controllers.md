---
layout: default
title: Controllers
description: O controlador das requisições HTTP.
permalink: /laravel/controllers
---

# Controllers

[:arrow_backward: Voltar](../laravel)

> Documentação oficial: [Controllers](https://laravel.com/docs/controllers)

- [**Introdução**](#introdução)
- [**Resource Controllers**](#resource-controllers)
  - [Route Model Bindings](#route-model-bindings)
- [**Single Action Controllers**](#single-action-controllers)
- [**Métodos**](#métodos)
  - [Construtor](#construtor)

## Introdução

Os Controllers são responsáveis por receber as requisições HTTP e responder com a resposta HTTP correta. Não é papel do controller conter regras de negocio, mas sim chamar as classes que detêm essas regras. O controller deve ser pequeno, simples e direto, sem praticamente nenhuma lógica.

Também pensando na diminuição do controller, não devemos ter nenhum HTML dentro dos controllers. Todo e qualquer HTML deve estar dentro de views.

Os controllers devem ser praticamente um reflexo das rotas, já que são elas que vão encaminhar a requisição para o controller certo. Então igual as rotas, devem haver dois tipos de controllers:

## Resource Controllers

Esse tipo de controller pode ser criado com o Laravel usando o comando `php artisan make:controller ResourceController -r` para web e `php artisan make:controller ResourceController --api` para API.

Seu nome deve ser composto pela junção de:

- Singular do recurso (ou recursos)
- "Controller"

```php
// Route::resource('users', UserController::class);
class UserController
{
  // ...
}

// Route::resource('posts.comments', PostCommentController::class);
class PostCommentController
{
  // ...
}
```

Os resources controllers não devem ter nenhum método além dos já criados pelo Laravel automaticamente (index, create, store, show, edit, update, destroy). Caso seja necessário, reavalie a sua forma de pensar nas rotas, poís elas devem ser claras em dizer de qual resource está se tratando. Caso ainda haja duvidas nas rotas releia a [documentação](../laravel/routes).

### Route Model Bindings

> Documentação oficial: [Route Model Bindings](https://laravel.com/docs/routing#route-model-binding)

Todos os métodos do resource que necessitarem de um ID devem utilizar o route model binding, para evitar uma linha a mais de query dentro do controller.

O corpo de um **resource controller** é (`php artisan make:controller UserController --model=User`):

```php
class UserController extends Controller
{
  public function index()
  {
    //
  }

  public function create()
  {
    //
  }

  public function store(Request $request)
  {
    //
  }

  public function show(User $user)
  {
    //
  }

  public function edit(User $user)
  {
    //
  }

  public function update(Request $request, User $user)
  {
    //
  }

  public function destroy(User $user)
  {
    //
  }
}
```

O corpo de um **nested resource controller** é (`php artisan make:controller PostCommentsController --model=Comment --parent=Post`):

```php
class PostCommentsController extends Controller
{
  public function index(Post $post)
  {
    //
  }

  public function create(Post $post)
  {
    //
  }

  public function store(Request $request, Post $post)
  {
    //
  }

  public function show(Post $post, Comment $comment)
  {
    //
  }

  public function edit(Post $post, Comment $comment)
  {
    //
  }

  public function update(Request $request, Post $post, Comment $comment)
  {
    //
  }

  public function destroy(Post $post, Comment $comment)
  {
    //
  }
}
```

> **Obs:** O **nested resource controller** pode não precisar em alguns métodos, como o edit e destroy por exemplo, da model do **parent** (no caso acima do Post). Nesses casos pode ser usado o [Shallow Nested](https://laravel.com/docs/controllers#shallow-nesting) para evitar a necessidade de criar as rotas com o ID do parent, e por consequência evitar a model desnecessária.

## Single Action Controllers

> Documentação oficial: [Single Action Controllers](https://laravel.com/docs/controllers#single-action-controllers)

Esse tipo de controller deve ter apenas um método `__invoke`, e deve ser usado no caso de rotas de ação. Para criar um **single action controller**, use o comando `php artisan make:controller ActionController --invokable`.

Seu nome deve ser composto pela junção de:

- Nome da ação
- Singular do recurso
- "Controller"

Tentando manter o nome do controller com o inglês correto. Exemplo:

```php
// Route::post('users/{user}/verify-delete', VerifyDeleteUserController::class);
class VerifyUserDeleteController
{
  public function __invoke(User $user)
  {
    // ...
  }
}
```

Existe outra opção para Single Action Controller que é separa-los em um pasta chamada `Actions` e criar o controller com o nome sem "Controller". Exemplo:

```php
// Route::post('users/{user}/verify-delete', VerifyDeleteUserController::class);
namespace App\Http\Controllers\Actions;

class VerifyUserDelete
{
  public function __invoke(User $user)
  {
    // ...
  }
}
```

> **Obs:** Essa pasta _Actions_ deve estar dentro do namespace onde normalmente o controller estaria. Por exemplo, se esse controller é do modulo de relatórios seu namespace seria `App\Http\Controllers\Reports\Actions`.

Essa decisão deve ser do time de desenvolvimento, e assim que decidida, todos devem seguir o padrão adotado.

## Métodos

A ordem dos argumentos de um método no controller é:

1. [Request](../laravel/requests)
2. Model Bindings, seguindo a ordem dos parâmetros nas rotas.
3. Outras injeções de dependência, como [Services](../laravel/services).

### Construtor

Não é necessário o uso do método `__construct`, principalmente para instanciar Models. Isso só aumenta o tamanho do controller.

O construtor pode e deve ser usado para Dependency Injection de [Services](../laravel/services) que serão utilizadas por mais de um método. Porém se cada método precisar de um Service diferente, deve-se usar o dependency injection do próprio.

Exemplo:

```php
// Bom
class UserController
{
  public function update(Request $request, User $user, SendEmailService $sendEmailService)
  {
    // ...
  }

  public function delete(User $user, ExternalAPIService $externalApiService)
  {
    // ...
  }
}

// Ruim
class UserController
{
  protected SendEmailService $sendEmailService;
  protected ExternalAPIService $externalApiService;

  public function __construct(SendEmailService $sendEmailService, ExternalAPIService $externalApiService)
  {
    $this->sendEmailService = $sendEmailService;
    $this->externalApiService = $externalApiService;
  }

  public function update(Request $request, User $user)
  {
    // ...
  }

  public function delete(User $user)
  {
    // ...
  }
}
```

> **Obs:** Em hipótese nenhuma o método `__construct` deve ser utilizado para buscas dados na model ou qualquer dado que não seja estático. Isso gera grandes problemas na testabilidade do sistema já que acopla o controller a um dado que não pode ser controlado ou mockado pelos testes.

[Voltar para o topo](#controllers)
