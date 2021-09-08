---
layout: default
title: DTO
description: Data Transfer Object.
permalink: /laravel/dto
---

# DTO

[:arrow_backward: Voltar](../laravel)

> Documentação da lib: [DTO](https://github.com/spatie/data-transfer-object)

- [**Introdução**](#introdução)
- [**Como criar um DTO?**](#como-criar-um-dto)
- [**Exemplo de como utilizar**](#exemplo-de-como-utilizar)

## Introdução

Os DTO's servem para encapsular dados que serão repassados entre services, ou qualquer outra parte, da nossa aplicação.

Pensamos no seguinte passo-a-passo de exemplo:

- Recebemos do `Request $request` um conjunto de dados que estão armazenados em um `array` associativo.

- Essa dados são passados para uma service que transforma os valores desse array para um padrão específico (como transformar strings em instancias do Carbon).

- Esse array é repassado para algumas outras services que calculam valores extras e adicionam mais dados ao array.

- Por fim, esse array chega até o service final que irá persistir os dados no banco de dados.

Como saber os valores dentro desse array sem ter que dar `dump` em cada um dos services? Como tipar esse array para indicar para quem for usar o service os dados que esse array precisa conter?

## Como criar um DTO?

DTO não passa de uma classe com atributos públicos.

```php
class MyDTO
{
	public string $foo;

	public string $bar;

	public int $baz;

	public OtherDTOCollection $collection;
}
```

Só isso já bastaria para criar um DTO, porém para termos mais funcionalidades que somente uma classe não traz, podemos utilizar a lib `spatie/data-transfer-object` da seguinte maneira:

```php
use Spatie\DataTransferObject\DataTransferObject;

class MyDTO extends DataTransferObject
{
	public string $foo;

	public string $bar;

	public int $baz;

	public OtherDTOCollection $collection;
}
```

Devemos salvar ela no diretório `app/DataTransferObjects`.

Para mais entendimento sobre a lib, buscar na [documentação oficial](https://github.com/spatie/data-transfer-object).

## Exemplo de como utilizar

Veja seguinte código:

```php
class FooController extends Controller
{
	public function store(FooRequest $request)
	{
		$this->fooService->create($request->validated());

		// ...
	}
}

class FooService
{
	public function create(array $data): Foo
	{
		$data['bar'] = 'example';
		$data['value'] = $this->calculateFooService->handle($data);

		// ...

		return Foo::create($data);
	}
}

class CalculateFooService
{
	public function handle(array $data): int
	{
		// ...
	}
}
```

Dentro da função `create` do service `FooService`, antes de chamar `Foo::create`, qual o valor de `$data`? Sabemos que com certeza ele é do tipo `array`, mas e seu conteúdo? Dentro da função `handle` do service `CalculateFooService` como saberemos que é necessário ter uma atributo `bar` dentro do `array $data`?

Vamos refatorar:

```php
class FooDTO extends DataTransferObject
{
	// ...

	public ?string $bar;

	public ?int $value;

	public static function fromRequest(FooRequest $request): self
	{
		return new self($request->validated());
	}
}

class FooController extends Controller
{
	public function store(FooRequest $request)
	{
		$dto = FooDTO::fromRequest($request);

		$this->fooService->create($dto);

		// ...
	}
}

class FooService
{
	public function create(FooDTO $dto): Foo
	{
		$dto->bar = 'example';
		$dto->value = $this->calculateFooService->handle($dto);

		// ...

		return Foo::create($dto->toArray());
	}
}

class CalculateFooService
{
	public function handle(FooDTO $data): int
	{
		// ...
	}
}
```

[Voltar para o topo](#dto)
