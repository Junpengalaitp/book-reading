### 第一章 重构 第一个案例
* 重构第一步，为即将修改的代码建立一组可靠的测试环境
* 把长函数切开，并把小块的代码移至合适的类
* 任何一个傻瓜都能写出计算机可以理解的代码。唯有写出人类容易理解的代码，才是优秀的程序员

### 第二章 重构原则
* 何为重构
  * 名词：对软件内部结构的一种调整，目的是不改变软件可观察行为的前提下，提高其可理解性，降低其修改成本。
  * 动词：使用一系列重构手法，在不改变软件可观察行为的前提下，调整其结构。
* 为何重构
  * 改进软件设计
  * 使软件更容易理解
  * 帮助找到BUG
  * 提高编程速度
* 何时重构
  * 三次法则
  * 添加功能时重构
  * 修补错误时重构
  * 复审代码时重构

### 第三章 代码的坏味道
  * 重复代码
  * 过长函数
  * 过大的类
  * 过长参数列
  * 发散式变化
    * 经常因为不同原因在不同方向上发生变化
  * Shotgun Surgery(霰弹式修改)
    * 每次遇到变化，都必须在许多不同类做出许多小修改
  * Feature Envy(依恋情结)
  * Data Clumps(数据泥团)
  * Primitive Obsession(基本类型偏执)
  * Switch Statements
  * Parallel Inheritance Hierarchies(平行继承体系)
    * 当为一个类添加子类时，必须也为另一个类相应增加子类
  * Lazy Class(冗赘类)
  * Speculative Generality(夸夸其谈未来性)
  * Temporary Field(令人迷惑的暂时字段)
  * Message Chains(过度耦合的消息链)
  * Middle Man(中间人)
  * Inappropriate Intimacy
  * Alternative Class with Different Interfaces(异曲同工的类)
  * Incomplete Library Class(不完美的库类)
  * Data Class
  * Refused Bequest
    * 子类应该继承超类的函数和数据，当他们不需要继承，需要为这个子类新建一个兄弟类
  * 过多的注释

### 第四章 构筑测试体系
* 频繁地运行测试。每次编译请把测试类也考虑进去，每天至少执行每个测试一次。
* 编写未臻完善的测试并实际运行，好过对完美测试的无尽等待。
* 考虑可能出错的边界条件，把测试火力集中。
* 当事情被认为应该会出错时，别忘了检查是否抛出了预期的异常。
* 不要因为测试无法捕捉所有bug就不写测试，因为测试的确可以捕捉到大多数Bug.\

### 第五章 重构列表
* 重构的记录格式
  * 名称(name)
  * 概要(summary)
  * 动机(motivation)
  * 做法(mechanics)
  * 范例(examples)
* 寻找引用点
* 这些重构手法有多成熟

### 第六章 重新组织函数
* Extract Method(提炼函数)
  * 过长的函数，或者需要注释才能弄懂的代码
  * 创造一个新函数，根据这个函数的意图来对它命名（依据做什么，而不是怎么做）
  * 将提炼出的代码从原函数复制到新建的目标函数中
  * 仔细检查提炼出的代码，看看其中是否有任何局部变量的值被它改变。如果一个临时变量值被修改了，看看是否可以将被提炼代码段处理为一个查询，并将结果赋值给相关变量。
  * 将被提炼代码段中需要读取的局部变量，当作参数传给目标函数。
  * 处理完所有局部变量后，进行编译
  * 在原函数中，将被提炼的代码段替换为被目标函数的调用
  * 编译，测试。
* Inline Method(内联函数)
  * 将这个函数的所有被调用点都替换为函数本体
* Inline Temp(内联临时变量)
* Replace Temp with Query(以查询取代临时变量)
* Introduce Explaining Variable(引入解释性变量)
* Split Temporary Variable(分解临时变量)
* Remove Assignments to Parameters(移除对参数的赋值)
* Replace Method with Method Object(以函数对象取代函数)
* Substitute Algorithm(替换算法)

### 第7章 在对象之间搬移特性
* Move Method(搬移函数)
  * 如果一个类有太多行为，或者一个类与另一个类有太多合作而形成太多耦合
* Move Field(搬移字段)
  * 对于一个字段，在其所在类之外的另一个类中有更多函数使用了它。
* Extract Class(提炼类)
  * 含有大量函数和数据，不易理解
* Inline Class(将类内联化)
  * 一个类不再有单独存在的理由
* Hide Delegate(隐藏“委托关系”)
  * 封装意味着每个对象都尽可能少了解系统的其他部分
* Remove Middle Man(移除中间人)
* Introduce Foreign Method(引入外加函数)
* Introduce Local Extension(引入本地扩展)
  * 建立一个新类，使它包含这些额外函数。让这个扩展瓶成为源类的子类或者包装类。

### 第8章 重新组织数据
* Self Encapsulate Field(自封装字段)
  * 当你想访问超类的一个字段，却又想在子类中将对这个变量的访问改为一个计算后的值。
* Replace Data Value with Object(以对象取代数据)
  * 一个数据项，需要与其他数据和行为一起使用才有意义
* Change Value to Reference(将值对象改为引用对象)
  * 从一个类衍生出彼此相等的实例，希望将它们替换为同一个对象。
* Change Reference to Value
  * 有一个引用对象，很小且不可变，而且不易管理
* Replace Array with Object(以对象取代数组)
* Duplicate Observed Data(复制“被监视数据”)
  * 你有一些领域数据置身于GUI控件中，而领域函数需要访问这些数据。
* Change Unidirectional Association to Bidirectional(将单向关联改为双向关联)
* Change Bidirectional Association to Unidirectional
* Replace Magic Number with Symbolic Constant
* Encapsulate Field
* Encapsulate Collection
  * 让函数返回集合的一个只读副本
* Replace Record with Data Class(以数据类取代记录)
* Replace Type Code with Class(以类取代类型码)
* Replace Type Code with Subclass(以子类取代类型码)
* Replace Type Code with State/Strategy(以State/Strategy取代类型码)
* Replace Subclasses with Fields(以字段取代子类)

# 第9章 简化条件表达式
* Decompose Conditional(分解条件表达式)
  * 从if, else, then 提炼出独立函数
* Consolidate Conditional Expression(合并条件表达式)
  * 将if测试串合并为一个条件表达式
* Consolidate Duplicate Conditional Fragments(合并重复的条件片段)
  * 在条件表达式的每个分支上有着相同的一段代码
  * 将这段重复代码搬移到条件表达式外
* Remove Control Flag(移除控制标记)
  * 以break或return语句取代控制标记
* Replace Nested Conditional with Guard Clauses(以卫语句取代嵌套条件表达式)
* Replace Conditional with Polymorphism(以多态取代条件表达式)
* Introduce Null Object(引入Null对象)
* Introduce Assertion(引入断言)

# 第10章 简化函数调用
* Rename Method
* Add Parameter
* Remove Parameter
* Separate Query from Modifier(将查询函数和修改函数分离)
* Parameterize Method(令函数携带参数)
* Replace Parameter with Explicit Methods(以明确函数取代参数)
* Preserve Whole Object(保存对象完整)
* Replace Parameter with Methods(以函数取代参数)
* Introduce Parameter Object(引入参数对象)
* Remove Setting Method(移除设值函数)
* Hide Method(隐藏函数)
* Replace Constructor with Factory Method
* Encapsulate Downcast(封装向下转型)
* Replace Error Code with Exception(以异常取代错误码)
* Replace Exception with Test(以测试取代异常)

### 第11章 处理概括关系
* Pull Up Field(字段上移)
* Pull Up Method(函数上移)
* Pull up Constructor Body(构造函数本体上移)
* Push Down Method(函数下移)
* Push Down Field(字段下移)
* Extract Subclass(提炼子类)
* Extract Superclass(提炼超类)
* Extract Interface(提炼接口)
* Collapse Hierarchy(折叠继承关系)
* Form Template Method(塑造模版函数)
  * 一些子类中相应的某些函数以相同顺序执行类似操作，但各个操作的细节上有所不同
  * 将这些操作分别放入独立函数中，并保持它们都有相同的签名，于是原函数也就变得相同了。然后将原函数上移至超类
* Replace Inheritance with Delegation
  * 某个子类只使用超类接口的一部分，或者是根本不需要继承来的数据
  * 在子类中新建一个field保存超类，调整子类使其委托超类，然后去掉两者间的继承关系。
* Replace Delegation with Inheritance(以继承取代委托)

### 第12章 大型重构
* Tease Apart Inheritance(梳理并分解继承体系)
  * 某个继承体系同时承担两项责任
  * 建立两个继承体系，并通过委托关系让其中一个可以调用另一个
* Convert Procedural Design to Objects(将过程化设计转化为对象设计)
* Separate Domain from Presentation(将领域和表述/显示分离)
* Extract Hierarchy(提炼继承体系)
  * 某个类做了太多工作，其中一部分工作是以大量条件表达式完成的
  * 建立继承体系，以一个子类表示一种特殊情况