#C#のタプル（Tuple）

この記事はC# ver7.3現在のタプルを紹介します。

##Why Tuple?

C#はオブジェクト指向言語であり、クラスと構造体の豊富な構文が用意されています。 
ところが、分かりやすくて単純なデータ構造、僅かな参照数しかない場合、
わざわざクラスを定義するのが手間がかかります。ソースコードも無駄に長くなって可読性もひくくなります。
このような複数のデータ要素を含む単純な構造が必要な場合は、新しいタプル機能が最適です。

##タプルとは

タプルとは、データ メンバーを表す複数のフィールドを含む軽量なデータ構造です。 
フィールドは検証されず、独自のメソッドを定義することはできません。

##基本的な使い方

タプルを作成するには、各メンバーを値に割り当てます。

    var user = ("userId","userName");
    //或いは
    (string UserId,string UserName) user = ("userId","userName");
    //或いは
    var user = (UserId:"userId",UserName:"userName");

このような、タプルは「データを複数並べたもの」です。
C#のタプルとは、「名前のない複合型」と理解してもいいでしょう。

ジェネリックな型の型引数にも使えます。

var dic = new Dictionary<(string s, string t), (int x, int y)>
{
    { ("a", "b"), (1, 2) },
    { ("x", "y"), (4, 8) },
};
Console.WriteLine(dic[("a", "b")]); // (1, 2)

タプルの作成は、より効率的かつ生産的です。 
タプルは、複数の値を保持するデータ構造を定義するための、よりシンプルで軽量な構文です。 
次の例のメソッドは、整数のシーケンス内で見つかった最小値と最大値を返します。

private static (int Max, int Min) Range(IEnumerable<int> numbers)
{
    int min = int.MaxValue;
    int max = int.MinValue;
    foreach(var n in numbers)
    {
        min = (n < min) ? n : min;
        max = (n > max) ? n : max;
    }
    return (max, min);
}

このようにタプルを使用すると、いくつかの利点があります。
・返される型を定義する class または struct を作成する手間がかからない。
・新しい型を作成する必要がない。
・言語の拡張により、Create<T1>(T1) メソッドを呼び出す必要がなくなる。

##メンバー参照

メンバーの参照の仕方は普通の型と変わりません。
タプルのメンバーは書き換えも可能です。

var name = (given:"Donald",family:"Trump");
Console.WriteLine(name.given);//Donald
Console.WriteLine(name.family);//Trump

//メンバーの書き換え
var range = (max:100,min:1);
range.max = 50;
range.min = -50
Console.WriteLine(range.max);//50
Console.WriteLine(range.min);//-50
//タプル自身を書き換え
range = (10,5);
Console.WriteLine(range.max);//10
Console.WriteLine(range.min);//5

なお、タプルのメンバーはフィールドになっています (プロパティではない)。 
フィールドになっているということは、例えば、参照引数(ref)に直接渡せます (これが、プロパティだと無理)。

##タプルの分解

タプルは、各メンバーを分解して、それぞれ別の変数に受けて使うことができます。

var range = (max:100,min:1);
// 分解宣言
(int num1,int num2) = range;
//或いは
var (num1,num2) = range;
//或いは
int num1,num2;
(num1,num2) = range;

また、ユーザー定義型 (クラス、構造体、またはインターフェイス) も簡単に分解できます。
型の作成者は、型を構成するデータ要素を表す任意の数の out 変数に対して値を割り当てる Deconstruct メソッドを 1 つ以上定義できます。 
たとえば、次の Person 型は、person オブジェクトを、名と姓を表す要素に分解する Deconstruct メソッドを定義しています。

public class Person
{
    public string FirstName { get; }
    public string LastName { get; }
    public int Age{ get; }

    public Person(string first, string last, int age)
    {
        FirstName = first;
        LastName = last;
        Age = age;
    }

    public void Deconstruct(out string firstName, out string lastName, out int age)
    {
        firstName = FirstName;
        lastName = LastName;
        age = Age;
    }
}

deconstruct メソッドを使用すると、Person から、FirstName プロパティと LastName プロパティを表す 2 つの文字列を割り当てることができます。

var p = new Person("Donald", "Trump", (int)(DateTime.Now - New DateTime(1946,6,14)).TotalYears);
var (first, last, age) = p;

##破棄

破棄は、アプリケーション コードで意図的に使用しない一時的なダミー変数です。 
破棄は、未割り当ての変数と同等です。つまり、値がありません。 
破棄変数は 1 つのみであり、破棄変数には記憶域も割り当てられないため、破棄を使用するとメモリの割り当てを減らすことができます。 
また、コードの意図がわかりやすくなるため、読みやすさと保守性が向上します。
変数を破棄と指定するには、変数名にアンダースコア (_) を指定します。

var p = new Person("Donald", "Trump", (int)(DateTime.Now - New DateTime(1946,6,14)).TotalYears);
var (_, last, _) = p;//名前だけを利用したい
Console.WriteLine(last);

## ==、!= での比較

タプルの「==、!=」演算子は、コンパイラーによる特別な処理が入ります。

タプルに対する==比較は、以下のように、メンバーごとの==を&&で繋いだものに展開されます。

int(int a, (int x, int y) b) t = (1, (2, 3))
Console.WriteLine(t == (1, (2, 3)));
//実際にコンパイル時に、下記の様に展開される
Console.WriteLine(t.a == 1 && t.b.x == 2 && t.b.y == 3);

!=演算子も同じように||で繋いだものになります。

##結論

メソッドは複数返す値が必要な場合、
従来だとoutを使うか、予めクラスを定義しなければなりませんが、
タプルを利用して簡単に実現できます。
手間が減らす一方に、より分かりやすくなります。
ただし、複雑なデータ構造の場合、タプルを濫用してしまうと、
逆に分かりにくくなります。十分気を付ける上に活用しましょう。
