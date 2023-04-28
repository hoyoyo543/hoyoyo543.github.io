---
title: DDDにおけるEntityについて
date: 2020-03-03 23:30:08
tags:
---

IDDD本を読んだりしてEntityのイメージが固まってきたので書き残します。  
sample codeのコメントを詳細に説明していきます。
## sample code

```
<?php

// エンティティ
class User {

    private $id;
    private $name;
    private $age;
    private $gender;
    private $email;
    private $address;

    // すべてのプロパティの値をコンストラクタで設定します。
    private function __construct($id, $name, $age, ?Gender $gender, $email, $address)
    {
        // セッターを利用してプロパティに値を格納します。
        $this->setId($id);
        $this->setName($name);
        $this->editAgeGender($age, $gender);
        $this->setEmail($email);
        $this->setAddress($address);
    }

    // セッターでバリデートします。
    public function setId($id)
    {
        if ($this->id) {
            throw new DomainException('idの変更はできません。');
        }
        $this->id = $id;
    }

    public function setName($name)
    {
        $this->name = $name;
    }

    public function setEmail($email)
    {
        $this->email = $email;
    }

    public function setAddress($address)
    {
        $this->address = $address;
    }

    public function getGender()
    {
        return $this->gender;
    }

    // プロフィール編集の時に名前と年齢を変更できるという振る舞いです。
    public function editProfile($name, $age)
    {
        $this->setName($name);
        $this->editAgeGender($age, $this->getGender());
    }

    // 自身を作成するファクトリーメソッドを持ちます。
    // 会員登録時のユーザーを作成するという振る舞いです。
    public static function memberEntry ($id, $name, $email)
    {
        return new self($id, $name, null, null, $email, null);
    }

    // エンティティ全体の状態のバリデートを行う、これを呼び出すのはドメインサービス
    public validate(ValidationNotificationHandler $handler) 
    {
        (new UserValidator($this, $handler)).validate();
    }
}

// Validatorは再利用可能な抽象バリデーター
// エンティティ個別のバリデーターにエンティティ全体のバリデートロジックをもたせます。
class UserValidator extends Validator{

    private $user;

    public function __construct(User $user, ValidationNotificationHandler $handler)
    {
        parent::__construct($handler);
        $this->user = $user;
    }

    public function valiate()
    {
        if ($this->hasWarpedAgeGender()) {
            $this->notificationHandler()->handleError('年齢と性別の関係が不正です。');
        }
    }

    // 年齢と性別の関係のバリデートロジック
    public function hasWarpedAgeGender()
    {
        if ($this->age >= 30 && $this->age < 40 && $gender === Gender::MAN) {
            return true;
        }
        if ($this->age >= 20 && $this->age < 30 && $gender === Gender::WOMAN) {
            return true;
        }

        return false;
    }
}
```

## 原則
Entityは識別子を持って一意であり永続化されます。

## Entityの振る舞いとは
多くは下記の二点のことを指します。  
・自身の状態を変更するメソッド  
・自身を作成するファクトリメソッド  
※デザインパターンのファクトリメソッドのことではありません。

例えば `sample code` での振る舞いは、  
・プロフィール編集の時に名前と年齢を変更できる。 (自身の状態を変更するメソッド )  
・会員登録できる。 (自身を作成するファクトリメソッド)  
という振る舞いをエンティティが持つことになります。

## すべてのプロパティの値をコンストラクタで設定します
不完全な状態のインスタンスを作成しないようにコンストラクタですべてのプロパティを設定できるべきです。  
下記のアンチパターンのようにコンストラクタで状態が完結せずにセッターでセットしてインスタンスの状態が完成するべきではありません。
```
// アンチパターン
$user = new User($id, $name);
$user->setAge(30);
```
もちろんインスタンスの状態が完成している物に対して、変更する目的でセッターを利用するのは問題ありません。

## セッターでバリデーションを行い、プロパティに値を格納します。
セッターでバリデーションを行うことにより。正当な値がプロパティに格納されます。
コンストラクタでバリデーションは行いません、ミュータブルなエンティティにとって状態を変更する時にコンストラクタのバリデーションを利用できないからです。

## 複数の項目に渡るバリデーションは、オブジェクト全体を`遅延バリデート`する
セッターを作成し、相互にバリデーションを行うことはできません。  
また、制約に関連する項目すべてを引数にとり状態を変化させるメソッドを作成すると、制約の追加や変化に応じてメソッドの引数がどんどん多くなっていき、利用しずらい振る舞いになります。  
制約に関連する項目の変更がすべて揃った状態か、それとも途中の中途半端な状態かというのはエンティティに判断させることは難しくなりますので、ドメインサービスにてバリデートの実行のタイミングを判断します。  
エンティティにはどのバリデーターを利用するかをだけを決めさせて、エンティティ全体のバリデートロジックはバリデーターに担当させることで、エンティティの振る舞いの責務が埋もれてしまわないようにします。  
（※矛盾するように感じるかもしれませんが、プロパティ個別のバリデートはエンティティのセッターで行います。）  
```
// アンチパターン １

// 各項目ごとのセッターで複数の項目にわたるバリデートを行う。
// この例だと、女性で25歳の状態を男性で33歳に変更したい場合に
// 変更できなくなる
public function setAge($age)
{
    if ($age >= 30 && $age < 40 && $this->gender === Gender::MAN) {
        throw new DomainException('30代の男性しか登録できません');
    }
    if ($age >= 20 && $age < 30 && $this->gender === Gender::WOMAN) {
        throw new DomainException('20代の女性しか登録できません');
    }

    $this->age = $age;
}

public function setGender($gender)
{
    if ($this->age >= 30 && $this->age < 40 && $gender === Gender::MAN) {
        throw new DomainException('30代の男性しか登録できません');
    }
    if ($this->age >= 20 && $this->age < 30 && $gender === Gender::WOMAN) {
        throw new DomainException('20代の女性しか登録できません');
    }

    $this->gender = $gender;
}

// アンチパターン 2
// 複数の項目に渡るバリデーションはその項目数を引数にもったメソッドでプロパティにセットする。
// その後、男性は港区だけという制約が増えると振る舞いの引数が増えどんどん扱いにくいメソッドになっていきます。
public function editAgeGender($age, ?Gender $gender)
{
    if ($this->age < 30 && $this->gender === Gender::MAN) {
        throw new DomainException('30歳未満の男性は登録できません');
    }

    if ($this->age < 25 && $this->gender === Gender::WOMAN) {
        throw new DomainException('25歳未満の女性は登録できません');
    }

    $this->age = $age;
    $this->gender = $gender;
}
```


