# Clean Code PHP

针对[英文版Clean Code PHP](https://github.com/jupeter/clean-code-php)的一些翻译和自己的理解，不定期补充

## Table of Contents

  1. [简介](#简介)
  2. [变量](#变量)
     * [使用有意义且可读的变量名称](#使用有意义且可读的变量名称)
     * [对于同一类型的变量使用相同的词汇表](#对于同一类型的变量使用相同的词汇表)
     * [使用易于查找的变量名 (part 1)](#使用易于查找的变量名-part-1)
     * [使用易于查找的变量名 (part 2)](#使用易于查找的变量名-part-2)
     * [使用解释型变量](#使用解释型变量)
     * [避免嵌套太深尽量提早return (part 1)](#避免嵌套太深尽量提早return-part-1)
     * [避免嵌套太深尽量提早return (part 2)](#避免嵌套太深尽量提早return-part-2)
     * [避免意识流式映射](#避免意识流式映射)
     * [避免过度的可读](#避免过度的可读)
     * [正确使用默认参数避免短路运算和条件运算](#正确使用默认参数避免短路运算和条件运算)
  3. [函数](#函数)
     * [函数参数](#函数参数)
     * [函数功能单一性](#函数功能单一性)
     * [方法名标识功能](#方法名标识功能)
     * [每一个函数一个抽象层级](#每一个函数一个抽象层级)
     * [不要使用标识型参数](#不要使用标识型参数)
     * [避免副作用](#避免副作用)
     * [避免写全局方法](#避免写全局方法)
     * [避免使用单例模式](#避免使用单例模式)
     * [封装条件](#封装条件)
     * [避免消极条件](#避免消极条件)
     * [避免多条件](#避免多条件)
     * [避免类型检查 (part 1)](#避免类型检查-part-1)
     * [避免类型检查 (part 2)](#避免类型检查-part-2)
     * [删除死代码](#删除死代码)
  4. [对象和数据结构](#对象和数据结构)
     * [使用对象封装](#使用对象封装)
     * [对象中应该包含private/protected成员](#对象中应该包含privateprotected成员)
  5. [类](#类)
     * [组合优于继承](#组合优于继承)
     * [避免链式调用](#避免链式调用)
  6. [SOLID](#solid)
     * [单一职责原则 (SRP)](#单一职责原则-srp)
     * [开闭原则 (OCP)](#开闭原则-ocp)
     * [里氏替换原则 (LSP)](#里氏替换原则-lsp)
     * [Interface Segregation Principle (ISP)](#interface-segregation-principle-isp)
     * [Dependency Inversion Principle (DIP)](#dependency-inversion-principle-dip)
  7. [Don’t repeat yourself (DRY)](#dont-repeat-yourself-dry)
  8. [Translations](#translations)

## 简介

这只是一个PHP简洁代码指导方针，不是硬性的编码标准，所以并不是每条规则都需要遵守，甚至有些不是被大多数coder认同的
灵感来自于[clean-code-javascript](https://github.com/ryanmcdermott/clean-code-javascript)
虽然很多coder依然在使用PHP 5，但该文档的很多例子是基于PHP 7.1+的

## 变量

### 使用有意义且可读的变量名称

**糟糕的:**

```php
$ymdstr = $moment->format('y-m-d'); //ymdstr的可读性很差
```

**优雅的:**

```php
$currentDate = $moment->format('y-m-d');
```

**[⬆  回顶部](#table-of-contents)**

### 对于同一类型的变量使用相同的词汇表

**糟糕的:**

```php
getUserInfo();
getUserData();
getUserRecord();
getUserProfile();
```

**优雅的:**

```php
getUser();
```

**[⬆  回顶部](#table-of-contents)**

### 使用易于查找的变量名 (part 1)

一般我们阅读的代码量远远大于写的代码量，所以书写可读易查的代码是很重要的。如果你命名的变量名**不能**帮助读者理解你的程序，这将令读者非常头疼，所以尽量书写易读的代码吧

**糟糕的:**

```php
// 448到底表示什么呢？一个数字的意义是无穷的
$result = $serializer->serialize($data, 448);
```

**优雅的:**

```php
$json = $serializer->serialize($data, JSON_UNESCAPED_SLASHES | JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE);
```

### 使用易于查找的变量名 (part 2)

**糟糕的:**

```php
// 4到底表示什么？
if ($user->access & 4) {
    // ...
}
```

**优雅的:**

```php
class User
{
    const ACCESS_READ = 1;
    const ACCESS_CREATE = 2;
    const ACCESS_UPDATE = 4;
    const ACCESS_DELETE = 8;
}

if ($user->access & User::ACCESS_UPDATE) {
    // do edit ...
}
```

**[⬆  回顶部](#table-of-contents)**

### 使用解释型变量

**糟糕的:**

matches[1]，match[2]是无意义的参数名

```php
$address = 'One Infinite Loop, Cupertino 95014';
$cityZipCodeRegex = '/^[^,]+,\s*(.+?)\s*(\d{5})$/';
preg_match($cityZipCodeRegex, $address, $matches);

saveCityZipCode($matches[1], $matches[2]);
```

**不那么糟糕的:**

这种方式相对好点，但我们还是严重依赖于正则，matches有可能是空的

```php
$address = 'One Infinite Loop, Cupertino 95014';
$cityZipCodeRegex = '/^[^,]+,\s*(.+?)\s*(\d{5})$/';
preg_match($cityZipCodeRegex, $address, $matches);

[, $city, $zipCode] = $matches;
saveCityZipCode($city, $zipCode);
```

**优雅的:**

通过修改正则设置子模式来减少对正则的依赖，matches必然会有key(city,zipCode)

```php
$address = 'One Infinite Loop, Cupertino 95014';
$cityZipCodeRegex = '/^[^,]+,\s*(?<city>.+?)\s*(?<zipCode>\d{5})$/';
preg_match($cityZipCodeRegex, $address, $matches);

saveCityZipCode($matches['city'], $matches['zipCode']);
```

**[⬆  回顶部](#table-of-contents)**

### 避免嵌套太深尽量提早return (part 1)

太多的if else嵌套只会让你的代码阅读起来十分艰难，简单永远比复杂好

**糟糕的:**

```php
function isShopOpen($day): bool
{
    if ($day) {
        if (is_string($day)) {
            $day = strtolower($day);
            if ($day === 'friday') {
                return true;
            } elseif ($day === 'saturday') {
                return true;
            } elseif ($day === 'sunday') {
                return true;
            } else {
                return false;
            }
        } else {
            return false;
        }
    } else {
        return false;
    }
}
```

**优雅的:**

```php
function isShopOpen(string $day): bool
{
    if (empty($day)) {
        return false;  // 提前return
    }

    $openingDays = [
        'friday', 'saturday', 'sunday'
    ];

    return in_array(strtolower($day), $openingDays, true);
}
```

**[⬆  回顶部](#table-of-contents)**

### 避免嵌套太深尽量提早return (part 2)

**糟糕的:**

```php
function fibonacci(int $n)
{
    if ($n < 50) {
        if ($n !== 0) {
            if ($n !== 1) {
                return fibonacci($n - 1) + fibonacci($n - 2);
            } else {
                return 1;
            }
        } else {
            return 0;
        }
    } else {
        return 'Not supported';
    }
}
```

**优雅的:**

```php
function fibonacci(int $n): int
{
    if ($n === 0 || $n === 1) {
        return $n;
    }

    if ($n > 50) {
        throw new \Exception('Not supported');
    }

    return fibonacci($n - 1) + fibonacci($n - 2);
}
```

**[⬆  回顶部](#table-of-contents)**

### 避免意识流式映射

不要强迫读者费劲地去理解其中变量代表的意思，至简很重要

**糟糕的:**

```php
$l = ['Austin', 'New York', 'San Francisco'];

for ($i = 0; $i < count($l); $i++) {
    $li = $l[$i];  //$l[$i]需要结合上下文才能get到意思
    doStuff();
    doSomeOtherStuff();
    // ...
    // ...
    // ...
    // Wait, what is `$li` for again?
    dispatch($li);  //$li同上
}
```

**优雅的:**

```php
$locations = ['Austin', 'New York', 'San Francisco'];

foreach ($locations as $location) {
    doStuff();
    doSomeOtherStuff();
    // ...
    // ...
    // ...
    dispatch($location);  //通过location很直白知道表达的意思是位置
}
```

**[⬆ 回顶部](#table-of-contents)**

### 避免过度的可读

如果你的类/对象名已经表达了一些信息，那么就没有必要在变量名中重复表达了

**糟糕的:**

```php
class Car
{
    public $carMake;
    public $carModel;
    public $carColor;

    //...
}
```

**优雅的:**

```php
class Car
{
    public $make;
    public $model;
    public $color;

    //...
}
```

**[⬆  回顶部](#table-of-contents)**

### 正确使用默认参数避免短路运算和条件运算

**不太优雅:**

`$breweryName` 有可能是 `NULL`

```php
function createMicrobrewery($breweryName = 'Hipster Brew Co.'): void
{
    // ...
}
```

**不太糟糕:**

比上个版本更易理解，增加三目运算来避免`null`值

```php
function createMicrobrewery($name = null): void
{
    $breweryName = $name ?: 'Hipster Brew Co.';
    // ...
}
```

**优雅的:**

在PHP 7+中, 你可以使用 [参数类型约束](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration) 来保证 `$breweryName` 不会是 `NULL`

```php
function createMicrobrewery(string $breweryName = 'Hipster Brew Co.'): void
{
    // ...
}
```

**[⬆  回顶部](#table-of-contents)**

## 函数

### 函数参数

限制参数个数是极其重要的，因为这样会使得对应的测试变得简单。如果函数有超过3个参数，将导致参数组合变得多样化，也意味着将有大量的场景等着你测试

0参数是最理想的，1到2个也是ok的，但是应该避免3个及以上。如果你的函数需要超过2个参数，那就要考虑把功能拆分到多个函数中，特殊情况下，可以考虑使用更高级的对象作为参数来传递信息

**糟糕的:**

```php
function createMenu(string $title, string $body, string $buttonText, bool $cancellable): void
{
    // ...
}
```

**优雅的:**

```php
class MenuConfig
{
    public $title;
    public $body;
    public $buttonText;
    public $cancellable = false;
}

$config = new MenuConfig();
$config->title = 'Foo';
$config->body = 'Bar';
$config->buttonText = 'Baz';
$config->cancellable = true;

function createMenu(MenuConfig $config): void
{
    // ...
}
```

**[⬆  回顶部](#table-of-contents)**

### 函数功能单一性

这应该是目前为止最最最重要的软件工程规则了。如果一个函数集成了过多的功能，它将变得很难撰写、测试和理解。单一功能的函数更加的易读易重构，所以请严格遵守这条规则，这将使你与其他无视该规则的coder拉开距离

**糟糕的:**

`emailClients`包含了查用户功能和检查是否活跃功能

```php
function emailClients(array $clients): void
{
    foreach ($clients as $client) {
        $clientRecord = $db->find($client);
        if ($clientRecord->isActive()) {
            email($client);
        }
    }
}
```

**优雅的:**

`emailClients`发邮件给活跃用户功能，不再集成查活跃用户功能
`activeClients`返回活跃用户功能
`isClientActive`判断用户是否活跃功能

```php
function emailClients(array $clients): void
{
    $activeClients = activeClients($clients);
    array_walk($activeClients, 'email');
}

function activeClients(array $clients): array
{
    return array_filter($clients, 'isClientActive');
}

function isClientActive(int $client): bool
{
    $clientRecord = $db->find($client);

    return $clientRecord->isActive();
}
```

**[⬆  回顶部](#table-of-contents)**

### 方法名标识功能

**糟糕的:**

```php
class Email
{
    //...

    public function handle(): void
    {
        mail($this->to, $this->subject, $this->body);
    }
}

$message = new Email(...);
// 无法明确handle的具体功能
$message->handle();
```

**优雅的:**

```php
class Email 
{
    //...

    public function send(): void
    {
        mail($this->to, $this->subject, $this->body);
    }
}

$message = new Email(...);
// 一目了然
$message->send();
```

**[⬆  回顶部](#table-of-contents)**

### 每一个函数一个抽象层级

当你的函数太多不同抽象层级的代码混在一起的时候，是极其不易于复用和测试的，可读性也很差

**糟糕的:**

```php
function parseBetterJSAlternative(string $code): void
{
    $regexes = [
        // ...
    ];

    $statements = explode(' ', $code);
    $tokens = [];
    foreach ($regexes as $regex) {
        foreach ($statements as $statement) {
            // ...
            // 获取tokens
        }
    }

    $ast = [];
    foreach ($tokens as $token) {
        // lex...
        // 依赖于token，获取ast，与获取token不是同一抽象层
    }

    foreach ($ast as $node) {
        // parse...
        // 依赖于ast，得到最终结果，与获取ast不是同一抽象层
    }
}
```

**糟糕的:**

我们已经拆分除了一些功能，把代码中的不同抽象层解耦出来，但是`parseBetterJSAlternative()`依然是很复杂不易测试的

```php
function tokenize(string $code): array
{
    $regexes = [
        // ...
    ];

    $statements = explode(' ', $code);
    $tokens = [];
    foreach ($regexes as $regex) {
        foreach ($statements as $statement) {
            $tokens[] = /* ... */;
        }
    }

    return $tokens;
}

function lexer(array $tokens): array
{
    $ast = [];
    foreach ($tokens as $token) {
        $ast[] = /* ... */;
    }

    return $ast;
}

function parseBetterJSAlternative(string $code): void
{
    $tokens = tokenize($code);
    $ast = lexer($tokens);
    foreach ($ast as $node) {
        // parse...
    }
}
```

**优雅的:**

最好的方案是把`parseBetterJSAlternative()`依赖的东西剥离出来，类`BetterJSAlternative`只需要知道相关的处理类(`Tokenizer`,`Lexer`)就可以了，至于是否重载这两个类改写对应的处理方法，都是很方便哒

```php
class Tokenizer
{
    public function tokenize(string $code): array
    {
        $regexes = [
            // ...
        ];

        $statements = explode(' ', $code);
        $tokens = [];
        foreach ($regexes as $regex) {
            foreach ($statements as $statement) {
                $tokens[] = /* ... */;
            }
        }

        return $tokens;
    }
}

class Lexer
{
    public function lexify(array $tokens): array
    {
        $ast = [];
        foreach ($tokens as $token) {
            $ast[] = /* ... */;
        }

        return $ast;
    }
}

class BetterJSAlternative
{
    private $tokenizer;
    private $lexer;

    public function __construct(Tokenizer $tokenizer, Lexer $lexer)
    {
        $this->tokenizer = $tokenizer;
        $this->lexer = $lexer;
    }

    public function parse(string $code): void
    {
        $tokens = $this->tokenizer->tokenize($code);
        $ast = $this->lexer->lexify($tokens);
        foreach ($ast as $node) {
            // parse...
        }
    }
}
```

**[⬆  回顶部](#table-of-contents)**

### 不要使用标识型参数

使用标识性参数意味着你的函数将需要做更多的事，这违反了单一功能规则。如果你的函数需要根据标识进行`if else`，拆分他们

**糟糕的:**

`$temp`bool类型的标识

```php
function createFile(string $name, bool $temp = false): void
{
    if ($temp) {
        touch('./temp/'.$name);
    } else {
        touch($name);
    }
}
```

**优雅的:**

```php
function createFile(string $name): void
{
    touch($name);
}

function createTempFile(string $name): void
{
    touch('./temp/'.$name);
}
```

**[⬆  回顶部](#table-of-contents)**

### 避免副作用

一个包含输入和输出的函数可能会产生副作用。这种副作用可能是写某个文件、改变某个全局变量甚至可能把你账户的钱打给别人
有时候你可能需要产生这些副作用，像前面说到的写某个文件，但你必须明确在哪些地方写入。不要在多个函数和类里写入同一个文件，使用单一的服务去做这件事情
需要注意的是避免在对象之间传递无结构的状态或使用可变的数据，不要忽略代码中可能出现副作用的地方，如果能做到上面这些，你的coding将会更加轻松加愉快

**糟糕的:**

`splitIntoFirstAndLastName()`对global变量`$name`进行了更改，会影响其他使用`$name`的函数

```php
$name = 'Ryan McDermott';

function splitIntoFirstAndLastName(): void
{
    global $name;

    $name = explode(' ', $name);
}

splitIntoFirstAndLastName();

var_dump($name); // ['Ryan', 'McDermott'];
```

**优雅的:**

```php
function splitIntoFirstAndLastName(string $name): array
{
    return explode(' ', $name);
}

$name = 'Ryan McDermott';
$newName = splitIntoFirstAndLastName($name);

var_dump($name); // 'Ryan McDermott';
var_dump($newName); // ['Ryan', 'McDermott'];
```

**[⬆  回顶部](#table-of-contents)**

### 避免写全局方法

在很多语言中，污染全局变量是一个糟糕的实践，因为这可能导致程序内部包之间的冲突
举个栗子：你有一个生成全局配置的方法`config()`，当你引入其他包含相同方法的包时，会产生冲突

**糟糕的:**

```php
function config(): array
{
    return  [
        'foo' => 'bar',
    ]
}
```

**优雅的:**

```php
class Configuration
{
    private $configuration = [];

    public function __construct(array $configuration)
    {
        $this->configuration = $configuration;
    }

    public function get(string $key): ?string
    {
        return isset($this->configuration[$key]) ? $this->configuration[$key] : null;
    }
}
```

//加载`configuration`并且创建实例
```php
$configuration = new Configuration([
    'foo' => 'bar',
]);
```

现在你只需要在你的项目中通过`Configuration`来生成配置即可

**[⬆  回顶部](#table-of-contents)**

### 避免使用单例模式

单例模式是一种[反模式](https://en.wikipedia.org/wiki/Singleton_pattern)。
转述自Brian Button：
1.他们通常会使用一个全局实例，为什么这种做法是糟糕的？因为你隐藏了你项目代码中的相关依赖，而没有通过接口将这些依赖暴露出来。通过全局方式来避免传递是一种[code smell](https://en.wikipedia.org/wiki/Code_smell)
2.他们违背了[单一职责原则](单一职责原则)，因为**他们自己控制了创建和生命周期**
3.他们本质上使代码[藕合](https://en.wikipedia.org/wiki/Coupling_%28computer_programming%29)。在很多情况下，这使得测试变得艰难
4.单例模式的全局实例将持续整个项目周期。这就造成了另外一个缺陷，在某些场景你会需要按照一定的顺序进行测试，这对于单元测试来说是糟糕的，因为每一个单元测试都应该是相互独立的

来自[Misko Hevery](http://misko.hevery.com/about/)关于[root of problem](http://misko.hevery.com/2008/08/25/root-cause-of-singletons/)的想法也很不错

**糟糕的:**

```php
class DBConnection
{
    private static $instance;

    private function __construct(string $dsn)
    {
        // ...
    }

    public static function getInstance(): DBConnection
    {
        if (self::$instance === null) {
            self::$instance = new self();
        }

        return self::$instance;
    }

    // ...
}

$singleton = DBConnection::getInstance();
```

**优雅的:**

```php
class DBConnection
{
    public function __construct(string $dsn)
    {
        // ...
    }

     // ...
}
```

// 创建实例并初始化

```php
$connection = new DBConnection($dsn);
```

**[⬆  回顶部](#table-of-contents)**

### 封装条件

**糟糕的:**

```php
if ($article->state === 'published') {
    // ...
}
```

**优雅的:**

```php
if ($article->isPublished()) {
    // ...
}
```

**[⬆  回顶部](#table-of-contents)**

### 避免消极条件

**糟糕的:**

```php
function isDOMNodeNotPresent(\DOMNode $node): bool
{
    // ...
}

if (!isDOMNodeNotPresent($node))
{
    // ...
}
```

**优雅的:**

```php
function isDOMNodePresent(\DOMNode $node): bool
{
    // ...
}

if (isDOMNodePresent($node)) {
    // ...
}
```

**[⬆  回顶部](#table-of-contents)**

### 避免多条件

乍一看，这貌似是不可能的，很多人会不知道如何在代码中不使用`if`。你可以使用多态来替代`if`，正如前面所说的，函数功能单一性，当你的函数中用到`if`，这意味着你的函数做了不止一件事情，时刻记住函数功能单一性

**糟糕的:**

```php
class Airplane
{
    // ...

    public function getCruisingAltitude(): int
    {
        switch ($this->type) {
            case '777':
                return $this->getMaxAltitude() - $this->getPassengerCount();
            case 'Air Force One':
                return $this->getMaxAltitude();
            case 'Cessna':
                return $this->getMaxAltitude() - $this->getFuelExpenditure();
        }
    }
}
```

**优雅的:**

```php
interface Airplane
{
    // ...

    public function getCruisingAltitude(): int;
}

class Boeing777 implements Airplane
{
    // ...

    public function getCruisingAltitude(): int
    {
        return $this->getMaxAltitude() - $this->getPassengerCount();
    }
}

class AirForceOne implements Airplane
{
    // ...

    public function getCruisingAltitude(): int
    {
        return $this->getMaxAltitude();
    }
}

class Cessna implements Airplane
{
    // ...

    public function getCruisingAltitude(): int
    {
        return $this->getMaxAltitude() - $this->getFuelExpenditure();
    }
}
```

**[⬆  回顶部](#table-of-contents)**

### 避免类型检查 (part 1)

PHP是弱类型语言，这意味着你的函数可以使用任何类型的参数。有时候你需要在代码中对类型进行检查，有很多方法避免这种检查，首当其冲的就是统一APIs

**糟糕的:**

```php
function travelToTexas($vehicle): void
{
    if ($vehicle instanceof Bicycle) {
        $vehicle->pedalTo(new Location('texas'));
    } elseif ($vehicle instanceof Car) {
        $vehicle->driveTo(new Location('texas'));
    }
}
```

**Good:**

// 统一使用`travelTo()`方法，更多的细节放到实现中去做

```php
function travelToTexas(Traveler $vehicle): void
{
    $vehicle->travelTo(new Location('texas'));
}
```

**[⬆  回顶部](#table-of-contents)**

### 避免类型检查 (part 2)

在PHP7+中，如果不使用多态，可以使用新特性--参数类型检查，避免人为判断造成的复杂代码

**糟糕的:**

```php
function combine($val1, $val2): int
{
    if (!is_numeric($val1) || !is_numeric($val2)) {
        throw new \Exception('Must be of type Number');
    }

    return $val1 + $val2;
}
```

**优雅的:**

```php
function combine(int $val1, int $val2): int
{
    return $val1 + $val2;
}
```

**[⬆  回顶部](#table-of-contents)**

### 删除死代码

死代码就是那些不会执行到的无用代码，不要犹豫，删掉它，反正你还能在旧版本中找到它

**糟糕的:**

```php
function oldRequestModule(string $url): void
{
    // ...
}

function newRequestModule(string $url): void
{
    // ...
}

$request = newRequestModule($requestUrl);
inventoryTracker('apples', $request, 'www.inventory-awesome.io');
```

**优雅的:**

```php
function requestModule(string $url): void
{
    // ...
}

$request = requestModule($requestUrl);
inventoryTracker('apples', $request, 'www.inventory-awesome.io');
```

**[⬆  回顶部](#table-of-contents)**


## 对象和数据结构

### 使用对象封装

在PHP中你可以对方法使用关键字`public protected private`，通过这些关键字可以控制对象上的属性修改。

* 当你需要做更多操作而不仅仅是获取类属性时，你不需要遍历你的代码库去修改所有获取的入口
* 当你使用`set`来修改属性时，更易于验证
* 封装内部逻辑
* 当`getting`和`setting`时更容易添加日志和错误处理
* 继承重载
* 你可以为一个服务延迟的去获取这个对象的属性值

此外，这些规则同样适用于[Open/Closed](#openclosed-principle-ocp)

**糟糕的:**

```php
class BankAccount
{
    public $balance = 1000;
}

$bankAccount = new BankAccount();

// Buy shoes...
$bankAccount->balance -= 100;
```

**优雅的:**

```php
class BankAccount
{
    private $balance;

    public function __construct(int $balance = 1000)
    {
      $this->balance = $balance;
    }

    public function withdraw(int $amount): void
    {
        if ($amount > $this->balance) {
            throw new \Exception('Amount greater than available balance.');
        }

        $this->balance -= $amount;
    }

    public function deposit(int $amount): void
    {
        $this->balance += $amount;
    }

    public function getBalance(): int
    {
        return $this->balance;
    }
}

$bankAccount = new BankAccount();

// Buy shoes...
$bankAccount->withdraw($shoesPrice);

// Get balance
$balance = $bankAccount->getBalance();
```

**[⬆  回顶部](#table-of-contents)**

### 对象中应该包含private/protected成员

* `public`方法和属性被修改是很危险的，因为外部代码可以很轻易地依赖他们，而你毫无办法。**对于所有用户的类来说，在类中可以修改是相当危险的**
* `protected` 方法和属性的改动同`public`一样危险，因为他们对所有子类都是可见的，仅仅是访问机制不一样，封装没变，**对于所有子类来说，在类中修改也是相当危险的**
* `private` 保证了代码只有在**自己类的内部修改才是危险的**

因此，默认使用`private`，当你需要提供给外部类访问时使用`public/protected`
阅读[Fabien Potencier](https://github.com/fabpot)中的[blog post](http://fabien.potencier.org/pragmatism-over-theory-protected-vs-private.html)来获取更多相关信息

**糟糕的:**

```php
class Employee
{
    public $name;

    public function __construct(string $name)
    {
        $this->name = $name;
    }
}

$employee = new Employee('John Doe');
echo 'Employee name: '.$employee->name; // Employee name: John Doe
```

**优雅的:**

```php
class Employee
{
    private $name;

    public function __construct(string $name)
    {
        $this->name = $name;
    }

    public function getName(): string
    {
        return $this->name;
    }
}

$employee = new Employee('John Doe');
echo 'Employee name: '.$employee->getName(); // Employee name: John Doe
```

**[⬆  回顶部](#table-of-contents)**

## 类

### 组合优于继承

正如the Gang of Four在著名的[*Design Patterns*](https://en.wikipedia.org/wiki/Design_Patterns)中所说，应该尽可能的使用组合代替继承。组合和继承各有各的优点，但是在你本能的使用继承的时候，不妨考虑下是否可以使用组合解决同样的问题。
你也许会疑惑，什么时候该使用继承呢，这取决于你要处理的问题，以下是继承优于组合的场景：

1. 继承表示"is-a"关系，不是"has-a"(Human is-a Animal vs. User has-a UserDetails)
2. 你可以复用基类代码(所有动物的行为，人类都具有)
3. 你可以通过改变基类行为来影响改类所有的衍生类

**糟糕的:**

```php
class Employee 
{
    private $name;
    private $email;

    public function __construct(string $name, string $email)
    {
        $this->name = $name;
        $this->email = $email;
    }

    // ...
}

// `EmployeeTaxData`只是`Employees`的一个特征，符合`have-a`不符合`is-a` 

class EmployeeTaxData extends Employee 
{
    private $ssn;
    private $salary;
    
    public function __construct(string $name, string $email, string $ssn, string $salary)
    {
        parent::__construct($name, $email);

        $this->ssn = $ssn;
        $this->salary = $salary;
    }

    // ...
}
```

**优雅的:**

```php
class EmployeeTaxData 
{
    private $ssn;
    private $salary;

    public function __construct(string $ssn, string $salary)
    {
        $this->ssn = $ssn;
        $this->salary = $salary;
    }

    // ...
}

class Employee 
{
    private $name;
    private $email;
    private $taxData;

    public function __construct(string $name, string $email)
    {
        $this->name = $name;
        $this->email = $email;
    }

    public function setTaxData(string $ssn, string $salary)
    {
        $this->taxData = new EmployeeTaxData($ssn, $salary);
    }

    // ...
}
```

**[⬆  回顶部](#table-of-contents)**

### 避免链式调用

[链式调用](https://en.wikipedia.org/wiki/Fluent_interface)是一种面向对象的api，旨在通过[方法链](https://en.wikipedia.org/wiki/Method_chaining)提高代码的可读性

我们可以利用上下文来构建对象，这种方式可以减少代码的冗余(如[PHPUnit Mock Builder](https://phpunit.de/manual/current/en/test-doubles.html)和[Doctrine Query Builder](http://docs.doctrine-project.org/projects/doctrine-dbal/en/latest/reference/query-builder.html))，经常也会伴随而来一些代价：

1. 破坏[封装](https://en.wikipedia.org/wiki/Encapsulation_%28object-oriented_programming%29)
2. 破坏[设计](https://en.wikipedia.org/wiki/Decorator_pattern)
3. 在测试中很难进行[mock](https://en.wikipedia.org/wiki/Mock_object)
4. 多次提交的代码差异变得难以阅读

阅读[Marco Pivetta](https://github.com/Ocramius)的[blog post](https://ocramius.github.io/blog/fluent-interfaces-are-evil/)获取更多论述

**糟糕的:**

```php
class Car
{
    private $make = 'Honda';
    private $model = 'Accord';
    private $color = 'white';

    public function setMake(string $make): self
    {
        $this->make = $make;

        // NOTE: Returning this for chaining
        return $this;
    }

    public function setModel(string $model): self
    {
        $this->model = $model;

        // NOTE: Returning this for chaining
        return $this;
    }

    public function setColor(string $color): self
    {
        $this->color = $color;

        // NOTE: Returning this for chaining
        return $this;
    }

    public function dump(): void
    {
        var_dump($this->make, $this->model, $this->color);
    }
}

$car = (new Car())
  ->setColor('pink')
  ->setMake('Ford')
  ->setModel('F-150')
  ->dump();
```

**优雅的:**

```php
class Car
{
    private $make = 'Honda';
    private $model = 'Accord';
    private $color = 'white';

    public function setMake(string $make): void
    {
        $this->make = $make;
    }

    public function setModel(string $model): void
    {
        $this->model = $model;
    }

    public function setColor(string $color): void
    {
        $this->color = $color;
    }

    public function dump(): void
    {
        var_dump($this->make, $this->model, $this->color);
    }
}

$car = new Car();
$car->setColor('pink');
$car->setMake('Ford');
$car->setModel('F-150');
$car->dump();
```

**[⬆  回顶部](#table-of-contents)**

## SOLID
**SOLID**是由Michael Feathers首次提出的五个准则，后来被Robert Martin命名的缩写，也是面对象编程与设计的5个基本准则

* [S: 单一职责原则 (SRP)](#S:-职责单一原则-SRP)
* [O: 开闭原则 (OCP)](#O:-开闭原则-OCP)
* [L: 里氏替换原则 (LSP)](#L:-里氏替换原则-LSP)
* [I: 接口隔离原则 (ISP)](#I:-接口隔离原则-ISP)
* [D: 依赖倒置原则 (DIP)](#D:-依赖倒置原则-DIP)

### 单一职责原则 (SRP)

正如Clean Code所描述的，“永远都不要有太多的原因去修改一个类”。我们总是倾向于让单个类集成太多功能，就像飞机 上的行李箱塞满东西。这种情况下的类是松散的，并且很容易需要被修改
尽量减少修改类的次数是很重要的，因为你的每次修改都会影响依赖该类的其他模块

**糟糕的:**

```php
class UserSettings
{
    private $user;

    public function __construct(User $user)
    {
        $this->user = $user;
    }

    public function changeSettings(array $settings): void
    {
        if ($this->verifyCredentials()) {
            // ...
        }
    }

    private function verifyCredentials(): bool
    {
        // ...
    }
}
```

**优雅的:**

```php
class UserAuth 
{
    private $user;

    public function __construct(User $user)
    {
        $this->user = $user;
    }
    
    public function verifyCredentials(): bool
    {
        // ...
    }
}

class UserSettings 
{
    private $user;
    private $auth;

    public function __construct(User $user) 
    {
        $this->user = $user;
        $this->auth = new UserAuth($user);
    }

    public function changeSettings(array $settings): void
    {
        if ($this->auth->verifyCredentials()) {
            // ...
        }
    }
}
```

**[⬆  回顶部](#table-of-contents)**

### 开闭原则 (OCP)

正如Bertrand Meyer所说，“软件实体(classes, modules, functions,etc.)应该可扩展，不可修改”，意思就是你应该允许其他人在不修改已有功能的情况下进行功能扩展

**糟糕的:**

```php
abstract class Adapter
{
    protected $name;

    public function getName(): string
    {
        return $this->name;
    }
}

class AjaxAdapter extends Adapter
{
    public function __construct()
    {
        parent::__construct();

        $this->name = 'ajaxAdapter';
    }
}

class NodeAdapter extends Adapter
{
    public function __construct()
    {
        parent::__construct();

        $this->name = 'nodeAdapter';
    }
}

class HttpRequester
{
    private $adapter;

    public function __construct(Adapter $adapter)
    {
        $this->adapter = $adapter;
    }

    public function fetch(string $url): Promise
    {
        $adapterName = $this->adapter->getName();

        if ($adapterName === 'ajaxAdapter') {
            return $this->makeAjaxCall($url);
        } elseif ($adapterName === 'httpNodeAdapter') {
            return $this->makeHttpCall($url);
        }
    }

    private function makeAjaxCall(string $url): Promise
    {
        // request and return promise
    }

    private function makeHttpCall(string $url): Promise
    {
        // request and return promise
    }
}
```

**优雅的:**

```php
interface Adapter
{
    public function request(string $url): Promise;
}

class AjaxAdapter implements Adapter
{
    public function request(string $url): Promise
    {
        // request and return promise
    }
}

class NodeAdapter implements Adapter
{
    public function request(string $url): Promise
    {
        // request and return promise
    }
}

class HttpRequester
{
    private $adapter;

    public function __construct(Adapter $adapter)
    {
        $this->adapter = $adapter;
    }

    public function fetch(string $url): Promise
    {
        return $this->adapter->request($url);
    }
}
```

**[⬆  回顶部](#table-of-contents)**

### 里氏替换原则 (LSP)

这其实是一个很简单的原则，却起了个令人畏惧的名字。该原则通常被定义为“如果S是T的子类型，那么T类型对象可以被S类型对象替代，并且不会有任何找不到属性的报错”，这个定义也不是很容易理解
再直白点说，就是父类和子类可以替换使用，得到的结果依然是正确的
举个栗子，正方形在学术意义上属于矩形，但是正方形和矩形不能替换使用，如果在你的代码模型中出现正方形继承矩形，构成`is a`关系，恭喜你，你摊上大事了

**糟糕的:**

```php
class Rectangle
{
    protected $width = 0;
    protected $height = 0;

    public function render(int $area): void
    {
        // ...
    }

    public function setWidth(int $width): void
    {
        $this->width = $width;
    }

    public function setHeight(int $height): void
    {
        $this->height = $height;
    }

    public function getArea(): int
    {
        return $this->width * $this->height;
    }
}

class Square extends Rectangle
{
    public function setWidth(int $width): void
    {
        $this->width = $this->height = $width;
    }

    public function setHeight(int $height): void
    {
        $this->width = $this->height = $height;
    }
}

/**
 * @param Rectangle[] $rectangles
 */
function renderLargeRectangles(array $rectangles): void
{
    foreach ($rectangles as $rectangle) {
        $rectangle->setWidth(4);
        $rectangle->setHeight(5);
        $area = $rectangle->getArea(); // BAD: Will return 25 for Square. Should be 20.
        $rectangle->render($area);
    }
}

$rectangles = [new Rectangle(), new Rectangle(), new Square()];
renderLargeRectangles($rectangles);
```

**Good:**

```php
abstract class Shape
{
    protected $width = 0;
    protected $height = 0;

    abstract public function getArea(): int;

    public function render(int $area): void
    {
        // ...
    }
}

class Rectangle extends Shape
{
    public function setWidth(int $width): void
    {
        $this->width = $width;
    }

    public function setHeight(int $height): void
    {
        $this->height = $height;
    }

    public function getArea(): int
    {
        return $this->width * $this->height;
    }
}

class Square extends Shape
{
    private $length = 0;

    public function setLength(int $length): void
    {
        $this->length = $length;
    }

    public function getArea(): int
    {
        return pow($this->length, 2);
    }
}

/**
 * @param Rectangle[] $rectangles
 */
function renderLargeRectangles(array $rectangles): void
{
    foreach ($rectangles as $rectangle) {
        if ($rectangle instanceof Square) {
            $rectangle->setLength(5);
        } elseif ($rectangle instanceof Rectangle) {
            $rectangle->setWidth(4);
            $rectangle->setHeight(5);
        }

        $area = $rectangle->getArea(); 
        $rectangle->render($area);
    }
}

$shapes = [new Rectangle(), new Rectangle(), new Square()];
renderLargeRectangles($shapes);
```

**[⬆ back to top](#table-of-contents)**

### Interface Segregation Principle (ISP)

ISP states that "Clients should not be forced to depend upon interfaces that
they do not use." 

A good example to look at that demonstrates this principle is for
classes that require large settings objects. Not requiring clients to setup
huge amounts of options is beneficial, because most of the time they won't need
all of the settings. Making them optional helps prevent having a "fat interface".

**Bad:**

```php
interface Employee
{
    public function work(): void;

    public function eat(): void;
}

class Human implements Employee
{
    public function work(): void
    {
        // ....working
    }

    public function eat(): void
    {
        // ...... eating in lunch break
    }
}

class Robot implements Employee
{
    public function work(): void
    {
        //.... working much more
    }

    public function eat(): void
    {
        //.... robot can't eat, but it must implement this method
    }
}
```

**Good:**

Not every worker is an employee, but every employee is a worker.

```php
interface Workable
{
    public function work(): void;
}

interface Feedable
{
    public function eat(): void;
}

interface Employee extends Feedable, Workable
{
}

class Human implements Employee
{
    public function work(): void
    {
        // ....working
    }

    public function eat(): void
    {
        //.... eating in lunch break
    }
}

// robot can only work
class Robot implements Workable
{
    public function work(): void
    {
        // ....working
    }
}
```

**[⬆ back to top](#table-of-contents)**

### Dependency Inversion Principle (DIP)

This principle states two essential things:
1. High-level modules should not depend on low-level modules. Both should
depend on abstractions.
2. Abstractions should not depend upon details. Details should depend on
abstractions.

This can be hard to understand at first, but if you've worked with PHP frameworks (like Symfony), you've seen an implementation of this principle in the form of Dependency
Injection (DI). While they are not identical concepts, DIP keeps high-level
modules from knowing the details of its low-level modules and setting them up.
It can accomplish this through DI. A huge benefit of this is that it reduces
the coupling between modules. Coupling is a very bad development pattern because
it makes your code hard to refactor.

**Bad:**

```php
class Employee
{
    public function work(): void
    {
        // ....working
    }
}

class Robot extends Employee
{
    public function work(): void
    {
        //.... working much more
    }
}

class Manager
{
    private $employee;

    public function __construct(Employee $employee)
    {
        $this->employee = $employee;
    }

    public function manage(): void
    {
        $this->employee->work();
    }
}
```

**Good:**

```php
interface Employee
{
    public function work(): void;
}

class Human implements Employee
{
    public function work(): void
    {
        // ....working
    }
}

class Robot implements Employee
{
    public function work(): void
    {
        //.... working much more
    }
}

class Manager
{
    private $employee;

    public function __construct(Employee $employee)
    {
        $this->employee = $employee;
    }

    public function manage(): void
    {
        $this->employee->work();
    }
}
```

**[⬆ back to top](#table-of-contents)**

## Don’t repeat yourself (DRY)

Try to observe the [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) principle.

Do your absolute best to avoid duplicate code. Duplicate code is bad because 
it means that there's more than one place to alter something if you need to 
change some logic.

Imagine if you run a restaurant and you keep track of your inventory: all your 
tomatoes, onions, garlic, spices, etc. If you have multiple lists that
you keep this on, then all have to be updated when you serve a dish with
tomatoes in them. If you only have one list, there's only one place to update!

Oftentimes you have duplicate code because you have two or more slightly
different things, that share a lot in common, but their differences force you
to have two or more separate functions that do much of the same things. Removing 
duplicate code means creating an abstraction that can handle this set of different 
things with just one function/module/class.

Getting the abstraction right is critical, that's why you should follow the
SOLID principles laid out in the [Classes](#classes) section. Bad abstractions can be
worse than duplicate code, so be careful! Having said this, if you can make
a good abstraction, do it! Don't repeat yourself, otherwise you'll find yourself 
updating multiple places anytime you want to change one thing.

**Bad:**

```php
function showDeveloperList(array $developers): void
{
    foreach ($developers as $developer) {
        $expectedSalary = $developer->calculateExpectedSalary();
        $experience = $developer->getExperience();
        $githubLink = $developer->getGithubLink();
        $data = [
            $expectedSalary,
            $experience,
            $githubLink
        ];

        render($data);
    }
}

function showManagerList(array $managers): void
{
    foreach ($managers as $manager) {
        $expectedSalary = $manager->calculateExpectedSalary();
        $experience = $manager->getExperience();
        $githubLink = $manager->getGithubLink();
        $data = [
            $expectedSalary,
            $experience,
            $githubLink
        ];

        render($data);
    }
}
```

**Good:**

```php
function showList(array $employees): void
{
    foreach ($employees as $employee) {
        $expectedSalary = $employee->calculateExpectedSalary();
        $experience = $employee->getExperience();
        $githubLink = $employee->getGithubLink();
        $data = [
            $expectedSalary,
            $experience,
            $githubLink
        ];

        render($data);
    }
}
```

**Very good:**

It is better to use a compact version of the code.

```php
function showList(array $employees): void
{
    foreach ($employees as $employee) {
        render([
            $employee->calculateExpectedSalary(),
            $employee->getExperience(),
            $employee->getGithubLink()
        ]);
    }
}
```

**[⬆ back to top](#table-of-contents)**

## Translations

This is also available in other languages:

* :cn: **Chinese:**
   * [php-cpm/clean-code-php](https://github.com/php-cpm/clean-code-php)
* :ru: **Russian:**
   * [peter-gribanov/clean-code-php](https://github.com/peter-gribanov/clean-code-php)
* :es: **Spanish:**
   * [fikoborquez/clean-code-php](https://github.com/fikoborquez/clean-code-php)
* :brazil: **Portuguese:**
   * [fabioars/clean-code-php](https://github.com/fabioars/clean-code-php)
   * [jeanjar/clean-code-php](https://github.com/jeanjar/clean-code-php/tree/pt-br)
* :thailand: **Thai:**
   * [panuwizzle/clean-code-php](https://github.com/panuwizzle/clean-code-php)
* :fr: **French:**
   * [errorname/clean-code-php](https://github.com/errorname/clean-code-php)

**[⬆ back to top](#table-of-contents)**
