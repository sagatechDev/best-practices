---
layout: default
title: Services
description: O cérebro da nossa aplicação.
permalink: /laravel/services
---

# Services

[:arrow_backward: Voltar](../laravel)

- [**Introdução**](#introdução)
- [**Action Service**](#action-service)
- [**Services de Contexto**](#services-de-contexto)

## Introdução

Serão nas services que manteremos as regras de negócio da aplicação. Elas devem ser pensadas para executarem um função especifica no nosso sistema, além de que devem tentar ser ao máximo isoladas de outras regras para que, sua mudança ou remoção não afete nossa aplicação como um todo.

O Laravel não fala sobre as Services em sua documentação, deixando em aberto para a comunidade decidir como fazer essa separação das regras de negócio.

No fim a service não passa de uma classe com um método que recebe parâmetros, executa uma ação e devolve valores ou gera side-effects.

As services não devem depender de dados e funções que funcionam só no contexto da requisição web ou do usuário estar ou não autenticado, poís isso impossibilita reutilizá-la em outras situações, como nos comandos do console.

As services devem estar localizadas em `app/Services`.

Vão haver dois tipos de services:

## Action Service

Assim como houve com as rotas e controllers, teremos também services que executam uma única ação. Essas serão usadas em casos de fluxos complexos onde para fazer um ação vários passos precisam ser executados. Muito provavelmente essa service, dependendo do seu tamanho, pode ser dividida em várias services menores para reaproveita-las em vários locais.

Um exemplo seria a ação de gerar um extrato mensal de um determinado grupo.

Para criar o nome dessas services, devemos usar o nome da ação que ela executa, por exemplo, se a ação é **GenerateGroupExtract**, a service será **GenerateGroupExtractService**.

Essa classe deve conter apenas um método publico com o nome de `handle`, `execute` ou `run` (ou qualquer outro que dê o sentido de executar a ação). Caso necessário, pode haver funções privados para facilitar o entendimento e separar a ação em passos menores para que possam ser de fácil manutenção.

> **Obs:** O nome do método deve ser decido entre a equipe e seguido por todos.

```php
class GenerateGroupExtractService
{
	protected CalculateTaxesService $calculateTaxesService;

	public function __constructor(CalculateTaxesService $calculateTaxesService)
	{
		$this->calculateTaxesService = $calculateTaxesService;
	}


	public function handle(Group $group): Extract
	{
		$valuesCalculated = $this->calculateExtract($group);

		// ...

		return $this->generateExtract($someData);
	}

	private function calculateExtract(Group $group): array
	{
		$taxesCalculated = $this->calculateTaxesService->handle($group->taxes);

		// ...
	}

	private function generateExtract(array $data): Extract
	{
		// ...
	}
}
```

## Services de Contexto

Essas services, ao contrário das action services, devem ser criadas para agrupar algumas ações relacionadas a um único contexto.

> **Obs:** Cuidado para não criar um contexto grande demais e com muitos métodos.

Um exemplo seria um service ligada a uma model. Ele pode ser usado para criar, editar e deletar um determinado model, onde suas regras de negocio sejam diferentes de simplesmente trabalhar com o banco de dados.

```php
class UserService
{
	protected CheckActiveUserService $checkActiveUserService;

	public function __constructor(CheckActiveUserService $checkActiveUserService)
	{
		$this->checkActiveUserService = $checkActiveUserService;
	}

	public function create(array $data, User $parentUser): User
	{
		$isParentUserActive = $this->checkActiveUserService->handle($parentUser);

		if (!$isParentUserActive) {
			throw new ParentUserNotActiveException();
		}

		$data['parent_user_id'] = $parentUser->id;

		// ...

		return User::create($data);
	}

	public function update(User $user, array $data): User
	{
		// ...
	}

	public function delete(User $user): void
	{
		// ...
	}
}
```

> **Obs:** Repare que no método `create` recebe o usuário pai desse novo usuário por parâmetro, mesmo que esse fosse o usuário autenticado na aplicação. Não devemos utilizar `auth()->user()` dentro do service para que ele possa ser utilizado em outros contextos.

Outro exemplo seria um service que executa ações relacionada a API de terceiros.

```php
class ExternalAPIService
{
	private string $host;
	private array $headers;
	private array $config;

	public function __construct()
	{
		$this->host = config('external-api.host');
		$this->config = config('external-api.config');
		$this->headers = config('external-api.headers');
	}

	public function post(array $bodyRequest, string $uri, array $additionalData = [])
	{
		// ...

		return $response;
	}

	public function get(array $queryParameters, string $uri, array $additionalData = [])
	{
		// ...

		return $response;
	}
}
```

Nesse caso, a service não ser separada em várias Actions Services ajuda a não repetir o código do construtor em vários locais, além de centralizar as ações que se podem ser feitas com a API externa em um único local.

[Voltar para o topo](#services)
