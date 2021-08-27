---
layout: default
title: Validação
description: Tratamento de dados.
permalink: /laravel/validation
---

# Validação

[:arrow_backward: Voltar](../laravel)

> Documentação oficial: [Validation](https://laravel.com/docs/validation)

- [**Introdução**](#introdução)
- [**Form Request**](#form-request)
  - [Acessar dados no Controller](#acessar-dados-no-controller)
  - [Modificar dados do formulário](#modificar-dados-do-formulário)
  - [Mesmo formulário, dados diferentes](#mesmo-formulário-dados-diferentes)

## Introdução

Todos os dados que entram em nossas aplicações devem ser validados antes de chegarem ao Controller. E para isso usaremos os [Form Requests](https://laravel.com/docs/validation#form-request-validation).

Não use validações em controllers.

```php
// Ruim
public function store(Request $request)
{
	$request->validate([
		'title' => 'required|unique:posts|max:255',
		'body' => 'required',
		'publish_at' => 'nullable|date',
	]);

	// ...
}

// Bom
class PostRequest extends Request
{
	public function rules()
	{
		return [
			'title' => ['required', 'unique:posts', 'max:255']
			'body' => ['required'],
			'publish_at' => ['nullable', 'date'],
		];
	}
}

public function store(PostRequest $request)
{
	// ...
}
```

## Form Request

Com os form request podemos fazer várias alterações no nosso formulário.

Por exemplo se precisarmos mudar ou adicionar um campo podemos usar a função `prepareForValidation`. Um bom exemplo disso seria a separação de um grande formulário em vários pequenos "formulários".

Um exemplo seria um formulário que contem dados tanto do usuário quando do seu endereço. O ideal seria separa ambos para que possam ser tratados separadamente.

Além disso os dados do formulário vêm em português, mas apenas trabalhamos com variáveis em ingles. Podemos usar o `prepareForValidation` para fazer a mudança.

```php
class UserAddressRequest extends Request
{
	public function prepareForValidation()
	{
		$this->merge([
				'user' => [
				'name' => $this->nome,
				'email' => $this->email,

				// ...
			],
				'address' => [
				'street' => $this->rua,
				'number' => $this->numero,

				// ...
			]
		]);
	}

	public function rules()
	{
		return [
			'user.name' => // ...
			'user.email' => // ...
			// ...
			'address.street' => // ...
			'address.number' => // ...
			// ...
		];
	}
}
```

Assim dentro do controller todos os dados já estão separados, traduzidos e validados.

### Acessar dados no Controller

No controller devemos usar o `$request->validated()` para acessar todos os dados do formulário. Não devemos utilizar `$request->all()` pois ele abre brechas de segurança, como o envio de dados sem nenhum tipo de validação.

Se precisarmos acessar apenas um dado, devemos utilizar `$request->foo`, sendo `'foo'` o nome do campo. Caso esse valor possa ser `null`, e ele tenha algum valor `'default'`, podemos utilizar `$request->get('foo', 'default')`.

> **Obs:** Não utilize a variável `$request` fora do controller e nem repasse essa variável para outras classes. Sempre trafegue **dados** (arrays, objetos, collections, etc) entre classes e funções.

```php
public function store(UserAddressRequest $request) {
	$form = $request->validated();

	//...
}
```

```php
public function store(UserAddressRequest $request) {
	User::create($request->user);
	Address::create($request->address);

	//...
}
```

### Modificar dados do formulário

Caso seja necessário modificar os dados do formulário, podemos utilizar a função `passedValidation`.

Um exemplo seria transformar um string de data em uma instancia do Carbon.

```php
class SomeRequest extends Request
{
	public function rules()
	{
		return [
			'start_date' => ['required', 'date',  'date_format:Y-m-d\TH:i'],
		];
	}

	protected function passedValidation()
	{
		$this->replace([
			'start_date' => Carbon::createFromFormat('Y-m-d\TH:i',  $this->start_date),
		]);
	}
}
```

### Mesmo formulário, dados diferentes

Caso precisemos utilizar o mesmo formulário para vários controllers, podemos criar funções publicas dentro do FormRequest que nos retornem apenas os dados necessários para cada situação.

No entanto, deve-se tomar cuidado para não tentar reutilizar o mesmo formulário para casos diferentes, onde as validações sejam diferentes.
Um exemplo de mal uso seria, por exemplo, criar o mesmo formulário para criar um usuário e para editar o usuário, sendo que as validações não serão as mesmas.

Vamos a um exemplo. Imagine que temos três formulários iguais que seus dados irão para API's externas diferentes. E cada API dessa precisa desses dados com nomes dos campos diferentes. Poderíamos fazer assim:

```php
/*
 * API 1 - Lorem ipsum
 * API 2 - Convallis Sit
 * API 3 - Dolor Sit
 */
class SomeRequest extends Request
{
	public function rules()
	{
		return [
			'name' => ['required'],
			'email' => ['required'],
			'date' => ['required', 'date'],
			'http' => ['required'],
		];
	}

	public function loremPayload()
	{
		return [
			'user_name' => $this->name,
			'mail' => $this->email,
			'date_time' => $this->date,
			'http_url' => $this->http
		];
	}

	public function convallisPayload()
	{
		return [
			'nome' => $this->name,
			'seuEmail' => $this->email,
			'data' => $this->date,
			'url' => $this->http
		];
	}

	public function dolorPayload()
	{
		return [
			'UFullName' => $this->name,
			'VEmail' => $this->email,
			'DTime' => $this->date,
			'Url' => $this->http
		];
	}
}
```

E então dentro de cada controller, podemos algo como isso:

```php
class LoremController extends Controller
{
	public function store(SomeRequest $request)
	{
		$form = $request->loremPayload();

		// ...
	}
}
```

[Voltar para o topo](#validação)
