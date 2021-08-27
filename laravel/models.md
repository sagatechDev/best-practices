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

Podemos ter nas models:

- Relacionamentos
- Escopos
- Funções que grupam ações no DB

Não devemos utilizar a Model em outr

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

## Escopos

Escopos devem ser usados para criar filtros padrões da Model que contenham um nome que explique o que o filtro faz. Exemplo:

```php
// Ruim quando usado fora da Model
Model::where('active', 1)->get();

// Bom
Model::whereActive()->get();
```

Esse caso é simples, mas mostra como os escopos podem ajudar a organizar os filtros na Model de maneira mais descritível e menos pensando em queries no banco de dados, o que ajuda a novos desenvolvedores a entenderem o que uma query complexa faz.

[Voltar para o topo](#services)
