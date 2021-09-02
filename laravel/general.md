---
layout: default
title: Geral
description: Padrões gerais.
permalink: /laravel/general
---

# Geral

[:arrow_backward: Voltar](../laravel)

- [**Introdução**](#introdução)
- [**Tipagem**](#tipagem)
  - [Docblocks](#docblocks)
  - [Nullable e Union Types](#nullable-e-union-types)
  - [Retorno Void](#retorno-void)
- [**Traits**](#traits)
- [**Strings**](#strings)
- [**Operador Ternário**](#operador-ternário)
  - [Variáveis Auxiliares](#variáveis-auxiliares)
  - [Extraindo funções](#extraindo-funções)
- [**If**](#if)
  - [Chaves](#chaves)
  - [Happy Path](#happy-path)
  - [Evite `else`](#evite-else)
  - [Condições compostas](#condições-compostas)
- [**Comentários**](#comentários)
- [**Espaçamento**](#espaçamento)

## Introdução

> Muito desse documento é inspirado pelo [guia da Spatie](https://spatie.be/guidelines/laravel-php) com suas devidas mudanças e traduções.

Algumas dessas boas práticas são utilizadas em todas as linguagens e softwares que criamos. Então sempre que possível use-as.

Estilo de código deve seguir [PSR-1](https://www.php-fig.org/psr/psr-1/), [PSR-2](https://www.php-fig.org/psr/psr-2/) e [PSR-12](https://www.php-fig.org/psr/psr-12/). No geral, tudo deve seguir o padrão `camelCase`. Detalhes de exemplos de código detalhados estão em suas respectivas seções.

## Tipagem

Você deve utilizar a tipagem quando possível. Não use docblock.

```php
// Bom
class Foo
{
    public string $bar;
    public int $variable;
}

// Ruim
class Foo
{
    /** @var string */
    public $bar;
}

// Ruim 
class Foo
{
   // Sem tipagem
    public $variable;
}
```

### Docblocks

Não use docblocks para métodos que podem ser totalmente tipados (exceto se você precisa de uma descrição).

Apenas adicione uma descrição para prover mais contexto que a assinatura do próprio método. Use frases completas para descrever, incluindo ponto final.

```php
// Bom
class Url
{
    public static function fromString(string $url): Url
    {
        // ...
    }
}

// Ruim.
class Url
{
    /**
     * Create a url from a string.
     *
     * @param string $url
     *
     * @return \Spatie\Url\Url
     */
    public static function fromString($url)
    {
        // ...
    }
}
```

Se uma variável possui tipos múltiplos, o tipo mais comum deve ser primeiro.

```php
// Bom

/** @var \Spatie\Goo\Bar|null */

// Ruim

/** @var null|\Spatie\Goo\Bar */
```

### Nullable e Union Types

Sempre que possível, use a notação curta de tipo nulo, em vez de utilizar uma união do tipo com `null`.

```php
// Dentro de uma Classe

// Bom
public ?string $variable;

// Ruim
public string | null $variable;
```

### Retorno Void

Se um método retorna nada, deve-se indicar como `void`. Isto torna mais fácil para os desenvolvedores entenderem seu código.

```php
// Bom

// dentro de uma Model
public function scopeArchived(Builder $query): void
{
    $query->
        ...
}
```

## Traits

As traits devem ser escritas todas na mesma linha, seguindo o padrão utilizado dentro do código do Laravel.

```php
// Bom
class MyClass
{
    use TraitA, TraitB;
}

// Ruim
class MyClass
{
    use TraitA;
    use TraitB;
}
```

## Strings

Sempre que possível utilize "string interpolation" com `{}` por volta da variável, ao invés do operador `.`.

```php
// Bom
$greeting = "Hi, I am {$name}.";

// Ruim
$greeting = 'Hi, I am ' . $name . '.';
```

## Operador Ternário

O operador ternário deve ser utilizado sempre que possível, por tornar o código mais legível. Seus principais casos de uso são:

- Atribuição de valores que dependem de uma condicional.

```php
// Bom
$name = $isFoo ? 'foo' : 'bar';

// Ruim
if ($isFoo) {
    $name = 'foo';
} else {
    $name = 'bar';
}
```

- Chamada de métodos que dependem de uma condicional.

```php
// Bom
$condition
    ? $this->doSomething();
    : $this->doSomethingElse();

// Ruim
if ($condition) {
    $this->doSomething();
}
else {
    $this->doSomethingElse();
}
```

> **Obs:** Caso precise quebrar de linha na operador ternário, escreva cada expressão numa linha, antecedida pelo operador como no exemplo acima.

Para conseguirmos utilizar esses casos de usos mais vezes, devemos fazer mudanças no nosso código para ele seja mais legível, e por consequência, mais fácil de manter.

### Variáveis Auxiliares

Devemos criar variáveis auxiliares para armazenar uma condicional mais extensa. Essas variáveis devem iniciar com verbos em inglês que dão o sentido de verdadeiro ou falso, alguns exemplos são:

- `$isFoo`
- `$hasFoo`
- `$canFoo`
- `$shouldFoo`

Para entender mais sobre esse padrão leia esse [artigo](https://www.serendipidata.com/posts/naming-guidelines-for-boolean-variables).

Exemplo:

```php
// Bom
$hasActiveInvoices = !!$user->invoices()->whereActive()->count();

$message = $hasActiveInvoices ? 'User has some invoices.' : 'User has no invoices';

// Ruim
if (!!$user->invoices()->whereActive()->count()) {
  $message = 'User has some invoices.';
} else {
  $message = 'User has no invoices';
}

// Ruim também
$message = !!$user->invoices()->whereActive()->count()
  ? 'User has some invoices.'
  : 'User has no invoices';
```

### Extraindo funções

Devemos criar funções com o corpo de um `if else` e depois utilizar o operador ternário para evitar a duplicação de código e aumentar a legibilidade, já que o próprio nome da função já indicará o que ela faz.

Exemplo:

```php
// Ruim
if ($condition) {
  // código complexo A
} else {
  // código complexo B
}

// Bom
function doSomething() { /** código complexo A */ }
function doSomethingElse() { /** código complexo B */ }

$condition
    ? $this->doSomething();
    : $this->doSomethingElse();
```

## If

Alguns padrões para melhorarmos a legibilidade das nossas condições.

### Chaves

Devemos sempre abrir e fechar chaves.

```php
// Bom
if ($condition) {
   ...
}

// Ruim
if ($condition) ...
```

### Happy Path

Geralmente uma função deve ter o "caminho infeliz" primeiro e o "caminho feliz" por último. Na maioria dos casos isso fará com que o "caminho feliz" esteja sem indentação dentro da função, o que torna o código mais legível.

```php
// Bom
if (!$goodCondition) {
  throw new Exception;
}

// faça algo

// Ruim
if ($goodCondition) {
 // faça algo
}

throw new Exception;
```

### Evite `else`

No geral, `else` deve ser evitado porque ele torna o código menos legível. Em muitos casos, isso pode ser refatorado utilizando [early return](https://dorianneto.com.br/boas-praticas/torne-se-um-ninja-das-funcoes-com-early-return/). Isso também faz o caminho feliz ficar por último, o que é desejável.

```php
// Bom
if (!$conditionA) {
   // condition A falhou

   return;
}

if (!$conditionB) {
   // condition A passou, B falhou

   return;
}

// condition A e B passaram

// Ruim
if ($conditionA) {
   if ($conditionB) {
      // condition A and B passaram
   }
   else {
     // condition A passou, B falhou
   }
}
else {
   // condition A falhou
}
```

### Condições compostas

No geral, deve-se preferir separar as condições complexas ao invés de juntá-las em um único `if`. Isso torna o código mais fácil de debugar.

```php
// Bom
if (!$conditionA) {
   return;
}

if (!$conditionB) {
   return;
}

if (!$conditionC) {
   return;
}

// faça algo


// Ruim
if ($conditionA && $conditionB && $conditionC) {
  // faça algo
}
```

## Comentários

Comentários devem ser evitados ao máximo, escrevendo código descritivo. Se você precisar usar um comentário, formate-o da seguinte forma:

```php
// Deve ter um espaço antes do comentário de uma linha.

/*
 * Se você precisa explicar bastante pode usar um bloco de comentário.
 * Lembrar do * antes de cada linha. Blocos de comentário não
 * precisam ter apenas 3 linhas. Esse é apenas um exemplo.
 */
```

Uma possível estratégia para refatorar e remover os comentários é criar funções com nomes que subtitulam os comentários.

```php
// Bom
$this->calculateLoans();

// Ruim
// Start calculating loans
```

## Espaçamento

Declarações devem ser permitidas _respirar_. No geral, sempre adicione linhas em branco entre declarações, a não ser que elas sejam sequencias de uma linha parecidas. Isso não é algo que se pode impor, é uma questão mais do que parece melhor em cada contexto.

```php
// Bom
public function getPage(string $url): ?Page
{
    $page = $this->pages()->where('slug', $url)->first();

    if (!$page) {
        return null;
    }

    if ($page->private && !auth()->check()) {
        return null;
    }

    return $page;
}

// Ruim: Tudo está amontanhado junto.
public function getPage(string $url): ?Page
{
    $page = $this->pages()->where('slug', $url)->first();
    if (!$page) {
        return null;
    }
    if ($page->private && !auth()->check()) {
        return null;
    }
    return $page;
}
```

```php
// Bom: Uma sequencia de operações equivalentes de uma linha cada.
public function up()
{
    Schema::create('users', function (Blueprint $table) {
        $table->increments('id');
        $table->string('name');
        $table->string('email')->unique();
        $table->string('password');
        $table->rememberToken();
        $table->timestamps();
    });
}
```

Não adicione linhas em branco entre chaves `{}` chaves.

```php
// Bom
if ($foo) {
    $this->foo = $foo;
}

// Ruim
if ($foo) {

    $this->foo = $foo;

}
```

[Voltar para o topo](#geral)
