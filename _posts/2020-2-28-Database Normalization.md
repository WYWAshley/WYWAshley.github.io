首先，我们要明白一点，六大范式之间的关系：

<img src="/images/posts/Database Normalization/1.jpg"/>

也就是说 $5NF\in4NF\in BCNF\in3NF\in2NF\in1NF$

<img src="/images/posts/Database Normalization/0.png"/>

其次，范式的作用：

* 规范化目的是使结构更合理，消除存储异常，使数据冗余尽量小，便于插入、删除和更新。
* 范式越高意味着表的划分更细，一个数据库中需要的表也就越多，用户不得不将原本相关联的数据分摊到多个表中。当用户同时需要这些数据时只能采用连接表的形式将数据重新合并在一起，这个花费是巨大的，尤其是当需要连接的两张或者多张表数据非常庞大的时候，这严重地降低了系统运行性能。

## 一、1NF

**所有属性都不可再分**，也就是每一列都是不可再分的原子数据项，而不是集合、数组、记录等

<img src="/images/posts/Database Normalization/2.png"/>

<img src="/images/posts/Database Normalization/3.png"/>

<br>

## 二、2NF

加条件：每一个非主属性完全函数依赖于任何一个候选码。也就是说要求表有数据项作为记录的唯一标识符，其他的数据项要**完全函数依赖**于主键，如果是部分函数依赖，那么主键需要被拆分成多列，其中一列作为新主键，其他的和主键一一对应。

**候选码：** 若关系中的某一属性组的值能唯一地标识一个元组，而其子集不能，则称该属性组为候选码。若一个关系中有多个候选码，则选定其中一个为主码。

**主属性：** 所有候选码的**加和**属性称为主属性。不包含在任何候选码中的属性称为非主属性或非码属性。

**函数依赖：** 设R(U)是属性集U上的关系模式，X、Y是U的子集。若对于R(U)的任意一个可能的关系r，r中不可能存在两个元组在X上的属性值相等，而在Y上的属性值不等，则称Y函数依赖于X或X函数确定Y。

**完全函数依赖：** 设R(U)是属性集U上的关系模式，X、Y是U的子集。如果Y函数依赖于X，且对于X的任何一个真子集X’，都有Y不函数依赖于X’，则称Y对X完全函数依赖。记作：如果Y函数依赖于X，但Y不完全函数依赖于X，则称Y对X部分函数依赖。
<img src="/images/posts/Database Normalization/4.jpg"/>

<img src="/images/posts/Database Normalization/5.png"/>

其中 Sno, Sdept, Sloc, Cno, Grade 依次表示学生的学号、所在的系、住处、课程号、班级，并且每个系的学生住在同一个地方。可知 S-L-C 的码为（Sno, Cno），但是非主属性 Sloc、Sdep t并不完全函数依赖于码，因此关系模式 S-L-C(Sno, Sdept, Sloc, Cno, Grade) 不符合第二范式。

<br>

## 三、3NF

增加条件：非主属性既**不传递依赖于码**，也不部分依赖于码。也就是说，**属性不依赖于其它非主属性**。

<img src="/images/posts/Database Normalization/6.jpg"/>

<br>

## 四、BCNF

Boyce-Codd Normal Form（巴斯-科德范式）

通常情况下，巴斯-科德范式被认为没有新的设计规范加入，只是对第二范式与第三范式中设计规范要求更强，因而被认为是修正第三范式，也就是说，它事实上是对第三范式的修正，使数据库冗余度更小。

增加条件：要求每一个决定因素都包含码，也就是**消除主属性之间的部分函数依赖和传递函数依赖**.

比方说 AB->C,BC->A，此时 AB、BC 都是码即 ABC 都是主属性。所以ABC之间有什么函数依赖不在 1-3NF 的约束之内。所以需要有 BCNF 来约束主属性之间的函数依赖了。

 举个例子，R={AB->C,BC->A,C->A}，R 的码为 AB、BC，故主属性为 ABC，R 为 3NF。但是函数依赖 C->A，决定因素 C 不包含码 BC 故 R 不是 BCNF。

<br>

## 五、4NF

增加条件：属性之间不允许有非平凡且非函数依赖的多值依赖。非函数多值依赖就是U=X+Y+Z，允许 X 的一个值决定 Y 的一组值，这种决定关系与 Z 取值无关。非平凡的就是 Y 不属于 X 的子集。

例如，有这样一个关系 <仓库管理员，仓库号，库存产品号> ，假设一个产品只能放到一个仓库中，但是一个仓库可以有若干管理员，那么对应于一个仓库管理员有一个仓库号，而实际上，这个仓库号只与库存产品号有关，与管理员无关，就说这是多值依赖。为降低数据冗余，做模式分解，使子模式的 Z=Ø，仅有平凡多值依赖。对前例，子模式为 R1(仓库,雇员),R2(仓库,物资)

<br>

## 六、5NF

增加条件：把表尽可能的拆分成多表。

<br>