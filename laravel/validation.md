---
layout: default
title: Validação
description: Tratamento de dados.
permalink: /laravel/validation
---

# Validação

[:arrow_backward: Voltar](../laravel)

> Documentação oficial: [Validation](https://laravel.com/docs/validation)

- [Introdução](#introdução)

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

    ....
}

// Bom
public function store(PostRequest $request)
{
    ....
}

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
```

## Form Request

Com os form request podemos fazer várias alterações no nosso formulário.

Por exemplo se precisarmos mudar ou adicionar um campo podemos usar a função `prepareForValidation`. Um bom exemplo disso seria a separação de um grande formulário em vários pequenos "formulários".

Um exemplo seria um formulário que contem dados tanto do usuário quando do seu endereço. O ideal seria separa ambos para que possam ser tratados separadamente.

Além disso os dados do formulário vêm em português, mas apenas trabalhamos com variáveis em ingles. Podemos usar o `prepareForValidation` para fazer a mudança.

```php
class PostRequest extends Request
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
public function store(PostRequest $request) {
	Post::create($request->validated());

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
			'start_date' => ['required', 'date', 'date_format:Y-m-d\TH:i'],
		];
	}

	protected function passedValidation()
	{
		$this->replace([
			'start_date' => Carbon::createFromFormat('Y-m-d\TH:i', $this->start_date),
		]);
  }
}
```

[Voltar para o topo](#validação)
