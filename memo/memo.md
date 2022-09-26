C#で多態性（virtual, override）

　これがやりたかった。ようやく成功した。

　インタフェースや親クラス型として異なる子クラスのインスタンスを受け取り、同名の異なるメソッドを呼び出す。

<!-- more -->

# ブツ

* [][]

[]:

# コード

```csharp
interface IAccelerator {
    void Accelerator(int gear = -1);
}
```

```csharp
class Car : IAccelerator {
    private int gear;
    protected int Gear {
        get { return this.gear; }
        set { if (0<=value && value<=6) { this.gear = value; } }
    }
    public virtual void Accelerator(int gear = -1) {
        Console.WriteLine("{0}.{1}() {2}", GetType().Name, System.Reflection.MethodBase.GetCurrentMethod().Name, this.Gear);
    }
}
```

```csharp
class ATCar : Car {
    public override void Accelerator(int gear = -1) {
        if (6 == this.Gear) { this.Gear = 0; }
        this.Gear++;
        base.Accelerator();
    }
}
```

```csharp
class MTCar : Car {
    public override void Accelerator(int gear = -1) {
        if (1 == Math.Abs(this.Gear - gear)) {
            Console.WriteLine("{0}.{1}({2}) {3}", 
                GetType().Name, 
                System.Reflection.MethodBase.GetCurrentMethod().Name, 
                gear, this.Gear);
            this.Gear = gear;
        }
        else {
            Console.WriteLine("{0}.{1}({2}) {3} {4}", 
                GetType().Name, 
                System.Reflection.MethodBase.GetCurrentMethod().Name, 
                gear, this.Gear, "エンスト！");
            this.Gear = 0;
        }
    }
}
```

```csharp
Car car = new Car();
ATCar at = new ATCar();
MTCar mt = new MTCar();

car.Accelerator();
car.Accelerator();

Console.WriteLine("---------- AT ----------");

at.Accelerator();
at.Accelerator();
at.Accelerator();
at.Accelerator();
at.Accelerator();
at.Accelerator();
at.Accelerator();

Console.WriteLine("---------- MT ----------");
mt.Accelerator();
mt.Accelerator();
Console.WriteLine("---------- MT 引数あり ----------");
mt.Accelerator(1);
mt.Accelerator(2);
mt.Accelerator(3);
mt.Accelerator(2);
mt.Accelerator(4);
mt.Accelerator(3);
mt.Accelerator(1);

Console.WriteLine("========== Car ==========");

Car at1 = new ATCar();
Car mt1 = new MTCar();
at1.Accelerator(); // Carのものが呼び出される
at1.Accelerator(); // Carのものが呼び出される
mt1.Accelerator(1); // Carのものが呼び出される
mt1.Accelerator(2); // Carのものが呼び出される

Console.WriteLine("========== IAccelerator ==========");

IAccelerator at2 = new ATCar();
IAccelerator mt2 = new MTCar();
at2.Accelerator(); // Carのものが呼び出される
at2.Accelerator(); // Carのものが呼び出される
mt2.Accelerator(1); // Carのものが呼び出される
mt2.Accelerator(2); // Carのものが呼び出される

Console.WriteLine("========== var ==========");

var at3 = new ATCar();
var mt3 = new MTCar();
at3.Accelerator();
at3.Accelerator();
mt3.Accelerator(1); // error CS1501: 引数 1 を指定するメソッド 'Accelerator' のオーバーロードはありません
mt3.Accelerator(2); // error CS1501: 引数 1 を指定するメソッド 'Accelerator' のオーバーロードはありません

Console.WriteLine("========== Factory ==========");

var factory = new CarFactory();
var someCar = factory.create();
someCar.Accelerator();
someCar.Accelerator(1);
someCar.Accelerator(3);
someCar = factory.create(true);
someCar.Accelerator();
someCar.Accelerator(1);
someCar.Accelerator(3);
```
```sh
Car.Accelerator() 0
Car.Accelerator() 0
---------- AT ----------
ATCar.Accelerator() 1
ATCar.Accelerator() 2
ATCar.Accelerator() 3
ATCar.Accelerator() 4
ATCar.Accelerator() 5
ATCar.Accelerator() 6
ATCar.Accelerator() 1
---------- MT ----------
MTCar.Accelerator(-1) 0
MTCar.Accelerator(-1) 0
---------- MT 引数あり ----------
MTCar.Accelerator(1) 0
MTCar.Accelerator(2) 1
MTCar.Accelerator(3) 2
MTCar.Accelerator(2) 3
MTCar.Accelerator(4) 2 エンスト！
MTCar.Accelerator(3) 0 エンスト！
MTCar.Accelerator(1) 0
========== Car ==========
ATCar.Accelerator() 1
ATCar.Accelerator() 2
MTCar.Accelerator(1) 0
MTCar.Accelerator(2) 1
========== IAccelerator ==========
ATCar.Accelerator() 1
ATCar.Accelerator() 2
MTCar.Accelerator(1) 0
MTCar.Accelerator(2) 1
========== var ==========
ATCar.Accelerator() 1
ATCar.Accelerator() 2
MTCar.Accelerator(1) 0
MTCar.Accelerator(2) 1
========== Factory ==========
ATCar.Accelerator() 1
ATCar.Accelerator() 2
ATCar.Accelerator() 3
MTCar.Accelerator(-1) 0
MTCar.Accelerator(1) 0
MTCar.Accelerator(3) 1 エンスト！
```

# 情報源

* [多態性][]
* [overrideとnewの違い][]

[多態性]:https://ufcpp.net/study/csharp/oo_polymorphism.html
[overrideとnewの違い]:https://qiita.com/matari/items/df13d71b70a65decadef

## overrideとnew

　`override`は同名メソッドを上書きする。`new`は同名メソッドを新たに生成する。

　`override`だと子メソッドが呼ばれる。`new`だと呼び出したインスタンスの型による。親クラスやインタフェースでインスタンスを受け取ったときは親メソッドが呼ばれる。インスタンスを子クラスにキャストすれば子メソッドが呼ばれる。

## 多態性

　多態性をやるときは抽象型でインスタンスを受け取り、それぞれの子メソッドを呼んでほしい。なので`override`でないと多態性を実現できない。`new`だと抽象型のメソッドが呼ばれてしまう。

　そういう認識で合っていると思う。

