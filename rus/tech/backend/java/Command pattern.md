среда, 6 мая 2009 г. в 18:32:45

Я предпочитаю переводить pattern как "тропинка". Тропинки в программировании это общие закономерности которые прокладывают разные программисты для решения похожих задач.

Command pattern оказывается нужным когда понятие "комманды" становится более чем просто вызовом функции. Например когда надо делать undo, транзакции внутри программы, правильную полоску установки (installer progress bar), когда надо делать многостраничный опросник (wizard), когда хочется записать то что делает пользователь или наоборот выполнить пользовательские команды (macro).

В коде такая тропинка отличается тем что комманда становится классом. А точней делается интерфейс Command, и уже от него создаются конкретные комманды. Конкретные комманды могут не просто делать что-то по себе, но и иметь внедрённый (incapsulated) связанный объект. Я конечно в качестве примера беру игру WoW где охотник может поставить ловушку, которая сработает когда на неё наткнётся противник.

---

Конечно Wow не на Java написан, но для идеи отличающейся от [переключателя и ламочки](http://en.wikipedia.org/wiki/Command_pattern) сойдёт:

```java
interface Command {  
void execute();  
}  
  
class IceTrapAttack implements Command {  
  private Target target;  
  private Trap trap;   public IceTrapAttack(Target target, Trap trap){  
    this.target=target;  
    this.trap=trap;  
  }  
  public void execute() {  
    trap.disappear();  
    target.receiveAttack();  
    target.blockFreeze();  
  }  
}
```

Теперь по идее устаналивается ловушка, которая будет атаковать цель незная по сути кто этой целью будет и не беспокоясь о **существовании правильных методов у цели**. В терминологии же получается что ловушка это Envoker, а цель это Receiver.

Теперь если говорить о макро-коммандах. В WoW пользователь может примитивными коммандами задать последовательность действий. Скажем макрокомманда отступления у охотника выглядела бы как установка ловушки, отпрыгивание, стрела замедления и аспект гепарда. Внутри Macro выглядело бы так:

```java
class Macro implements Command{  
  private List commands = new ArrayList();  
  
  public void add(Command c) {  
    commands.add(c);  
  }  
  
  public void run() {  
    Iterator it = commands.iterator();  
    while (it.hasNext())  
      ((Command) it.next()).execute();  
  }  
}
```

А вызывать можно просто

```java
    Macro macro = new Macro();  
    macro.add(new SetTrapCommand());  
    macro.add(new DisengageCommand());  
macro.add(new ConcussiveShotCommand());  
    macro.add(new AspectCheetahCommand());  
    macro.run();
```