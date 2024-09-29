

# C++

## 初识C++

**“”与‘’区别：**

“”代表字符串，‘’代表字符，每个字符串都有一个'\0'结束符



**注释**：        先CTRL + K，然后CTRL + C
**取消注释**： 先CTRL + K，然后CTRL + U



**std::cout**是一个类，在C++能用，C不能用
**printf**是一个函数，在C++能用,C也能用



**起名规则**

1. 不能重名
2. 不能和C/C++语言关键字重名
3. 必须是字母或字母和数字的组合，符号仅——可以使用
4. 名字不能用数字开头



## 数据类型

### **初始化**

1. **类型 名字；**

   - 在函数内定义时，不会初始化
   - 在函数外定义，也就是全局变量时，会初始化为0

2. **类型 名字{初始值}；**

   - 若是没有初始值，默认为0
   - bool类型默认为false
   - 当类型收窄有数据丢失风险时，编译器会报错，例如大转小，浮点数转整数

   ```C++
   char a = 1;
   int b{a};//成功，因为没有风险
   
   int c = 1;
   short d{ c };//报错，因为大转小
   
   float a{ 1.0 };
   int b{ a };//报错，因为浮点数转整数
   ```

3. **类型 名字(初始值)；**

   - 若是没有初始值，默认为0
   - bool类型默认为false

4. **类型 名字=初始值；**

   - 未初始化无法打印



**判断类型**

```C++
cout << typeid(i).name() <<endl;
```



### **带符号的整数类型表**

**注意：**signed表示有符号，包括0

| 类型                                                         | 内存占用（字节） | 取值范围                                                     |
| ------------------------------------------------------------ | ---------------- | ------------------------------------------------------------ |
| signed char<br />char                                        | 1                | -128到127<br />char取值范围在各个系统与编译器有可能不相同，需要根据实际情况 |
| short<br />short int<br />unsigned short<br />unsigned short int | 2                | -32768到32767                                                |
| int<br />unsigned<br />unsigned int                          | 4                | -2'147'483'648到2'147'483'647                                |
| long<br />long int<br />unsigned long<br />unsigned long int | 4(32位)/8(64位)  | **4字节** -2'147'483'648到2'147'483'647<br />**8字节** -9'223'372'036'854'775'808到9'223'372'036'854'775'807 |
| long long<br />long long int<br />unsigned long long<br />unsigned long long int | 8                | -9'223'372'036'854'775'808到9'223'372'036'854'775'807        |



### **整形字面量的表达**

| 进制     | 表达方式           |
| -------- | ------------------ |
| 二进制   | 0b+二进制+后缀     |
| 八进制   | 0+八进制数+后缀    |
| 十进制   | 十进制数+后缀      |
| 十六进制 | 0x+十六进制数+后缀 |

**后缀说明：**

1. L/l代表该值为long型
2. LL/ll代表该值为long long 型
3. U/u代表该值为无符号型，可以和L/ll或者LL/ll组合使用，比如65535ULL



### **浮点数数据类型**

| 类型        | 内存占用（字节） | 取值范围              |
| ----------- | ---------------- | --------------------- |
| float       | 4                | 大约7位精度±3.4E±38   |
| double      | 8                | 大约15位精度±1.7E±308 |
| long double | 8                | 大约15位精度±1.7E±308 |



### **浮点数表达**

1. 单精度浮点数（float）小数后加f或F，例如200.0f/200.0F
2. **不带后缀的浮点数一律视为double类型**，例如200.0
3. 后缀为L/l的浮点数视为long double类型
4. 浮点数不进可以表达小数，也可以表达整数，比如1E2=100.0



### **类型转换顺序表**


|   类型转换顺序表   |
| :----------------: |
|    long double     |
|       double       |
|       float        |
| unsigned long long |
|     long long      |
|   unsigned long    |
|        long        |
|    unsigned int    |
|        int         |

**short+char=int**

**变量生命周期在一个括号内**

**有符号和无符号相加减得到的结果是无符号**



### **布尔**

```C++
bool bcase{ intIn % 2 == 0 };//判断奇数偶数，奇数为false，偶数为true
```



## 格式化输出

|   类型   | 内存占用 |    说明    |
| :------: | :------: | :--------: |
|   char   |    1     | ascii字符  |
| wchar_t  |    2     | 宽字节字符 |
| char16_t |    3     | utf_16字符 |
| char32_t |    4     | utf_32字符 |

**auto能自动选择类型**



**字符串初始化**

```C++
char str[0xFF]{ 0x48,'e','l','l','o','\0'};//\0=数字0,C语言中的字符串以0结尾
char str1[0xFF]{ "Hello阿斯蒂芬" };
char* str2 = (char*)"Hello阿斯蒂芬";//本来"Hello"的类型是const char[]，需要强转为char*
const char* str3 = "Hello";
char* str4 = new char[0xFF] {"Hello"};	


#include<locale>
setlocale(LC_ALL, "chs");//更改本地地区，不然下面的中文打印不出来

wchar_t wstr[0xFF]{ L"z站" };//加L代表宽字节
wcout << wstr << endl;//cout没有实现对宽字节的支持，需要使用wcout

char16_t char16{ u'a' };//utf-16，u代表采用utf-16标准
char32_t char32{ U'a' };//utf-32，U代表采用utf-32标准
```



**输出处理**

**字符串处理函数（cctype头文件）**

|       函数        |            说明            |
| :---------------: | :------------------------: |
| int isupper(char) |   判断字符是否为大写字母   |
| int islower(char) |   判断字符是否为小写字母   |
| int isalpha(char) |     看看字符是否为字母     |
| int isdigit(char) |     看看字符是否为数字     |
| int isalnum(char) | 看看字符是否是字母或者数字 |
| int isspace(char) |     看看字符是不是空白     |
| int isblank(char) |     看看字符是不是空格     |
| int ispunct(char) |   看看字符是不是标点符号   |
| int isprint(char) |     看看字符能不能打印     |
| int iscntrl(char) |   看看字符是不是控制字符   |
| int isgraph(char) |   看看字符是不是图形字符   |
| int tolower(char) |      将字符转换为大写      |
| int toupper(char) |      将字符转换为小写      |



## 命名空间

用namespace定义

```C++
namespace lGame {
	int HP{ 1000 };
	int MP{ 1000 };
	int lv{ 1 };

	namespace Weapon {
		int lv{ 2 };
		int damage{ 3000 };
		namespace WeaponInfo {
			int lv = lGame::lv;
			int lv1 = lv;
			int lv2 = { 3 };
		}

	}
}
```

使用命名空间：

```C++
using namespace std;
	using std::cout;//推荐，防止冲突
	using lGame::HP;
	using namespace lGame::Weapon;

cout << HP << endl;
cout << lv << endl;

int c = lGame::Weapon::damage;
int lv = lGame::lv;
int lv1 = lGame::Weapon::WeaponInfo::lv;
int lv2 = lGame::lv;

cout << lGame::HP << endl;
cout << HP << endl;
cout << c << endl;
cout << lv << endl;
cout << lv1 << endl;
 cout << lv2 << endl;
```

命名空间变量**优先本地**，本地没有再变为命名空间的值
使用using namespace需要具体调用哪一层，比如调用第2层是using namespace lGame::Weapon



## 变量

**局部变量**：在函数内

**全局变量**：在开头定义（在内存中存放在全局变量区）

**当局部变量与全局变量冲突时用::访问**

**静态变量**：static 类型 变量名称；

```C++
static int count1{ 200 };//全局静态变量，main函数能调用
int ave(int a, int b) {
	static int count{ 200 };//局部静态变量不会因为括号结束而清除，会一直保留
	cout << count++ << endl;
	return (a + b) / 2;
}
count1 = 200;
```



**自定义变量**

```C++
#define 整数 int//define直接与编译器进行沟通
#define eLong long

typedef int eInt;

using eInt32 = int;
```

define定义的名字能在确定别的变量的类型中使用，using和typedef定义的名字则不能



**枚举**

1. 枚举类型默认int
2. 成员只能是整数类型，不能是float等
3. 枚举类型和其他类型转换需要强制转换
4. 默认情况下，枚举类型的下一个项的初始值是上一个项的初始值+1



## 运算

### 优先级

![image-20221019114752247](https://raw.githubusercontent.com/being1752/image/main/img/image-20221019114752247.png)

一元运算符的优先级比二元运算符高

**异或运算常用于加解密，a异或b能得到c，c异或b能得到a**



### 位运算符

| 符号 | 描述 | 运算规则                                                     |
| ---- | ---- | ------------------------------------------------------------ |
| &    | 与   | 两个位都为1时，结果才为1，否则为0                            |
| \|   | 或   | 两个位都为0时，结果才为0，否则为1                            |
| ^    | 异或 | 两个位相同为0，相异为1                                       |
| ~    | 取反 | 0变1，1变0                                                   |
| <<   | 左移 | 各二进位全部左移若干位，高位丢弃，低位补0                    |
| >>   | 右移 | 各二进位全部右移若干位，对无符号数，高位补0，有符号数，各编译器处理方法不一样，有的补符号位（算术右移），有的补0（逻辑右移） |



### int与unsigned int补位区别

1. unsigned int是无符号数，没有正负之分，所以左移右移都是补0

2. int是有符号数，有正负之分

   - 当int的数是正数时，和unsigned int一样左移右移都是补0

   - 当int的数时负数时，右移补0，左移补1



### 异或运算交换两数

```C++
a=0101
b=1010

a=a^b=1111
b=a^b=0101
a=a^b=1010
```




### 逻辑运算符

| 运算符 |  名称  | 注意区分 |                   说明                    |
| :----: | :----: | :------: | :---------------------------------------: |
|   &&   | 逻辑与 |    &     |     a&&b a和b 都为true时表达式为true      |
|  \|\|  | 逻辑或 |    \|    | a\|\|b a和b其中任意一个为true表达式为true |
|   ！   | 逻辑非 |    ！    |          ！a a为false时！a为true          |

**注意：**

1. 一元运算符的优先级大于二元运算符
2. 位运算大于逻辑运算
3. ~！>&>|



### 正反比较插入

```C++
int a[10]{ 1,3,22,25,31,32,58,73,98,105 };//正序
int acount = sizeof(a) / sizeof(int);//数组长度

for (int i = 0; i < acount; i++) {
	if (bcase ^ (x < a[i])) {
		getIndex = i;
		break;
	}
}
```

**分析：**

1. 这里能判断是因为无论正序还是倒序，**x < a[i]的值都能随着改变，bcase的a[0] > a[1]也是**
2. 当是正序时，x < a[i]的值为0，bcase的a[0] > a[1]为0，异或得0
3. 当是倒序时，x < a[i]的值为1，bcase的a[0] > a[1]为1，异或得0



## 输入输出

### **scanf和printf**

![image-20221021101303962](https://raw.githubusercontent.com/being1752/image/main/img/image-20221021101304708.png)

![image-20221021101333730](https://raw.githubusercontent.com/being1752/image/main/img/image-20221021101333730.png)



**char**

```C++
char Name[0x5];

scanf("%s", Name);
scanf_s("%s", Name,5);//5用来限定输入字符串的最大值，如果超过则不接受

scanf_s("%d %s %d", &num, Name, 10, &score);//字符串需要输入最大值，其他不需要
```

**wchar_t**

```C++
setlocale(LC_ALL, "chs");

wchar_t wstr[0x5];
wchar_t* wstr1 = new wchar_t[0x5];

wscanf(L"%s", wstr);
wcin >> wstr;

wprintf(L"%s", wstr);
```

**int和char**

```C++
int x[0x10];
char Name[0x10];

cout << x << endl;//0000001A2156F980,相当于指针，输出内存地址
cout << Name << endl;//烫烫烫烫烫烫烫烫烫，cout对于char类型会当作字符串处理，一直读取直到遇到\0结束

printf("%s", Name);//烫烫烫烫烫烫烫烫烫
printf("%X", Name);//E515FAD8
```

**strlen和wcslen**

计算字符串长度（不包括\0）

```C++
char Name[0x5];
scanf_s("%s", Name,5);
cout << strlen(Name) << endl;

wchar_t wstr[0x5];
wcin >> wstr;
cout << wcslen(wstr) << endl;


```



### cout

对于字符串数组，能直接打印字符串

原理是从该指针地址开始查看值，遇到'\0'就停止打印

```C++
char* str = (char*)"你好";
std::cout << str << std::endl;//你好
std::cout << str+2 << std::endl;//好
```

- cout用来输出char，如果用来输出wchar则会输出地址，需要使用wchar输出wchar
- wchar默认不输出中文，可以通过添加代码`setlocale(LC_ALL, "chs");`更改地区来输出中文



### getch

```C++
int uin = _getch();//得到输入按键的ascii码
```



## 循环

### if

**对于if (a = b)**

- a=b可以转化为int a=b,if(a=b)=if(b)
- if是否执行，取决于b的值，若b!=0，则执行，等于0则不执行



**对于if ((96 < flag) && (flag < 123))**
不能使用47 < flag < 58，这样会无法判断，要用(47 < flag)&&(flag < 58)



a > b ? a : b + 5000000;//**先算+500000**
cs ? (10000 / cs) : 0;//**等价于if(cs)**



**C++17新特性**

能把新建变量写入判断位置上

```C++
if (int a, b, c; true) {//C++17新特性，能把新建变量写入判断位置上
	a = 2;
	b = 3;
	c = 5;
}
```



### switch

**c++17新特性**

```C++
case 666:
		[[fallthrough]];
```

**提示编译器跳过检测break**



```C++
case 2:
	int b = 2;
	break;
case 3:
	break;
```

**报错是因为case 2没有加括号，当case 3时int b的值在case 3中有效但没声明，如果case 2是最后一项则可以不加括号**

正确：

```C++
case 2:
{
	int b = 2;
}
	break;
case 3:
	break;
```



### for

```C++
for (int i = 3; i < 1000; i += 2) 
```

**i += 2不能写成i+2，会无法成功+2**

```C++
for(int i : x){}
```



**两种写法：**

```C++
for (crackPass = 0; crackPass < 100000000; crackPass++) {
		if (crackPass == password)break;
	}
```

优化写法：

```C++
for (crackPass = 0; crackPass != password; crackPass++);
```



### while

```C++
while (inKey == 'Y' || inKey == 'y') {}
```

**注意**

while括号内i++和++i的结果都是一样的，都是**先执行完自增**再执行接下来的代码

```C++
int i = 0;
while (i++ < 10) {//因为判断表达式要运算，所以需要做完所有运算
	std::cout << i << std::endl;
}
```



### **for、goto、while、do while区别**

- for，代码直观，涉及具体执行次数的时候用,汇编为先mov再add再cmp je
- while，原则比for高，但是涉及具体执行次数不直观,汇编为mov cmp je再jmp
- do while,执行效率最高，6行汇编

**if和switch**

小数量判断用if（1-2个），多数量判断用switch



## 数组

### 定义

**一维数组**：`int studentId[10];`
**二维数组**：

```C++
int studentId[3][5]{
		{ 1,2,3,4,5 },
		{ 1,2,3,4,5 },
		{ 1,2,3,4,5 }
	};
```

**注意：**

1. 二维数组和一维数组一样都是内存连续的，二维数组是一组接一组地排列

2. 数组大小的值必须是已经赋值了的，不能是空的

3. 使用sizeof查看数组的大小，是根据数组所有元素的多少来确定的

4. 数组引用

   ```C++
   int d[100];
   int &b[100]=b;//编译器过不去
   int(&b)[100] = d;//引用100个元素的数组
   ```

5. 根据cout特性，对于一个字符串的指针，能**自动解析字符串长度打印**出来

   ```C++
   char* str;
   str=(char*)("你好");//const char[5]
   cout << &str << endl;//0000007647DBF7D8，str指针的内存地址
   cout << hex << (int) * str << endl;//ffffffc4，以十六进制打印str指针的第一个内容
   cout << (int*)str << endl;//00007FF6F8A2BD34，str指针内存储的地址
   cout << &("你好") << endl;//00007FF6F8A2BD34，字符串地址
   cout << &("你好") << endl;//00007FF6F8A2BD34，没改变
   cout << str << endl;//你好
   ```

6. 对于int数组

   ```C++
   int a[]{ 1,2,3,4,5 };
   int* a1;
   a1 = a;
   cout << &a1 << endl;//000000BDD399F668，指针的内存地址
   cout << *a1 << endl;//1,第一个位置的内容
   cout << a1 << endl;//00000033BCB7FC48，指针指向的内存地址
   ```

7. 在数组创建的那一刻，**sizeof函数的值就已经算好了**，接下来调用只是取出值而已

```C++
int studentId[3][5]{
	{ 1,2,3,4,5 },
	{ 1,2,3,4,5 }
};
cout << sizeof(studentId);//60
```



**遍历**

```C++
for (char i : studentId) {
		cout << i << endl;
}
```

作用跟迭代器差不多



### array

```C++
array<int, 2> studentId { 10001,10002 };

studentId.fill(999);//将所有元素都设置为目标数

cout << studentId.size() << endl;//返回拥有的元素数

cout << studentIdA[100] << endl;//这种方式会返回数据回来，不容易发现已经越界访问了

cout << studentId.at(100) << endl;//返回目标序号的内容，若没有会报错
```



### vector

```C++
vector<int> studentId;

studentId.assign(5, 100);//初始化

studentId.clear();//清空数组

cout<< studentId.empty() << endl;//判断当前容器是否为空，是则返回1

studentId.push_back(9600);//在后面插入数据
studentId.emplace_back(9600);//在后面插入数据
//使用push_back()函数需要调用拷贝构造函数和转移构造函数，而使用emplace_back()插入的元素原地构造，不需要触发拷贝构造和转移构造，效率更高。

cout << studentId.size() << endl;//获得当前元素总数

cout << studentId.at(2) << endl;//获得目标序号的元素
```



### unordered_map

添加元素

```C++
unordered_map<char,int> mp;
for(auto& ch : s)mp[ch]++;
```

#### 排序

**第一种方式**

排序前需要转化为`vector<pair<int, int>>`

```C++
 vector<pair<int, int>> vec;
 for(auto& it : mp)vec.push_back(it);
```

排序时使用lambda表达式

```C++
sort(vec.begin(),vec.end(),[](pair<int, int>& a, pair<int, int>& b){
	return a.second > b.second;
});
```

**第二种方式**

排序前需要转化为`vector<char> vec;`

```C++
vector<char> vec;
for(auto& it : mp) vec.push_back(it.first);
```

排序时使用lambda表达式，与第一种不同的是，需要传入mp

```C++
sort(vec.begin(), vec.end(),[&mp](char& a, char& b){
	return mp[a] > mp[b];
});
```



## 指针

### **定义**

```C++
//C
int* a{};//指针一定要初始化，不然有可能因为乱指导致破坏其他数据
int *b{};//上下一样

int aa{ 5000 };
int* pa{ &aa };//只能同类型指向

//c++
int* a=new int;
int* a=new int[50];
```

**注意：**

1. 指针一定要初始化，不然有可能因为乱指导致破坏其他数据

2. 当指针指向变量时，只能同类型指向

3. 指针的大小**跟操作系统相关与变量类型无关**，32位操作系统的指针大小为4字节，64位操作系统的指针大小为8字节

4. 数组第一个数据的内存地址不一定是数组的内存地址

   ```C++
   int a[5]{};
   int* b{ a };
   std::cout << &a << std::endl;//00000062CEB2FC98
   std::cout << &a[0] << std::endl;//00000062CEB2FC98
   std::cout << &b << std::endl;//00000062CEB2FCC8，内存地址与另外三个不一样
   std::cout << &b[0] << std::endl;//00000062CEB2FC98
   ```

5. 指针是一个存放了其他变量内存地址的变量，仍然需要内存来存放

   ```C++
   int a = 1;
   int* b{ &a };
   
   cout << &a << endl;//得到a的内存地址，0000005D7C8FF7B4
   cout << b << endl;//得到b中存放的内存地址，0000005D7C8FF7B4
   cout << *b << endl;//得到b指向的内存地址中的值，1
   cout << &b << endl;//得到b的内存地址，0000005D7C8FF7D8
   ```



### 指针数组

**一维指针数组：**

```C++
int studentId[5]{ 10001,10002,10003,10004,10005 };
int* ptrStudentId[5];
```

**二维指针数组：**

```C++
int studentIdA[2][2]{
		{10001,10002},
		{20001,20002}
	};
int* ptrstudentIdA[2][2];
```

二维指针是一块内存的连续



### 多级指针

| 代码                 | 值      | 内存地址 |
| -------------------- | ------- | -------- |
| int a{500};          | 500     | 0x50000  |
| int* ptr{&a};        | 0x50000 | 0x50100  |
| int** pptr{&ptr};    | 0x50100 | 0x50200  |
| int*** ppptr{&pptr}; | 0x50200 | 0x50300  |



### 指针类型转换

```C++
unsigned ua{ 2342 };
float* ptr{  };

ptr = (float*)&ua;

cout << ua << endl;//2342
cout << *ptr << endl;//3.28184e-42
```

本来不同类型编译器不能成功转换，但是指针本质是内存地址，都是8字节，所以能强制转换，打印结果分别按照**变量和指针类型**打印



```C++
int b{ 123 };
int* p{ &b };
long long* up{ (long long*)&p };

cout << *p << endl;//123
cout << *up << endl;//672561887092(不固定)
```

结果不同是因为数据原本是int 4字节，但是因为long long是8字节,需要向下继续读取，把别的内存地址的内容一起读取了



```C++
int b{ 123 };
int* tr{ &b };
*tr = -1;
char* r{ (char*)tr };
*r = 'A';

cout << b << endl;//-191
```

char是1字节，数据原本是int 4字节，上面操作修改了第一个字节，使得数据出现变化



### 指针内存加减

```C++
int a[]{ 10001,20001,300001,400001 };
int* ptr{ &a[0] };

cout << (*ptr)++ << endl;//10001,先显示*ptr的值，然后让*ptr的值++
cout << *(ptr++) << endl;//10002,指向的地址向后移1*指针类型的大小位
cout << *ptr++ << endl;//20001,等于(*ptr)++
```



### 常量指针

**理解：**本质是个指针，只是指向常量，所以无法修改常量的值

```C++
int a{ 1000 };
int b{ 2500 };

const int* ptr{ &a };//const int*的作用是禁止修改指针指向的内存地址里的值，但是可以修改指向的内存地址
ptr = &b;

cout << *ptr;//2500
```



### 指针常量

**理解**：本质是个常量，只是指向内存地址，因为本身是一个常量，所以无法修改内存地址，但是可以修改内存地址里的值

```C++
int c{ 1000 };
int d{ 1000 };
int* const pptr{ &c };//int* const的作用是禁止修改指向的内存地址，但是可以修改指针指向的内存地址里的值
*pptr = 9999;

cout << c;//9999
```



### 常量指针常量

**理解：**因为指针常量指向内存地址，所以常量指针常量指向内存地址的值，因为常量的性质，无法修改内存地址里的值

```C++
int e{ 1000 };
int f{ 1400 };
const int* const ppptr{ &a };

ppptr = &f;//报错
*ppptr = 123;//报错

cout << *ppptr;
```



### 数组与指针的关系

数组的底层实现就是指针，数组名本身是一个指针，数组元素是这个指针按照一定量偏移后对应的内存区域的内容



### 数组指针

数组指针就是带有数组信息的指针，与普通指向数组名的指针不同

```C++
int test[2][5]{
	{1001,1002,1003,1004,1005},
	{2001,2002,2003,2004,2005}
};

int* ptestA[5];//五个int类型的指针(数组指针)
int* ptest1{ (int*)test };//多维数组转一维数组
int(*ptest)[5] {test};//数组指针，5代表有5个元素位置

cout << test[1][4] << endl;//2005
cout << ptest1[9] << endl;//2005
cout << ptest[1][4] << endl;//2005

ptest = ptest + 1;//这里+1的值是数据类型的大小5*int

cout << sizeof(ptest1) << endl;
cout << sizeof(ptest) << endl;
```



### 动态内存分配

**C语言分配方式**

**都是分配到堆上，需要手动free**

`void* malloc(size_t size);`
**含义：**malloc 将为用户分配size_t字节个内存，并返回内存分配地址，如果分配失败则返回0
**注意：**malloc返回的是void类型的指针，需要强制转换

`void* calloc(size_t count,size_t size);`
**含义：**calloc将为用户分配count乘size_t字节个内存，并返回内存分配地址，如果分配失败则返回0
**注意：**calloc初始化所有变量为0，效率没malloc高

`void* realloc(void*  _Block,size_t size);`
**含义：**realloc将为用户重新分配内存，_Block是用户已经分配好的内存，size是要求重新分配的大小，函数返回重新分配后的内存
对于另外分配内存，流程是先分配，再复制，**再清除原数组，这时原数组指针变成悬挂指针**

`void free(void* _Block);`
**含义：**free将释放内存，_Block是要释放的内存地址
**注意：**

- free后的指针是悬挂指针，仍然指向原本地址，所以需要让它等于0，**若是释放已经释放的指针接下来的代码大概率会报错**
- **释放指针后再使用该指针不会报错**，这是隐患点

```C++
int* p = (int*)malloc(x * sizeof(int));//malloc返回的是void类型的指针，需要强制转换
int* pm = (int*)calloc(x , sizeof(int));//calloc初始化所有变量为0，效率没malloc高
int* pn = (int*)realloc(p, x);//realloc是将原本已经分配好的内存复制到重新分配的内存里

if (p == nullptr) {
	cout << "内存分配失败！";
}
else {
	p[0] = 9;
	p[1] = 2;
	p[2] = p[0] * p[1];
	cout << p[0] << " " << p[1] << " " << p[2] << endl;
}

cout << p << endl; //000001E52BFEFA30
	
free(p); p = 0;//free后的指针是悬挂指针，仍然指向原本地址，所以需要让它等于0
free(pm); pm = 0;
free(pn); pn = 0;
```



**分配到栈上，不需要free**

`void *alloca(size_t size);`

**注意：**由于栈的大小有限，且才为2MB左右（window下，linux为1M），所以能够申请的空间是有限的。一旦申请的空间超过了函数栈的大小，即申请空间失败，该函数不会像 malloc() 函数那样返回空指针，而是会导致栈溢出，很有可能使得程序崩溃。



**C++分配方式**

`数据类型*指针变量名称=new 数据类型；`
`数据类型*指针变量名称=new 数据类型[数量]；`

**注意：**失败返回0，底层实现用malloc

`delete 指针；`
`delete[] 指针；`

**注意：**

- 释放一个指针用`delete(指针);`
- 释放一组指针用`delete[] 指针;`
- **释放指针后再次使用该指针**会报错，即使**该指针已经被其他指针指向**后也是会报错

```C++
int* p{}, * pa{};

p = new int;
pa = new int[x];//失败返回0，底层实现用malloc

*p = 500;
p[0] = 500;

delete(p);
delete[] pa;
```



### 内存复制

`void* memcpy(void* _Dst,const void* _Src,size_t _Size);`
**含义：**memcpy可以将`_Src`区域的内存复制到`_Dst`区域，复制的长度为`_Size`
**注意：**数组的值可以不用全部复制，复制后原数组还在

```C++
int a[5]{ 1,2,3,4,5 };
int* b = new int[5];
memcpy(b, a, 5 * sizeof(int));

for (int i=0; i < 5;i++)cout << b[i] << endl;
```



`void* memset(void* _Dst,int val,size_t _Size);`
**含义：**memset可以将指定内存区域每一个字节的值都设置为`val`,`_Size`为要设置的长度(字节)
**注意：**val的值要根据_Size的值的类型来确定范围，不然会出现截断，比如当要复制的值的类型是int时，val的取值范围是0~0xFF

```C++
int* c = new int[5];
memset(c, 0, 4 * 5);
memcpy(a, c, 4 * 5);

for (int i = 0; i < 5; i++)cout << a[i] << endl;
```



### 引用

- 引用本质就是被阉割的指针，虽然取值引用变量得到的是原值的内存地址，但是引用变量也是占用内存的

- **引用确定后就无法更改引用的指向**

- 引用可以理解为指针常量`int* const`，汇编都是一样的语句，但是有些情况下无法使用常量指针代替引用，比如

  ```C++
  int i=5, j=6;
  int* const array[]={&i,&j};
  
  int i=5, j=6;
  int& array[]={i,j};//报错，二义错误，是将数组元素array[0]本身的值变成8，还是将array[0]所引用的对象的值变成8
  ```

- 强行更改引用指向，以下代码只能在Debug 模式x86下运行，Debug 模式x86下的int变量前后都有4字节调试信息，也就是一个int变量有12字节

  ```C++
  int i = 5, j = 6;
  int& r = i;
  int* addr;
  int dis;
  
  pi = &i;
  pj = &j;
  dis = (int)&j - (int)&i;//计算相差地址
  addr = (int*)((int)&j + dis);//指向引用r的地址
  
  (*addr) = (int)&j;    //将j的地址赋给引用r（此处把r看作指针）
  ```

  

**语法：**

- 数据类型& 变量名称{引用对象的名称}；
- 使用&能查看原本变量的地址

```C++
int a{ 231 };
int& la{ a };//引用确定后就无法更改引用的指向
int& la1{ a };
int& la2{ a };

cout << la << endl;//231
cout << &a << endl;//00000061184FFAC4
cout << &la << endl;//00000061184FFAC4
cout << &la1 << endl;//00000061184FFAC4
cout << &la2 << endl;//00000061184FFAC4
```

**注意：**

- 引用必须要先初始化，不然会报错

  ```C++
  int& la2;//报错
  ```
  
- 引用确定后就无法更改引用的指向

  ```C++
  int a{ 231 };
  int c{ 20323 };
  int& la1{ a };
  
  la = &c;//报错
  la = c; //相当于a=c
  ```

- 使用for循环需要注意&

  ```C++
  int b[]{ 1001,1002,1003,1004 };
  
  for (int x : b) {
  	x = x + 1;
  }//这里只是对x的值进行增加，没有对b的值增加
  
  for (int& x : b) {
  	x = x + 1;
  }//因为有&是引用，这里能对b的值增加
  ```
  
- 引用和指针的各种数据类型

  ```C++
  int b = { 1 };
  int& c = b;
  int * a = { &b };
  
  cout << typeid(c).name() << endl;//int
  cout << typeid(&c).name() << endl;//int * __ptr64
  
  cout << typeid(a).name() << endl;//int * __ptr64
  cout << typeid(*a).name() << endl;//int
  cout << typeid(&a).name() << endl;//int * __ptr64 * __ptr64
  
  cout << c << endl;//1
  cout << &c << endl;//000000F75799F8F4
  
  cout << a << endl;//000000F75799FA34
  cout << *a << endl;//1
  cout << &a << endl;//000000F75799FA10
  ```



### std::forward与&

- `std::forward` 与 `&` 虽然都涉及到**参数的引用类型**，但它们的作用是不同的。
- `&` 是**引用运算符**，用于声明一个变量或参数为左值引用类型，表示该变量或参数可以绑定到一个左值（lvalue）上。左值引用类型的变量或参数可以修改其所绑定对象的值。
- 而 `std::forward` 是一个**函数模板**，用于在泛型编程中实现**完美转发（perfect forwarding）**。它根据参数的类型和值类别来决定**将参数转发为左值引用或右值引用**。通常用于将参数以原始的方式转发给其他函数。

```C++
#include <iostream>
#include <utility>

void foo(int& x) {
    std::cout << "x is an lvalue reference\n";
}

void foo(int&& x) {
    std::cout << "x is an rvalue reference\n";
}

template<typename T>
void bar(T&& x) {   // x is a universal reference
    foo(std::forward<T>(x));  // forward x as either an lvalue or rvalue
}

int main() {
    int a = 42;
    bar(a);     // call foo(int&)
    bar(100);   // call foo(int&&)
    return 0;
}
```




### `*&`与`&*`

```C++
int a = 1;
int* p;
p = &a;
std::cout << &*p << std::endl;//先取p存储的地址里的值，再取该地址,0116FC2C
std::cout << *&p << std::endl;//先取p的地址，再取该地址存储的值,0116FC2C
std::cout << *&a << std::endl;//先取a的地址，再取该地址存储的值,1
```



###  结构体指针

```C++
typedef struct Role {
	short a;
	short b;
	int HP;  
	int MP;
}*PRole, * XPole, _ROLE;//PRole是结构体指针，_ROLE是普通结构体类型

Role user;
PRole puser = &user;

cout << sizeof(user) << endl;//12

user.MP = 50;
cout << puser->HP << endl;//500

puser->MP = 50;
cout << puser->HP << endl;//50

puser = &monster;
monster.HP = 5000;
cout << ( * puser).HP << endl;//5000
```

**解释：**因为内存会对齐，short的2字节不够int的4字节，如果只有a，那么就会空2字节用来对齐int的4字节

**注意：**"."用于结构体，"->"用于结构体指针



## 智能指针

**作用：**用于解决原生指针安全性不足的弊端



### 唯一指针

**语法：**`std::unique_ptr<类型> 变量名称{};`

**变量形式：**

```C++
unique_ptr<int> intPtr1{ new int{5} };
cout << intPtr1 << " " << *intPtr1 << endl;//0000018085BC9770 5
```

**数组形式：**

```C++
unique_ptr<int[]> intPtr2{new int[5] {5,4,3,2,1}};
cout << intPtr2 << " " << intPtr2[1] << endl;//00000270ED220820 4
```

**注意：**

智能指针无法指向指针或智能指针

```C++
intPtr2 = intPtr1;//报错，无法指针指向指针
unique_ptr<int[]> intPtrA{ intPtr1 };//报错，无法指针指向指针
```

**C++14以后声明：**

```C++
unique_ptr<int[]> intPtrA{make_unique<int[]>(5)};//有5个数组元素
unique_ptr<int> intPtrB{ make_unique<int>(5) };//指针声明初始化为5
```



**reset()**

**作用：**

1. 指针地址清零
2. 内存空间释放

```C++
unique_ptr<int[]> intPtrA{make_unique<int[]>(5)};

intPtrA.reset();
```



**get()**

**作用：**得到intPtrB指向的内存地址

```C++
unique_ptr<int> intPtrB{ make_unique<int>(5) };
int* a1{intPtrB.get()};

cout << a1 << " " << *a1 << endl;//0000024329A45220 5
cout << intPtrB << " " << *intPtrB << endl;//0000024329A45220 5
```



**release()**

**作用：**将指针指向空，但是不释放原来内存空间的值

```C++
unique_ptr<int> intPtrB{ make_unique<int>(5) };
intPtrB.release();

cout << intPtrB << " " << *intPtrB << endl;//*intPtrB报错，因为没有值，intPtrB的输出值是0000000000000000
```



**move()**

**作用：**让intPtrD指向intPtrC指向的内存地址，然后将intPtrC指向空

```C++
unique_ptr<int[]> intPtrC{ make_unique<int[]>(5) };
unique_ptr<int[]> intPtrD{  };

cout << intPtrC << " " << intPtrD << endl;//000001B00BBE0A00 0000000000000000

intPtrD = move(intPtrC);

cout << intPtrC << " " << intPtrD << endl;//0000000000000000 000001B00BBE0A00
```



### 共享指针

**语法：**`std::shared_ptr<类型> 变量名称{};`

**新式：**

```C++
shared_ptr<int> ptrA{ make_shared<int>(5) };
shared_ptr<int> ptrB{ ptrA };//ptrB指向ptrA的内存地址

cout << ptrA << " " << *ptrA <<endl;//000002984058F2A0 5
cout << ptrB << " " << *ptrB <<endl;//000002984058F2A0 5
```

**注意：**

- **新式（make_shared）**共享指针不支持数组



**旧式：**

```C++
	shared_ptr<int[]> ptrD{new int[5] {1,2,3,4,5}};
	shared_ptr<int> ptrE{ new int(5) };
	
	cout << ptrD << " " << ptrD[1] << endl;//000002259B3913A0 2
	cout << ptrE << " " << *ptrE << endl;//000001748A3B6A40 5
```

旧式能设置数组



**use_count(）**

**作用：**查看当前内存地址有几个共享指针在指

```C++
shared_ptr<int> ptrA{ make_shared<int>(5) };
shared_ptr<int> ptrB{ ptrA };
shared_ptr<int> ptrC{ ptrA };

cout << ptrB.use_count() << endl;//3
```



**unique()**

**作用：**查看当前共享指针是否唯一指向当前的内存地址

**注意：**C++17已经取消使用

```C++
shared_ptr<int> ptrA{ make_shared<int>(5) };
shared_ptr<int> ptrB{ ptrA };
shared_ptr<int> ptrC{ ptrA };

cout << ptrA.unique() << endl;//0
```



**reset()**

**作用：**将当前共享指针设置为nullptr，如果当前智能指针是最后一个拥有该指针的对象，那么将会释放内存

```C++
shared_ptr<int> ptrA{ make_shared<int>(5) };

ptrA.reset();
```



### 指针安全

#### 动态内存情况

指针没删除，内存没释放

```C++
int* r;
{
	int* a = new int[50];
	a[2] = 255;
	r=a;
}
cout << t[2] << endl;//255
```

指针删除，但是内存被释放

```C++
int* e;
{
	int* r = new int[5];
	e = r;
	r[0] = 1;
	delete[] r;
}
cout << *e << endl;//-572662307，说明内存空间已经被释放了，如果是想继续用该指针，可以用共享指针
```



#### 唯一指针情况

```C++
int* p;
{
		unique_ptr<int[]>b{ make_unique<int[]>(50) };
		b[2] = 250;
		p = b.get();
		cout << p[2] << endl;//250
}
cout << p[2] << endl;//-572662307，说明内存空间已经被释放
```



#### 共享指针情况

```C++
shared_ptr<int> c{  };
{
	shared_ptr<int> s{ make_shared<int>(5) };
	c=s;
	cout << *s << endl;//5
}
cout << *c << endl;//5，指针成功被保留
```



#### enable_shared_from_this

使用sharedptr时，如果遇到下面的情况，会重复析构，造成崩溃

```C++
class MyClass {
public:
	std::shared_ptr<MyClass> getShared() {
		//return shared_from_this();
		return std::shared_ptr<MyClass>(this);
	}
};
void run() {
	auto sp1 = std::make_shared<MyClass>();
	auto sp2 = sp1->getShared(); //!! 重复析构，未定义行为
    /*sp1 和 sp2 有两个不同的控制块 管理相同的 Foo*/
}
```

**原因：**

- sp1与sp2是单独构造指向同一块内存，并不共享控制块

**解决办法：**

- 让MyClass继承enable_shared_from_this
- 让MyClass类通过shared_from_this返回指向自己的共享指针

```C++
class MyClass : public std::enable_shared_from_this<MyClass> {
public:
	std::shared_ptr<MyClass> getShared() {
		return shared_from_this();
	}
};
void run() {
	std::shared_ptr<MyClass> sp1 = std::make_shared<MyClass>();
	std::shared_ptr<MyClass> sp2 = sp1->getShared();
}
```



## 联合体

自定义数据类型

**联合体和结构体区别**

- 结构体中每个成员单独一个内存，内存对齐以空间最大的数据类型为准
- 联合体中每个成员共享内存，所以union的数据类型大小由其最大的成员变量决定

**赋值方式**

```C++
er->er.nHP = 3500;
er->ls.nHP = 3500;

(* er).er.nHP = 3500;
(* er).ls.nHP = 3500;
```

**注意：**

- union中的任意一个成员变量值的变动，都可能导致其他成员变量的值发生变动

- union类型的大小由其最大的成员变量决定

  ```C++
  union USER {
  	short sHP;
  	int nHP;
  	double fHP;
  };
  int main() {
  	USER user;
  	cout << sizeof(user) << endl;//8
  
  	user.nHP = 0;
  	user.sHP = -1;//[0xFF][0xFF][0][0][][][][]
  
  	cout << user.sHP << endl;//-1，正常赋值
  	cout << user.nHP<< endl;//65535，此时int的内存区域为0xFFFF0000，由于小端模式，读取内存变成0x0000FFFF，0x0000FFFF的值就是65535
  }
  ```



**结构体大小**

```C++
struct user {
	long long wq;
	short shp;
	char fg;
	double gf;
};

cout << sizeof(*er) << endl;//24
```

```C++
struct user {
	long long wq;
	short shp;
	double gf;
	char fg;
};

	cout << sizeof(*er) << endl;//32
```

**分析：**

- long long和double都是8字节，short是2字节，char是1字节
- 对于第一种情况，short和char夹在long long和double中间，因为内存对齐，8字节能容下short和char，所以24字节
- 对于第二种情况，short和char分别在long long和double中间，short和char无法在8字节内拼接，只能单独开8字节，所以32字节



**匿名结构体**

```C++
struct {
	short sHP;
	int nHP;
	double fHP;
}er;
```

**匿名联合体**

```C++
union {
	short sHP;
	int nHP;
	double fHP;
}ls;
```

**组合起来**

```C++
union USER {

	union {//匿名联合体
		short sHP;
		int nHP;
		double fHP;
	}ls;

	struct {//匿名结构体
		short sHP;
		int nHP;
		double fHP;
	}er;

};

int main() {
	USER* er = new USER{};

	er->er.nHP = 3500;
	er->ls.nHP = 3500;

	(* er).er.nHP = 3500;
	(* er).ls.nHP = 3500;

}
```



## 字符串

### 初始化

```C++
string str1{ "0123456" };
cin >> str1;
cout << str1 << endl;//0123456

string str4;
str4 = 56;
cout << str4 << endl;//8，直接等于数字就会转化为ASCII码
```

**截取**

```C++
string str1{ "0123456",2,3 };//2为截取起始位置（从0开始），3为截取的长度
cout << str2 << endl;//234
```

**复制**

```C++
string str2(2, 65);//2为复制个数，65为字符的ASCII码
string str3(2, 'A');//也可以这么写

cout << str2 << endl;//AA
```

**插入**

```C++
string id{ "id=;" };
id.insert(3, "testId");//第3个位置插入testId
cout << id << endl;//id=testId;

id.insert(3, "killertestid", 6, 6);//第3个位置插入killertestid的第6个开始长度为6的字符串
cout << id << endl;//id=testidtestId;

id.insert(3, "killertestid", 6);//第3个位置插入killertestid的第1个开始长度为6的字符串
cout << id << endl;//id=killertestidtestId;

id.insert(3, 6, '*');//第三个位置插入6个*
cout << id << endl;//id=******killertestidtestId;
```



### 字符串操作

**原生字符串拼接**

```c++
char str[0x10] = "123";
char strB[0x10] = "123";

char strC[0x20];
memcpy(strC, str, strlen(str));
memcpy(strC + strlen(str), strB, strlen(strB) + 1);

cout << strC << endl;//123123
```

**C++字符串拼接**

```C++
//第一种方式
string Str, ls;
ls = "123";
Str = ls + " " + "456";//开头一定要string的变量才不会报错
cout << Str << endl;//123 456

//第二种方式
ls = "123";
string str5 = ls + to_string(123);
cout << str5 << endl;//123123

//第三种方式
str = string{ "123" } + "123";
cout << str << endl; //123123

//第四种方式
str1="abc" + string{ "123" };
cout << str1 << endl;//abc123

//第五种方式
str3 = str3 + "asdf";
cout << str3 << endl;//asdf

//第六种方式
str3 += "nba" + string{ "sss" };
cout << str3 << endl;//nbasss

//第七种方式
#define SoftName "EDY"
#define SoftVersion "2.0"
str4 = "123""123";//这种方式要两个常量才可以
str4 = SoftName SoftVersion;
cout << str4 << endl;//EDY2.0

//第八种方式
char a[2]{ "1" };
string str5{ "sadf" };
str5 += a;
cout << str5 << endl;//sadf1
str5 += a+'o';//这里相当于str5 += char(a+'o')，也就是说a的ASCII码与'o'的ASCII码相加
```

**注意：**

- string的拼接不能直接字符串拼接字符串

  ```c++
  str1 = "abc" + "123" ;//报错
  ```

- str的内存地址和字符串的内存地址有可能不同

  ```C++
  string str{ "12345" };
  
  cout << str[0] << endl;
  cout << (int) & str << " " << (int)&str[0] << " " << (int)&str[1] << endl;//-913179752 -913179744 -913179743
  
  str = str + "sdaf";
  cout << (int)&str << " " << (int)&str[0] << " " << (int)&str[1] << endl;//后面的sdaf有可能因为当前位置内存不够，与前面的字符串另外动态分配内存到其他内存地址，所以str的内存地址和字符串的地址有可能不同
  ```



**c_str()**

```C++
const char* baseStr = str.c_str();//c_str()方法能得到一个const char*的指针 指向str储存 字符数据的内存空间
cout << (int)baseStr << endl;//-913179744
```



**data()**

```C++
const char* baseStr1 = str.data();//data()方法的作用和c_str()方法一样，C++17标准下，data()方法得到的是一个char*的指针
cout << (int)baseStr1 << endl;//-913179744
```



**replace()**

```C++
string strId{ "id=user;" };
strId.replace(3, 4, "zhangsan!");
cout << strId << endl;//id=zhangsan!;

strId.replace(3, 4, "zhangsan!sdfasdfs",8);//替换4个字节
cout << strId << endl;//id=zhangsangsan!;
```



**erase()**

```C++
string strId{ "id=user;" };
strId.erase(3);//第3位后面全部删除掉
strId.erase(3,4);//第3位后面4个字符删除掉
cout << strId << endl;//id=
```



**clear()**

```C++
strId.clear();//删除全部字符串，不能指定
```



**append()**

```C++
string str6{ "123" };
str6.append(" 456").append(" 789").append(" 456", 2);//append的用法和初始化提到的一样，如append(" 456",2)
cout << str6 << endl;//123 456 789 4
```



**substr()**

```C++
string str7;
str7 = str6.substr(7).substr(2);//从第7位开始截断，substr的用法和初始化提到的一样
cout << str7 << endl;//89 4
```



**length()**

```C++
string str8{ "123" };
cout << str8.length() << endl;//3

string str9{ "123撒打发" };
cout << str9.length() << endl;//9，因为length()函数中文算作2字节
```



### 转化

![image-20221026111342848](https://raw.githubusercontent.com/being1752/image/main/img/image-20221026111342848.png)

```C++
string cIn = "123";
int x = std::stoi(cIn);

cout << x <<endl;//123
```



### 数字转字符串

第一种，二个循环

```C++
unsigned int val = -1;
char str[12]{};
char strR[12]{};
int len{};
bool bfs = val < 0;
if (bfs)val = -1 * val;
do {
	str[len++] = val % 10 + 48;
	val = val / 10;
} while (val);
if (bfs)str[len++] = '-';
for (int i = 0; str[i]; i++) {
	strR[i] = str[len - i - 1];
}
std::cout << strR;
```

第二种，一个循环

```C++
int val = -1;
char str[12]{};
int len{ 11 };
bool bfs = val < 0;
if (bfs)val = -1 * val;
do {
	str[--len] = val % 10 + 48;
	val = val / 10;
} while (val);
if (bfs)str[--len] = '-';
std::cout << &str[len];
```

第三种，去掉if，精简代码

```C++
int val = 0;
char str[12]{};
int len{ 11 };
bool bzs = (val >= 0);
val = val * (bzs * 2 - 1);//bzs只有2个结果，一个1，一个0，0-1=-1，1-1=0，所以乘2，0*2-1=-1，1*2-1=1
do {
	str[--len] = val % 10 + 48;
} while (val = val / 10);
//推导过程
//	0|len-(1-bzs)
//	1|len-(1-bzs)
//	0|'-'*(bzs+1)*(1-bzs) + str[len] * bzs
//	1|'-'*(bzs+1)*(1-bzs) + str[len] * bzs
str[len = len - (1 - bzs)] = '-' * (bzs + 1) * (1 - bzs) + str[len] * bzs;
std::cout << &str[len];
```



### 字符串流

```C++
#include<sstream>
using std::stringstream;

stringstream strS;

strS << "你好" << "123";//字符串流
string strX = strS.str();//从字符串流中提取字符串

cout << strX << endl;//你好123
```



## 函数

### 语法

```C++
返回类型 函数名称(参数，参数，.....参数){
	函数功能区
	return 返回值;
}
参数的语法 参数类型 参数名称
```



**函数本质**

- **函数名是一个内存地址**，所以无论是`*`还是`&`，都返回函数指针的内存地址

  ```C++
  cout << add << endl;//00007FF6A3051316，函数名就是一个内存地址
  cout << &add << endl;//00007FF6A3051316，函数名就是一个内存地址
  cout << *add << endl;//00007FF6A3051316
  ```

- 函数的功能是编译好的二进制代码

- 在汇编中，当执行函数时，call指令执行后会将当前函数下一步的地址压入栈再跳转，通过push ebp将当前的ebp地址压入栈

- 可以打印出函数的数据

  ```C++
  #include<bitset>
  
  char* str = (char*)Add;//将函数的内容转化为char字符串
  
  for (int i = 0; i < 30; i++) {
  		printf("%X\n", (unsigned char)str[i]);//十六进制打印函数内容
  
  		cout << bitset<8>(str[i]) << endl;//二进制8位打印函数内容
  }
  str[0] = 25;//报错，因为有函数保护
  ```

  

**汇编执行函数流程**

- 当执行函数时，call指令执行后会将当前函数下一步的地址压入栈再跳转
- push ebp将当前的ebp地址压入栈，然后esp向上提升一位
- mov ebp，esp 将esp的值赋值给ebp
- sub esp 8 esp开辟空间存放变量
- 执行具体函数指令
- pop ebp 将ebp弹出栈，并且根据弹出栈的值回到原来ebp的地方
- ret 将函数跳转地址弹出栈，并让CPU指向该地址，然后继续执行



### 传参数组

```C++
void Sort(int a[4]) {
	cout << sizeof(a) << endl;//8，因为int a[4]被当成指针了
}

void Sort(int a[], unsigned count) //可以写为void Sort(int* a, unsigned count)
{}

void Sort(int a[][2],unsigned count)//调用二维数组
{}

void Sort(int a[]) {
	a[0] = 1;
}
int a[5]{ 230,234,1234,4576,255 };
cout << a[0] << endl;//230
Sort(a);//void Sort(int a[])
cout << a[0] << endl;//1，由此可以知道数组传参函数中的改变会影响原本数组
```

**引用传递数组**

```C++
void ave(int(&art)[100]) {
	cout << sizeof(art) << endl;
	for (auto x : art);
}
```



### 传参结构体

```C++
//指针传参
struct Role {
	int Hp;
	int Mp;
	int damage;
};
Role rl{200,300};

//指针传参
int Exp(Role* rl) {
	return rl->Hp + rl->Mp;
}
Role* r{ &rl };
cout << Exp(r) << endl;

//引用传参
int Exp(Role& rl) {
	return rl.Hp + rl.Mp;
}
cout << Exp(rl) << endl;
```

在函数里使用结构体的话**需要构建，浪费内存**，所以使用指针或引用就能直接**调用已经分配好**的



**函数返回结构体**

```C++
typedef struct Role {
	char* Name;
	int Hp;
	int MaxHp;
	int Mp;
	int MaxMp;
	int lv;
} *PROLE, ROLE;

ROLE CreateMonster(const char* str, int Hp, int Mp) {//用变量方式会2次赋值造成性能浪费
	Role rt{ cstr(str),Hp,Hp,Mp,Mp,1 };
	return rt;
}

PROLE CreateMonster(const char* str, int Hp, int Mp) {//结构体指针方式

	//Role rt{ cstr(str),Hp,Hp,Mp,Mp,1 };
	//return &rt;//这种方式返回的是局部变量，因为rt不是指针

	PROLE rt=new ROLE{ cstr(str),Hp,Hp,Mp,Mp,1 };
	return rt;
}

Role& CreateMonster(const char* str, int Hp, int Mp) {//返回引用，缺陷是无法返回空

	PROLE rt = new ROLE{ cstr(str),Hp,Hp,Mp,Mp,1 };
	cout << typeid(rt).name() << endl;
	cout << typeid(*rt).name() << endl;
	return *rt;
}
```



### 引用

```C++
bool act(const role& acter, role& beacter) {
	beacter.hp -= acter.damage;
	return beacter.hp <= 0;
}

if (act(monster, user))cout << "角色死亡！" << endl;
```

主函数**不需要单独定义引用**

对于string字符串，可以以引用的形式放到函数修改



对于不同类型的引用传递

```C++
float p = 250.60f;
Add(p);//报错，类型转换出问题，要相同类型才行

void Add(int& a) {
	a += 100;
}
```



**引用的函数修改变量**

```C++
int& bigger(int& a, int& b) {
	return a > b ? a : b;
}

int f{ 324 }, g{ 123 };
bigger(f, g) = 500;
cout << f << endl;//500
cout << g << endl;//123
```

因为返回的是引用



### 指针

**函数参数的指针理解**

函数的**指针参数**是**单独一个内存空间，里面存放传入参数的值**，若传入的参数是主函数的变量的内存地址，则该指针参数能指向该变量的地址，从而进行修改

```C++
void xc(string* as) {
	as->append("asdf");
}
void xc(int* as) {
	(* as) = 123;
}

string sa{"sadf"};
xc(&sa);
cout << sa << endl;//sadfasdf

int a{ 345 };
xc(&a);
cout << a << endl;//123
```

如果主函数传入的是**指针**，因为指针本身储存指向变量的内存地址，所以还是**等于传入指针指向的变量的的内存地址**

```C++
void xc(int* as) {
	(* as) = 123;
}

int a{ 345 };
int* b{ &a };
xc(b);//等于xc(&a);
cout << a << endl;//123
```

若是改变了函数指针参数的指向，并不等于改变了主函数中传入指针的指向，因为**两个指针的内存地址不相等**

```C++
role user{ "奥特曼",1000,1500,5001231 };
role monster{ "小怪兽",1500,100,100 };
role* pRole = &monster;

cout << pRole << endl;//00000091D992F538
cout << &pRole << endl;//00000091D992F588

if (act(user, pRole))cout << pRole->Name << endl;//小怪兽，指向失败

bool act(const role& acter, role* beacter) {//role*&表示role类型指针的引用

	cout << beacter << endl;//00000091D992F538
	cout << &beacter << endl;//00000091D992F4C8

	beacter = (role*)&acter;//根据主函数，这句话是想将pRole指向user的地址
	return bEend;
}

//解答：因为传入的是指针变量，函数里为这个变量开设了内存，改变了这个变量的内存不等于改变原本指向的内存，所以为了能改变原来的值，在此基础上使用了引用，引用能共用原本指针的内存地址，所以能改变指向
//更改：将bool act(const role& acter, role* beacter)改为bool act(const role& acter, role*& beacter)，即将beacter从role类型指针变为role类型指针的引用
```



函数的指针参数可以**添加const表示常量**，主函数定义时**不需要**加上const

```C++
bool act(const role* acter, role* beacter) {
	beacter->hp -= acter->damage;
	return beacter->hp <= 0;
}

if (act(&monster, &user))cout << "角色死亡！" << endl;
```



### 函数指针

**语法：**函数返回类型 (*函数指针变量名)(参数类型 参数名称，...参数类型 参数名称)；

```C++
int Add(int a, int b) {
	return a + b;
}
int (*pAdd)(int, int){Add};//申明函数指针，变量名可以省略
int (*pAdd)(int, int) = Add;//另一种指向形式，发生隐式转换变成&Add，包含整个函数空间

cout << sizeof(pAdd) << endl;//8
cout << pAdd(1, 2) << endl;//3
```

#### **类型转换**

```C++
float AddX(int a, int b) {
	return (a + b)/2;
}
char(*pAddX)(int, int) {(char(*)(int,int))AddX};//强转类型，指出返回类型与传入参数
```

#### **自定义类型**

```C++
using usingFadd = char(*)(int, int);//自定义数据类型，声明一个函数指针类型
typedef char(*typedefAdd)(int, int);//自定义数据类型，声明一个函数指针类型

typedefAdd pAddXX { (typedefAdd)AddX };//typedef，(typedefAdd)是类型转换
usingFadd pAdd1{ (usingFadd)AddX };//using，(usingFadd)是类型转换
```

#### **typedef加与不加的区别**

- 不加typedef表示该函数指针只是一个指针，能直接赋值函数

- 加typedef表示该函数如果需要使用需要实例化

  ```C++
  int Sum(int a, int b)
  {
  	return a + b;
  }
  
  int(*pSum)(int a, int b);
  pSum = Sum;
  
  typedef int(*pSum)(int a, int b);
  pSum f = Sum;
  ```

  

#### **结构体函数**

```C++
struct Role {
	int hp;
	int mp;
};
using pRole = int(*)(int hp,int mp);//变量不加名称就按照顺序读取，加的话则能指定变量读取
int Exp(Role rl) {//Role rl本质上还是两个参数，因为能正常读取
	return rl.hp + rl.mp;
}

pRole pExp = (pRole)Exp;
cout << pExp(100, 350) << endl;//x86才能正常，x64的话结构体参数第二位读取不了
```

#### **函数做参数**

```C++
using pRole = int(*)(int hp,int mp);
int Add(int a, int b) {
	return a + b;
}
int Test(int a, int b, pRole x) {
	return x(a, b);//指定函数处理
}

cout << Test(120, 234, Add) << endl;
```

#### **函数指针和指针函数区别**

- 函数指针本身是一个指针，即可以指向特定类型函数的指针

  ```C++
  int (*pAdd)(int, int){Add};
  ```

- 指针函数是一个返回指针的函数

  ```C++
  int* xAdd(int a,int b);
  ```

- 也就是说加括号是函数指针，不加是指针函数

#### **函数指针函数返回函数**

```C++
int test(int a) {
	return a;
}
typedef int (*HOOK)(int);//函数指针

//普通的例子
HOOK a = test;//函数指针指向函数
HOOK test1() {//函数指针函数返回函数
	return a;
}

//类的例子
class HO {
public:
	HOOK b = test;//函数指针指向函数
	HOOK test2() {//函数指针函数返回函数
		return b;
	}
};

int main()
{
	HOOK H= (HOOK)test;//函数指针初始化
	HO HE;//类初始化
	std::cout << H(1) << std::endl;
	std::cout << test1()(1) << std::endl;//test1()函数能返回test函数，后面的(1)给返回的test函数传参，结果为1
	std::cout<<HE.test2()(1) << std::endl;//test2()函数能返回test函数，后面的(1)给返回的test函数传参，结果为1
}
```



### 默认参数

默认参数在没有传参时，按照默认参数执行函数，有传参则按照传入的参数执行函数

```C++
void Sort(int ary[], unsigned count, bool BigSort=true){} //默认参数只能放在参数末尾

int Add(int a,int& b=100)//错误，因为引用没有实例来指向，若是指针就可以
```

**注意：**

- c++可以在**类的声明**中，也可以在**函数定义**中声明缺省参数，但**不能既在类声明中又在函数定义**中同时声明缺省参数。
- 因此，将**定义或声明中的任一个**缺省参数删除即可。



### 不定量参数

#### main函数

- argc为参数个数，argv是二维数组，存放参数序号对应的参数
- 参数末尾有一个**空指针nullptr**，该空指针不计入参数个数

```C++
int wmain(int argc, TCHAR* argv[]);//支持unicode
int _tmain(int argc, TCHAR* argv[]);//_tmain()是个宏，用于支持unicode，是unicode环境则是wmain，不是unicode则是main

int main(unsigned count,char* arc[]) {//argc为参数个数，argv是二维数组，存放参数序号对应的参数
	cout << "一共有" << count << "个参数" << endl;
	for (int i = 0; i < count; i++)
		cout << "参数[" << i << "]=[" << arc[i] << "]" << endl;

	for (int i = 0; arc[i]; i++)
		cout << "地址：" << (int)arc[i] << "参数[" << i << "]=[" << arc[i] << "]" << endl;

}
```



#### 自定义函数

```C++
int x = AverAge(3, 53, 465, 11);//第一个数为后面不定量参数的个数（仅仅这里的用法，真正用法不需要传入个数），可以传入不同类型的参数，能读就行
cout << x << endl;//11

int AverAge(short count, ...) {
	va_list arg;//va_list是char类型的指针
	
	va_start(arg, count);//从count开始读取参数，这里count用于传入参数个数
	
	int x = va_arg(arg, int);//读取第一个参数（3）
	cout << x << endl;

	x = va_arg(arg, int);//读取下一个参数（53）
	cout << x << endl;

	x = va_arg(arg, int);//读取下一个参数（465）
	cout << x << endl;
    
    va_end(arg);//释放指针
    
    return x;
}
```



#### **对于va_arg()函数**

```C++
for (int i{}; i < count; i++) {
	cout << "arg:" << (int)arg << endl;//每次递增8，代表每执行一次va_arg就会先读取再向上递增一次，读取下一个数
	sum+= va_arg(arg, int);
}
```



### 右值引用

**语法：**

```C++
int&& e = 234;//&&代表右值引用，并不是引用的引用
```



#### **左值和右值**

- 左值是指有内存空间存放
- 右值是指没有具体内存空间存放，也就是临时存放

```C++
int c = 32 + 234;//左值赋值，左值是指有内存空间存放，右值是指没有内存空间存放，也就是临时存放
```

**注意：**左值并不一定是等号左边，右值并不一定是等号右边

```C++
*(a + 1) = 100;//a+1是个临时计算的值，也就是右值，所以等号左边并不一定是左值
```



#### **对于a++和++a**

```C++
cout << &(a++) << endl;//报错，a++返回右值，因为a++先自增再返回原来的临时对象
cout << &(++a) << endl;//正常，++a返回左值，因为++a先自增再返回现在的对象
cout << &(5+a) << endl;//报错，5+a返回右值
```



#### **函数引用和右值使用的区别**

**引用**

```C++
void Add(int& a) {//引用一定要接受明确的内存空间
	cout << a << endl;
}

int x = 32 + 234;
Add(x);//对于Add(int& a)，计算32 + 234，先分配内存空间保存，再传进来，这样会造成一次性能浪费

```

**右值**

```C++
void Add(int&& a) {
	cout << a << endl;
}

Add(32 + 234);//对于Add(int&& a)，直接右值引用，没有多一次性能浪费，仍然可以正常传入左值
```

**结构体**

```C++
struct Role {
	int hp;
	int mp;
};
Role CreateMonster() {
	Role rt{ 100,200 };
	return rt;
}
void Show(Role&& rl) {//不用新创建结构体保存传进来的数据，直接右值引用
	cout << rl.hp << endl;
}

Show(CreateMonster());
```



### 重载

#### **使用**

```C++
float ave(int a, int b) {//报错，因为返回值返回什么都可以，但是传入参数相同
	return (a + b) / 2;
}
int ave(int a, int b,int c) {
	return (a + b + c) / 2;
}
int ave(float a, float b, float c) {
	return (a + b + c) / 2;
}

cout << ave(1, 2) << endl;//1
cout << ave(1, 2, 3) << endl;//3
cout << ave(1.0f, 2.0f, 3.0f) << endl;//3
cout << ave(1.0f, 2.0f) << endl;//通过类型转换到int，执行第一个函数
```

#### **数组和指针**

```C++
int ave(int* pa, int count) {
	return 0;
}
int ave(int pa[], int count) {//报错，因为数组就是指针
	return 0;
}
```

#### **引用和普通变量**

```C++
int ave(int& a, int& b) {//编译器能通过
	return (a + b) / 2;
}
int ave(int a, int b) {
	return (a + b) / 2;
}

int a{ 1 }, b{ 2 };
cout << ave(a, b) << endl;//报错，ave(1, 2)中的1和2是常量，不可能传入引用，这个a和b都是变量，在没有int ave(int a, int b)函数的情况下能传入int ave(int& a, int& b)函数
cout << ave(1, 2) << endl;//1，传入int ave(int a, int b)函数
```

#### **强转和float**

```C++
int ave(int& a, int& b) {
	return (a + b) / 2;
}
float ave(float a, float b) {
	return (a + b) / 2;
}

char c = 100;
char d = 200;
cout << ave((int)c, (int)d) << endl;//强转类型后的参数是临时变量，无法传入引用，所以还是float
```

#### **const**

```C++
int ave(int a, int b) {
	return (a + b) / 2;
}
int ave(const int a, const int b) {
	return (a + b) / 2;
}
cout << ave(1, 2) << endl;//报错
```

```C++
int ave(const int& a, const int& b) {
	return (a + b) / 2;
}
int ave(int& a, int& b) {//编译器能通过
	return (a + b) / 2;
}

int a{ 1 }, b{ 2 };
cout << ave(a, b) << endl;//重载ave(int& a, int& b)

const int a{ 1 }, b{ 2 };
cout << ave(a, b) << endl;//重载ave(const int& a, const int& b)
```

#### **默认参数**

```C++
int ave(int a, int b,int c) {
	return (a + b + c) / 2;
}
int ave(int a, int b, int c,int d=100) {
	return (a + b + c) / 2;
}
cout << ave(1, 2, 3) << endl;//报错，因为传参无法判断重载哪一个函数
```



### 模板

#### **本质**

对于每次使用的参数类型都会生成对应的函数，例如生成`int bigger(int a, int b)`和`int bigger(float a, int b)` 

#### 函数模板

##### **一个参数**

```C++
template<typename type1>
type1 bigger(type1 a, type1 b) {//支持任何类型
	return a > b ? a : b;
}
int bigger(int a, int b) {//能和模板重载
	a = b;
	return a > b ? a : b;
}

template<typename type1>
type1 ave(type1 a, type1 b, type1 c) {//一个模板对应一个函数
	type1 x = a * b;
	type1 p[1999];
	return (a + b + c + x) / 2;
}

cout << ave(1.0f, 2.1f, 465.1f) << endl;
cout << ave(12, 2, 3) << endl;
cout << ave((char)11, (char)12, (char)13) << endl;

cout << ave<int>(1.0f, 2.1f, 465.1f) << endl;//将参数全部转化为int

int d{ 100 }, e{ 100 }, f{ 100 };
f = *ave(&d, &e, &f);//结果是错的，因为传入的是内存地址
```

##### **多个参数**

```C++
template<typename T1>
T1 ave1(T1 a, T1 b) {
	return (a + b) / 2;
}
cout << ave1(200, 200) << endl;

template<typename T1, typename T2>
T1 ave2(T2 a, T1 b) {
	return (a + b) / 2;
}
cout << ave2(200.01f, 200) << endl;
cout << ave2<float>(100, 200.01f) << endl;//指定T1为float
cout << ave2<float,int>(100, 200.01f) << endl;//指定T1为float,T2为int

template<typename TR, typename T1, typename T2>
TR ave3(T1 a, T2 b,int c=200) {//使用和平时的函数一样
	T1 x;
	T2* px;
	return (a + b) / 2;
}
cout << ave3<int, float, double>(100, 200.01f) << endl;
cout << ave3<double>(100, 200.01f) << endl;//指定TR为double
```

##### **默认参数**

```C++
template<typename TR = int,typename T1,typename T2>//TR默认指定int
TR ave1(T1 a, T2 b) {
	return (a + b) / 2;
}

template<typename T1, typename T2, typename TR = T1>//TR默认指定T1
TR ave2(T1 a, T2 b) {
	return (a + b) / 2;
}
```

##### **非类型模板参数**

```C++
template<int max=2000, int min=1000, typename T>//非类型模板参数需要<>指定，写前面是因为这样可以不用定义模板参数类型
bool ChangeHp(T& hp, T damage) {
	hp -= damage;
	max = 2500;//无法过编译器，模板的参数是替换，所以参数是常量，常量无法修改值
	if (hp > max)hp = max;
	return hp < min;
}
int x = 1000;
ChangeHp<x, 1000>(hp, 100);//报错，因为模板的参数是常量不是变量

template<typename T, T max = 2000, T min = 1000>//默认参数
bool ChangeHp(T& hp, T damage) {
	hp -= damage;
	if (hp > max)hp = max;
	return hp < min;
}
```

##### **模板数组**

```C++
//第一种，用于处理固定大小
template<typename T,short count>
T ave(T(&ary)[count]) {
	T all{};
	for (int i = 0; i < count; i++) {
		all += ary[i];
	}
	return all / count;
}
int a[5]{ 1,2,3,4,5 };
cout << ave(a) << endl;//3

//第二种，用于处理不固定大小
template<typename T>
void Sort(T* ary, unsigned count) {}
int a[5]{ 123,145,534,685,3456 };
Sort(a, 5);
```



##### **定义例外情况**

```C++
template<typename type1>
type1 bigger(type1 a, type1 b) {//支持任何类型
	return a > b ? a : b;
}
int bigger(int a, int b) {//执行顺序中重载高于模板
	return a > b ? a : b;
}
template<>
int* bigger(int* a, int* b) {//定义例外情况
	return *a > *b ? a : b;
}
```

##### **模板重载**

```C++
template<typename type1>
type1 ave(type1 a, type1 b, type1 c) {//模板重载
	return (a + b + c) / 3;
}
template<typename type1>
type1 ave(type1 a, type1 b) {
	return (a + b) / 3;
}
```

##### **注意引用**

```C++
template<typename type1>
type1 bigger(type1 a, type1 b) {
	return a > b ? a : b;
}

int a{ 200 }, b{ 100 };
int c;
c = *bigger(&a, &b);//在type1 bigger(type1 a, type1 b)的情况下，比较的是a和b的内存地址，b的内存地址比a大
cout << bigger(&a, &b) << endl;//000000979FB4F9F4，可以知道返回的是内存地址，所以需要加*
cout << c << endl;//100

//正确使用引用情况
template<typename type1>
type1 bigger(type1 &a, type1 &b) {
	return a > b ? a : b;
}
int a{ 200 }, b{ 100 };
int c;
c = bigger(a, b);
cout << c << endl;//200

template<typename T1, typename T2>
decltype(auto) bigger(T1 &a, T2 &b) {//只有在T1和T2的数据类型相同，也就是不能在有类型转换的时候，才能传引用出来
	return a > b ? a : b;
}
int a = 50;
int b = 5000000;
bigger(a, b) = -1;
cout << b << endl;//-1
```



#### 类模板

```C++
template<class op>
class Sample {
public:
	Sample(op a) {
		cout << "op" << endl;
	}
};

Sample<int> a(1);//op
```

其他操作与函数模板差不多

**枚举与类模板结合**

```C++
enum Operator {
	None,
	Accept,
	Recv,
	Send,
	Error
};
template<Operator op>
class Sample {
public:
	Sample() {
		cout << op << endl;
	}
};

Sample<Accept> a();//Accept
```



#### auto

```C++
char a;
auto b{ &a };//变成指针

const int c{ 1 };
auto b = c;//变成int类型，并不是const类型

int e{ 234 };
int& e1{ e };
auto b = e1;//变成int类型，而非int&

auto b = ave(1, 2);//利用函数返回值来确定类型
```

**拖尾函数**

```C++
auto bigger(int& a, int& b)->int& {//拖尾函数，强制返回int引用，当然，不一定非要返回引用
	return a > b ? a : b;
}
```

**C++11以后**利用decltype特性可以以下写法，但是比较累赘

```C++
auto bigger(int& a, int& b)->decltype(a > b ? a : b) {
	return a > b ? a : b;
}
```

**C++14以后写法**

```C++
decltype(auto) bigger(int& a, int& b) {
	return a > b ? a : b;
}
```



### decltype

```C++
decltype(a + b--) x;//只是根据表达式得到类型，能保留const、引用和指针，但是没有初始化
```

- 如果decltype内的表达式经历了运算，那么得出的数据类型是根据运算结果是否有固定的内存地址（左值）来决定的，如果**有固定的内存地址**则得出的类型为**该类型的引用类型**，如果**没有固定的内存地址**，则得出的类型为**该结果的类型**

  ```C++
  int h{ 100 }, i{ 200 };
  int* ha{ &h };
  
  decltype(a + b) x;//相当于int x
  decltype(*ha) x;//相当于int& x
  decltype(ha[0])x;//相当于int& x
  decltype((h)) x;//加个括号也算作一次运算，所以相当于int& x
  ```

- 如果decltype内的**表达式是一个函数**，那么得出的数据类型是根据**函数的返回类型**来确定的

  ```C++
  decltype(ave(100,200)) x;//相当于int x
  ```

**注意**

- decltype**不会真的执行**表达式或函数的运算，只是根据表达式或函数来**判断运算结果的类型**

- auto则会**执行表达式或函数**，根据返回类型确定类型

  

### 内联函数(inline)

**使用inline关键字声明**

```C++
inline int add(int a, int b) {//内联函数有可能直接a+b，没有出栈入栈，算是个对编译器的建议
	return a + b;
}

add(1, 2);//建议编译器直接计算add(1+2)
```

内联函数会建议编译器把这个函数处理成内联代码以提升性能，至于是否采用由编译器决定



## 联合编程

[C++与C的区别终于说清楚了！ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/395364319)

### 声明

- 不存在内存分配，只是告诉编译器函数的存在
- 参数不写或写别的参数也是可以的，不用分配内存空间
- 可以声明很多次，但是定义只能定义一次
- 声明函数自动添加extern关键字

```C++
int ave(int a231, int bsdf);//声明，参数不写或写别的参数也是可以的，不用分配内存空间
int ave(int a231, int bsdf);//可以声明很多次，但是定义只能定义一次
extern int ave(int a231, int bsdf);//声明函数自动添加extern关键字
```

#### 预声明

- 预声明告知编译器，该定义会提前使用
- 如果在类A中没有调用类B的函数或者成员变量， 没有真正使用类B，那么通过引用或指针的形式能解决编译问题
- 原理是通过**放置指针占位符来跳过解释**，等到后面编译到类B时再回过来解释类B
- 类的预声明使用形式只有两种：指针或引用

```C++
class B;//预声明

class A {
public:
    void test(B b) {}//报错，没有使用指针或者引用形式传入类B
    void test(B& b) {}//正确
};

class B {};
```



### 定义

- 涉及内存的分配和访问，定义只能定义一次
- 声明变量时，不加变成普通变量，extern关键字针对全局
- 定义变量可以放在下面

```C++
extern int all;//声明变量，不加变成普通变量，extern关键字针对全局

int all;//定义变量
int all{ 123 };//定义变量

int main() {
	extern int all; //extern关键字针对全局，在函数内变成普通变量
	all = 100;
	cout << all << endl;//100
	int c = ave(1, 2);
}
int ave(int a, int b) {//定义，要分配内存空间
	return (a + b) / 2;
}

int all{ 123 };//定义变量可以放在下面
```



### 头文件

指.h文件

- 源文件引用头文件后会把头文件的内容复制到该源文件里
- 静态变量、静态函数、内联函数在使用的cpp各个文件中并不共用内存地址，这样就能在头文件定义函数
- **不能在头文件定义函数、定义全局变量（除非加extern）和实现静态成员变量**，不然如果**多个CPP文件同时引用该头文件**会造成重复定义

```C++
int ave(int a, int b);
int bigger(int a, int b);

static int atest = 999;//static变量在使用的cpp各个文件中并不共用内存地址，这样就能在头文件定义变量

extern int acount;//声明变量

//静态函数
static void Test() {//静态函数在使用的cpp各个文件中并不共用内存地址，这样就能在头文件定义函数

}

void Test() {//整个工程中只有这一个函数的内存地址，当然，对于这个函数需要走头文件声明源文件定义

}//不能在头文件定义函数，不然多个CPP文件引用该头文件会造成重复定义

//内联函数
inline int TestA() {//内联函数在使用的cpp各个文件中并不共用内存地址，这样就能在头文件定义函数
	return 1;
}
```



**头文件引用**

**第一种方式**

```C++
#pragma once//整个文件针对引用的源文件只引用一次，但是需要编译器的支持（实际上第二种方法的简写，原理一样）
```

**第二种方式**

```C++
#ifndef _HEMATH_//效果与#pragma once一样
#define _HEMATH_//原理是当第一次包含emath.h时，因为没有定义#define _HEMATH_，所以执行#define _HEMATH_到#endif的代码
#endif//第二次包含时，因为已经定义了#define _HEMATH_，所以跳过执行
//只作用于#define _HEMATH_到#endif
```

在源文件中

```C++
#define _VIP//定义了就执行接下来的代码，没有就不执行
void ShowWelcome() {
//#ifndef//与#ifdef互反
#ifdef _VIP
	cout << "你好！" << endl;
#endif
}
```



### 源文件

指.cpp文件



**为什么模板类需要放在.h文件实现，正常类可以放在.cpp文件实现？**

- 模板类在编译时不会真的编译，因为模板仅在需要的时候才会实例化出来
- 当另一个cpp文件编译没有定义的模板类时，不知道另一个包含模板类定义的cpp文件，也不会去查找，而是会创建一个具有外部连接的符号并期待连接器能够将符号的地址决议出来
- 然而编译器无法实例化，因为本cpp文件里没有模板类的定义

### extern

1. 变量默认是内部链接，函数默认是外部链接
2. 因为cpp文件和c文件对于函数的定义不同
3. c文件定义ave函数为_ave，cpp文件定义ave函数为?ave@YANXZ
4. 所以需要extern关键字来进行C函数与C++函数相互转换
5. 全局变量也需要添加extern，因为多个cpp引用同一个头文件的全局变量会重复定义，造成冲突然后编译失败，加入extern后代表全局，让全局变量与函数一样只定义一次，头文件只声明，一个cpp文件定义，其他cpp文件使用

**C函数转C++函数**

```C++
//第一种定义方式
extern "C" int ave();//定义为C的函数
extern "C" int pve();

//第二种定义方式
extern "C" {
	int ave();
	int pve();
}

//第三种定义方式
extern "C" {
#include"extern1.h";
}
```



**C++函数转C函数**

- 如果C调用C++函数，C无法识别extern "C"
- `__cplusplus`定义在C中无法识别，所以可以在CPP文件里定义`__cplusplus`运行，C文件就不定义
- 声明为C函数就不支持重载

```C++
extern "C" int ave();//如果C调用C++函数，C无法识别extern "C"，所以在这种情况下失败

#ifdef __cplusplus//__cplusplus定义在C中无法识别，所以可以在CPP文件里定义__cplusplus运行，C文件就不定义
extern "C"
#endif
int ave();
//声明为C函数就不支持重载
```



### 制作SDK

1. 创建edoyun.cpp

   ```C++
   #include"edoyun.h"
   #define VERSION "1.0"
   int ave(int a, int b) {
   	return a * b / 2;
   }
   
   namespace edoyun {
   	const char* GetVersion() {
   		const char* str = VERSION;
   		return str;
   	}
   }
   ```

2. 制作edoyun.h

   ```C++
   #pragma once
   int ave(int a, int b);
   namespace edoyun {
   	const char* GetVersion();
   }
   ```

3. 工程属性的生成，将生成的lib文件和制作的edoyun.h**放在一个文件夹**内

4. 新项目中在**VC++目录**中将该文件夹的目录地址填入**包含地址**，以此来让编译器找到edoyun.h

5. 将该文件夹目录填入**库目录**以此让编译器找到lib文件位置

6. 通过以下语句调用该库

   ```C++
   #pragma comment(lib,"edoyun.lib")
   ```

7. 如果不想输入以上语句，可以在**属性-链接器-输入-附加依赖项**中将库添加上去就能直接调用lib文件里的函数



### 链接

变量、函数、结构都有自己的名字，这些名字具有不同的链接属性，链接器就是根据这些链接属性把各个对象文件链接起来

**链接属性分为三种**

- **内部链接属性：**该名称仅仅在本转换单元有效

  ```C++
  static int a = 100;
  const int pt = 1000;
  int a1 = 100;
  
  static void test(){}//外部转内部，不推荐
  ```

- **外部链接属性：**该名称在其他转换单元中也有效

  ```C++
  extern const int pt = 1000;
  extern const int pt;//新文件中加extern关键字声明就能使用了
  
  inline int a=350;//为了保证有统一定义需要写在头文件里，写在源文件里会不统一定义
  ```

- **无连接属性：**该名称仅仅能够用于该名称的作用域内访问



### static和inline

- static是在**不同单元里有自己的内存地址**，变量和函数都是一样，这个CPP文件被static修饰的变量和函数在另一个CPP文件里是不同的
- inline修饰变量是**C++17新特性**，如果inline写在CPP文件内，**变量和函数的值是依据main函数的CPP文件里被inline修饰的变量和函数的值**，所以最好需要写在头文件内统一标准



### define

- 用于将标识符A定义B
- 不可以是数字开头或纯数字

```C++
#define 整数 int//简单，但是不安全，因为可以把int定义为float
整数 a {250};
#define 234 int//不可以是数字开头或纯数字

#define _HH_ int a//把_HH_替换为后面的内容
_HH_{ 1 };//int a{1}

#define _in_//无意义，因为等于空，但是加入可以提高代码阅读性
#undef _in_//这句话以后删除定义

#define SUM(X,Y) X+Y*3
cout << SUM(X, Y) <<endl;//7

#define RELEASE(x) delete[] x;x=nullptr;//定义组合语句
int* pa = new int[10];
RELEASE(pa);//清除指针指向与指针本身

#define BIGGER(X,Y) ((X)>(Y)?(X):(Y))//加括号是为了防止无法处理符号
cout << BIGGER(1, 2) << endl;//2

#define SHOW(X) cout<<#X//加#能把X处理为字符串
SHOW(232SDFSDF)<<endl;//232SDFSDF

#define SHOW1(X,Y) void X##Y(){std::cout<<#X;}//定义函数名与函数
test22();//test
```



### 命名空间

#### 全局命名空间

**定义：**

- 所有具有链接属性的对象，只要没有定义命名空间，就默认定义在全局命名空间中
- 全局命名空间中成员的访问不用显示的指定，当局部名称覆盖了全局名称时才需要显式的指定全局命名空间

```C++
const int p{};//全局

int p = 20;
std::cout << p;//20
std::cout << ::p;//0，通过全局命名空间访问extern int a
```



#### 命名空间扩展

```C++
namespace t {
	int value;
}
namespace t {//命名空间拓展
	int e{24};//定义
	extern int r;//声明，需要C++17标准
}
```



#### 未命名命名空间

- 不给命名空间指定名称，将会声明一个未命名的命名空间
- 未命名的命名空间中声明的内容一律为**内部链接属性**，包括用**extern声明的内容**，未命名的命名空间**仅仅在本转换单元中有效**

```C++
namespace {//解决函数与其他源文件冲突的问题，与其他源文件互不干涉
	void THack() {
		std::cout << "Thack!\n";
	}
}

THack();
```



#### 命名空间别名

```C++
namespace htd {
	extern int height;
	void test();
	namespace hack {
		int ha{ 1 };
	}
}

namespace hack = htd::hack;//命名空间别名
```



#### 声明定义

**变量**

```C++
//第一种写法
namespace t {
	extern int r;//声明，需要C++17标准
}
namespace t {
	int r{250};//定义
}

//第二种写法
//头文件htd.h
namespace htd {
	extern int height;
}

//源文件
#include"htd.h"
int htd::height{ 12 };
```

**函数**

```C++
//第一种写法
namespace htd {
	extern int height{23};
	void test();
}
void htd::test() {
	std::cout << htd::height;
}

htd::test();//23

//第二种写法
namespace htd {
	extern int height{ 23 };
	void htd::test() {
		std::cout << htd::height;
	}
}

htd::test();//23
```



### 预处理

```C++
//变量定义
#ifdef UNICODE
wchar_t a;
#elif GBK
char16_t a;
#else
char a;
#endif

//版本控制
#define VERSION 102
#define SENDSMS 0

#if (VERSION==(100+3))&&SENDSMS
void sendSms() {

}
#elif VERSION==100||SENDSMS
void sendSms() {
#if VERSION==104//嵌套
#endif
void sendSms() {

}
#else
void sendSms() {

}
#endif
```



### 预定义宏

**标准预定义宏**

编译器支持ISO C99和ISO C++17标准指定了这些预定义宏

| 宏            | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| `__func__`    | 函数的名称                                                   |
| `__DATE__`    | 源文件的编译日期                                             |
| `__TIME__`    | 当前转换单元的转换时间                                       |
| `__FILE__`    | 源文件的名称                                                 |
| `__LINE__`    | 当前的行号                                                   |
| `__cplusplus` | 当翻译单元为C++时，`__cplusplus`定义为一个整数文本，否则为未定义 |



**MSVC预定义宏（只在微软编译器能用）**

| 宏               | 说明                                                    |
| ---------------- | ------------------------------------------------------- |
| `_CHAR_UNSIGNED` | 如果char类型为无符号，该宏定义为1否则为未定义           |
| `__COUNTER__`    | 从0开始，每次使用都会递增1                              |
| `_DEBUG`         | 如果设置了/lDd /mDd /mTd，该宏定义为1，否则为未定义     |
| `__FUNCTION__`   | 函数名称，不含修饰名                                    |
| `__FUNCDNAME__`  | 函数名称，包含修饰名                                    |
| `__FUNCSIG__`    | 包含了函数签名的函数名                                  |
| `_WIN32`         | 当编译为32位ARM，64位ARM，X68或X64定义为1，否则为未定义 |
| `_WIN64`         | 当编译为64位ARM或x64定义为1，否则未定义                 |
| `__TIMESTAMP__`  | 最后一次源代码修改的时间和日期                          |

```C++
int ave(int a, int b) {
	std::cout << __func__ << std::endl;
	return (a + b) / 2;
}
int main() {
	std::cout << __func__ << std::endl;//main
	ave(100, 200);//ave
	std::cout << __DATE__ << std::endl;//Nov  1 2022
	std::cout << __TIME__ << std::endl;//21:25:33
	std::cout << __FILE__ << std::endl;//C:\Code\c\C++基础\18-编译器理解函数\编译器理解函数\预定义宏.cpp
	std::cout << __LINE__ << std::endl;//11
	std::cout << __cplusplus << std::endl;//199711

#ifdef _CHAR_UNSIGNED
	std::cout << "char unsigned";
#else
#endif

	std::cout << __COUNTER__ << std::endl;//0

#ifdef _DEBUG
	std::cout << "debug mode!\n";
#endif
	std::cout << __FUNCTION__ << std::endl;
	std::cout << __FUNCDNAME__ << std::endl;
	std::cout << __FUNCSIG__ << std::endl;
}
```



### 断言机制

#### assert

语法：assert(bool表达式)

- 断言机制
- 如果括号内的bool表达式为false，则会调用std::abort()函数，弹出警告对话框
- assert宏需要头文件cassert

```C++
#define NDEBUG//在当前单元关闭assert，必须放在#include<cassert>前
#include<cassert>

int c;
std::cin >> c;
assert(c);//如果C为0则出现警告框
```



#### static_assert

语法：static_assert(bool表达式，"错误信息");

- static_assert用于编译时检查条件
- 不需要`#include<cassert>`
- 为0时错误列表显示错误信息
- 与assert不同，static_assert主要是用来在编译时检查重要的条件
- 因此检查的bool表达式中，只能用于常量，不能用于变量
- 内容为中文会乱码
- C++17新语法：static_assert(bool表达式);

```c++
static_assert(sizeof(int*)==4, "sadf");//不能判断变量，只能用于常量，为0时错误列表显示错误信息，内容为中文会乱码
static_assert(sizeof(int*) == 4);//C++17新语法
```



## 面向对象编程

### OOP

- OOP即面向对象编程，本质是一种编程思想
- OOP应当遵循OOD(面向对象设计)的原则，所有坏的OOP代码基本都是违反了OOD原则
- 面向对象编程特点：封装、继承、多态
- 虽然结构体也能放函数，但是不推荐，最好放在类里实现

**变量不放入public中就等于放入private**

```C++
class CROLE {//变量不在public默认不可以被访问

public://将下面的内容公开，使得能直接被访问
    int hp;
    int mp;
    

    void Init() {//初始化
        hpRecover = 3;
    }

    bool Act(NPC* beater) {
        beater->hp -= damage;
        return beater->hp > 0;
    }

private://将下面的内容私有，只有这个类才能访问
    int hpRecover;

public://下面的内容被公开
    int damage;
    int diamond;
};
```



### 成员函数

**第一种在源文件直接创建**

```C++
class ROLE {
private:
	int hpRecover;
	void Init();
public:
	int hp;
	int damage;
	void Act(ROLE& role);//函数不占类的空间
};

void ROLE::Act(ROLE& role) {//也可以写进类里
	role.hp -= damage;
}

void ROLE::Init() {
	hpRecover = 3;
}
```

**第二种在头文件声明，源文件定义函数**

头文件Role.h

```C++
class Role{
private:
	int hpRecover;
	void Init();
	int hp;
	int lv;
public:
	int damage;
	void Act(Role& role);//函数不占类的空间
	inline int GetHp() {
		return hp;
	}
	Role* bigger(Role* role);

	Role& SetLv(int newLv);
	Role& SetHp(int newHp);
	Role& SetDamage(int newDamage);
};
```

源文件Role.cpp

```C++
void Role::Act(Role& role) {
	role.hp -= damage;
}
void Role::Init() {
	hpRecover = 3;
}
Role* Role::bigger(Role* role) {
	role->lv > lv ? role : this;
}
Role& Role::SetLv(int newLv) {
	lv = newLv;
	return *this;
}
Role& Role::SetHp(int newHp) {
	hp = newHp;
	return *this;
}
Role& Role::SetDamage(int newDamage) {
	damage = newDamage;
	return *this;
}
```



**空类的对象也占用内存地址**

```C++
class P {//空类
};

P p1, p2;

std::cout << &p1 << std::endl;//006FF927，相差一个字节，说明即使是空类，在创建对象后也会填充1字节内容，把对象区分开
std::cout << &p2 << std::endl;//006FF926，要用Release模式才能成功，因为DeBug模式会加调试代码到程序中
```



### const

#### 定义

- const修饰的对象不能修改其成员变量的值
- const是静态对象，**修饰的变量、对象必须初始化**

```C++
const Role user;
user.lv = 2;//报错，不能改变，因为对象是const属性

Role monster;
const Role* puser{ &user };
puser->damage = 2;//报错，不能改变，因为指针是const属性
```

#### const函数

- const函数不能在**内部修改成员变量**，外部传入变量可以修改

  ```C++
  inline int GetHp(int& b)const {
  	lv = 2;//const函数不能在内部修改成员变量
      b = lv;//可以修改
  	return hp;
  }
  ```

- 不能通过**调用别的函数**修改成员变量

  ```C++
  inline int GetHp()const {
  	SetLv(2);//也不能通过调用别的函数修改成员变量
  	return hp;
  }
  ```

- **this**不能在const函数修改值

  ```C++
  inline int GetHp()const {
  	this->damage = 2;//变成常量指针，报错
  	return hp;
  }
  ```

- 对于引用函数，引用会传递指针出去，能修改值，这样与const相违背

```C++
int& Role::GetLv() const {//不能编译，因为引用会传递指针出去，能修改值，这样与const相违背
	return lv;
}
const int& Role::GetLv() const {//这样就能把传出的指针变成常量指针，无法修改值
	return lv;
}
```

- 第二个const让本函数无法修改类内的成员变量的值
- `const int&`是常量引用，引用本身确定指向之后就无法改变指向，等价于`int * const`，

```C++
const EBuffer& operator>>(short& data) const  {
	data = (short)atoi(c_str());
	return *this;
}
```



**例子分析：**

```C++
class A {
public:
	const int const add() const {}
};
```

- 第一个const表示该函数**返回值为常量**，即该函数返回的值不能被修改。
- 第二个const没有意义，既不能把返回值变成常量，也不能把函数变为常量函数
- 第三个const表示该函数**不会修改this指针所指向的对象**。也就是说，该函数是一个常量成员函数，它保证在**函数内部不会修改任何成员变量**，包括this指针所指向的对象的成员变量，**但是可以修改静态变量**，也能修改传入的变量。



#### const对象

const对象无法调用**非const函数**

```C++
int Role::GetDamage() {
	return damage;
}

const Role user;
user.GetDamage();//报错，const对象无法调用非const函数
```

使用**重载**能解决这一问题

```C++
int Role::GetDamage() {
	return damage;
}
int Role::GetDamage() const {//与int Role::GetDamage()重载
	return damage;
}
```



#### 类型转换

对于C++，使用**const_cast**将const对象传入非const函数

```C++
void test(Role* p) {
	p->SetHp(5000);
}
test((Role*)(&user));//C语言形式，传入const对象
test(const_cast<Role*>(&user));//C++形式，传入const对象
```



#### mutable

mutable声明的成员变量可以被const成员函数修改

```C++
mutable int lv;//加了mutable后，const函数能对其进行修改
```



### this

- this是一个常量指针，指向对象首地址，指向成员变量时用`->`
- 如果类里有虚函数，则this指针的地址就是虚表指针的地址，然后才是变量的地址

```C++
this->da=100;
```

对于下面的函数

```C++
const EBuffer& operator>>() const {
	//传入对象是const EBuffer，所以返回也要const EBuffer，函数也要是const
	//this是常量指针：EBuffer *const类型，*this是EBuffer对象
	//加上const的this是const EBuffer *const类型，那么*this是const EBuffer类型
	return *this;
}
```



### **return this**

- `temp* get(){return this;}`，返回的this是地址，即一个指向对象的指针
- `temp  get(){return *this;}`，返回的是对象的拷贝，即一个临时变量
- `temp& get(){return *this;}`，返回的是对象本身



### 构造函数

#### 定义

- 构造函数用于在新的类实例时执行初始化操作
- 构造函数名与所在的类重名，没有返回值
- 每个类如果开始没有定义构造函数，会创建默认构建函数，该函数**无参数，无返回值**
- 如果调用了自定义构造函数，默认构造函数不会自动运行
- 若是创建了与默认相同的构建函数，会覆盖掉默认构建函数
- 需要放在**public**下面

```C++
T{
    public:
    int hp;
    int mp;
    
	T() {//构造函数，任何一个类至少有一个构造函数
		hp = 100;
		mp = 800;
	}

	T(int _hp) {//explicit关键字用于禁止类型转换
		hp = _hp;
	}

	T(int _hp, int _mp) {
		hp = _hp;
		mp = _mp;
	}
	T(T &t) {
		hp = t.hp;
		mp = t.mp;
	}
}
```

**定义默认构造函数**

```C++
Role(){}//默认构造函数
Role() = default;//默认构造函数，效率更高
```

**默认参数构造函数**

```C++
Role() = default;
Role(int _lv=500) { lv = _lv; };//报错，因为不填参数能执行两种构造函数，要达到效果的话需要删掉上面的构造函数
```



#### explicit

explicit关键字修饰的构造函数会禁止类型转换

```C++
T(int _hp) {
		hp = _hp;
}
bool IsBig(T r) {//对于user.IsBig(300)，执行过程为300初始化r，使用构造函数T(int _hp)，然后再比较
		return r.hp > hp;
}
user.IsBig(300);
```

为了防止以上情况发生，对T(int _hp)构造函数进行以下修改

```C++
explicit T(int _hp) {//explicit关键字用于禁止类型转换
		hp = _hp;
}
```

当然，这里加上explicit后会提示报错，因为无法比较

**注意：**

**explicit关键字用来函数声明后，定义不需要加**

```C++
explicit operator int();//hstring转数字

explicit hstring::operator int(){}//不能过编译，提示不能使用“explicit”说明符声明“hstring::operator int”

```





#### 成员初始化列表

相比起正常初始化，效率更高，某些情况下只能使用这种方式进行初始化

```C++
//正常初始化
Role(int _lv, int _damage) {
	lv = _lv;
	damage = _damage;
	std::cout << "lv:" << lv << " damage:" <<damage << std::endl;
}

//成员初始化列表
Role(int _lv, int _damage) :lv{ _lv }, damage{ _damage } {//效率更高，某些情况下只能使用这种方式进行初始化
	std::cout << "lv:" << lv << " damage:" << damage << std::endl;
}
```

**要注意成员赋值顺序**

```C++
Role(int _lv, int _damage) :lv{ _lv }, damage{ _damage }, hp{ lv * 100 } {//要注意成员赋值顺序，lv的出现顺序需要在hp前面才能正常
	std::cout << "lv:" << lv << " damage:" << damage << std::endl;
}
```



#### 委托构造函数

先执行委托函数，再执行构造函数

```C++
Role(int _lv, int _damage):Role(_lv) {//委托构造函数
	hp = _lv * 100;
	damage = _damage;
	std::cout << "lv:" << lv << " damage:" <<damage << std::endl;
}
Role(int _lv) { lv = _lv; };
```



#### 复制构造函数

没有副本构造函数的情况下，编译器会强行添加复制构造函数，将所有值复制到新对象

```C++
Role(const Role& role) :hp{ role.hp } {//复制构造函数，不使用引用的的话会让不断调用Role(Role role)来初始化造成死循环，同类型能访问私有变量
	std::cout << "调用复制构造函数！" << std::endl;
}//因为是值传递，所以复制构造函数会生成一个对象来拷贝传入的对象，而这个过程临时生成的对象又会调用副本构造函数，就这样一直嵌套

Role userA(user);//没有复制构造函数的情况下，编译器会强行添加复制构造函数，将所有值复制到新对象
Role userA{ user };//调用复制构造函数
Role userA = user;//调用复制构造函数
userA = user;//这个等于与上面的等于不相同，因为类已经构造完了，所以不调用复制构造函数
```



### 析构函数

- 类生命周期结束时会被自动调用，一般用于扫尾工作，比如释放内存、关闭句柄等
- 如果一个类没有定义析构函数，编译器会自动添加一个空的析构函数
- 析构函数只能有一个

```C++
class Role {
	int* ary;
public:
	Role() {
		ary = new int[100];
		std::cout << "\n类被创建！";
	}

	~Role() {//析构函数
		delete[] ary;
		std::cout << "\n类被销毁！";
	}
	~Role() = default;//默认析构函数
};
```



### 类实例化

以下为类代码

```C++
class B {
public:
	B() {
		std::cout << "create B1" << std::endl;
	}
	B(int a) {
		std::cout << "create B2" << std::endl;
	}
	~B() {
		std::cout << "delete B" << std::endl;
	}
};
class A {
public:
	B b;
	int a;
	A() :b(0), a(0) {//如果没有初始化B，也会自动调用B的默认构造函数，如果b是指针，这里0等于NULL
		std::cout << "create A1" << std::endl;
	}
	A(int a) :b(0), a(0) {
		std::cout << "create A2" << std::endl;
	}
	~A() {
		std::cout << "delete A" << std::endl;
	}
	void show() {
		std::cout << "a" << std::endl;
	}
};
```

#### 栈创建

**直接调用默认构造函数**

- 默认构造函数不会被调用
- 没有传入参数，析构函数自动调用

```C++
A a;
std::cout << a.a << std::endl;
a.show();

结果：
create B2
create A1
0
a
delete A
delete B
```



**调用自定义构造函数**

- 析构函数自动调用

```C++
A a(1);//隐式调用构造函数
A b = A(2);//显式构造构造函数

结果：
create B2
create A2
create B2
create A2
delete A
delete B
delete A
delete B
```



**指针创建**

- 和上面的结果一样，b没有实例化

```C++
A a(1);
A *b = &a;

结果：
creat B2
create A2
delete A
delete B
```



#### 堆创建

- 析构函数需要通过delete手动调用

```C++
A *a=new A(1);
delete a;

结果：
creat B2
create A2
delete A
delete B
```



## 类与对象

### 静态成员变量

在类中通过static关键字声明一个类的静态成员变量，该变量特点：

1. 所有类的实例中，**共享**类中的静态成员变量，定义变量值需要放在外面

   ```C++
   class T {
   public:
       static int count;//静态变量需要放到外面单独定义，储存大小并不算入类的大小中
   };
   int T::count = 100;//定义static变量值需要放在外面
   T t1, t2, t3, t4;
   t1.count++;
   std::cout << t2.count << std::endl;//101
   std::cout << t3.count << std::endl;//101，值一样
   std::cout << &t2.count << std::endl;//00007FF7D544C000
   std::cout << &t3.count << std::endl;//00007FF7D544C000，内存地址一样
   ```

2. 类的静态成员变量在没有类的实例的情况下，依然可以访问

   ```C++
   class T {
   	static int count1;//不能直接访问
   public:
       static int count;//静态变量需要放到外面单独定义，储存大小并不算入类的大小中
       int hp;
   };
   T::count = 350;
   T t5;
   std::cout << t5.count << std::endl;//350
   std::cout << T::count << std::endl;//350
   ```

3. 类的静态成员变量并不完全属于类

   ```C++
   class T {
   public:
       static int count;//静态变量需要放到外面单独定义，储存大小并不算入类的大小中
       int hp;
   };
   std::cout << sizeof(T) << std::endl;//4
   ```

4. 在C++17新特性下，定义变量可以通过inline关键字在声明时定义

   ```C++
   class T {
   public:
       inline static int cou{150};//C++17新用法，直接定义变量
   };
   T t2;
   std::cout << t2.cou << std::endl;//150
   ```

5. const也可以直接定义，不需要C++17新特性

   ```C++
   class T {
   public:
       const static int cou2{100};//加入const关键字能直接定义
   };
   std::cout << t2.cou2 << std::endl;//100
   ```



### 静态成员函数

利用ststic关键字声明一个类的静态成员函数，该函数特点：

1. 不管有没有创建类的实例，都可以访问类的静态成员函数
2. 类的静态成员函数不能访问非静态的成员变量，因为**访问非静态的成员变量需要用到this指针，而静态成员函数不能使用this指针**
3. 类的静态成员函数不能是const
4. 类的静态成员函数不能使用this指针

```c++
class T {
    inline static int count{};
public:
    const int a;
    int b;
    static void Test() {
        std::cout << "sdfsdfsa" << std::endl;
        count++;//静态函数只能访问静态变量
        b = 100;//报错，静态成员函数不能访问非静态的成员变量
        a = 10l;//报错，是const变量
        hp++;//报错，因为hp还没内存地址
        this->hp = 1;//报错，this只能用于非静态成员函数内部
    }
    int hp;
};

const T t1;
T t2;
t1.Test();//const对象只能调用const成员函数和静态函数
T::Test();
t2.Test();//普通对象能调用const成员函数
```



### 友元类

- 友元会**破坏类的封装性**，所以建议仅仅在没有更好的选择的情况下使用友元
- 友元类**不是一种平等**的关系

**friend关键字声明**一个函数为某个类的友元函数，友元函数可以访问该类中的所有成员，包括private里的

友元函数在public里和private里都行

```c++
class T1;//声明类，不声明的话友元函数无法访问T1
class T {
	int hp;
	int mp;
	void GetHp() {
		std::cout << hp << std::endl;
	}
	void GetMp() {
		std::cout << mp << std::endl;
	}
	friend void SetMp(T& t1, T1& t2);//声明SetMp为友元函数
	friend void SetHp(T& t1, T1& t2);//声明SetHp为友元函数
};
class T1 {
	int hp;
	int mp;
	void GetHp() {
		std::cout << hp << std::endl;
	}
	void GetMp() {
		std::cout << mp << std::endl;
	}
	friend void SetMp(T& t1, T1& t2);//声明SetMp为友元函数
	friend void SetHp(T& t1, T1& t2);//声明SetHp为友元函数
};
void SetMp(T& t1,T1& t2) {//友元函数
	t1.mp = 2500;
	t1.GetMp();
}
void SetHp(T& t1, T1& t2) {////友元函数
	t1.hp = 2500;
	t1.GetHp();
}
```



friend关键字声明一个类为某个类的友元类，一个类的友元类可以访问该类中的所有成员

```c++
class T {
	int hp;
	int mp;
	void GetHp() {
		T1 t1;
		t1.GetHp();//报错，T1没有声明T是友元类
		std::cout << hp << std::endl;
	}
	void GetMp() {
		std::cout << mp << std::endl;
	}
	friend class T1;
};
class T1 {
	int hp;
	int mp;
	void GetHp() {
		T t1;
		t1.GetHp();//因为T中声明了T1是友元类，所以T1能访问T的全部成员
		std::cout << hp << std::endl;
	}
	void GetMp() {
		std::cout << mp << std::endl;
	}
};
```



### 嵌套类

- 在类中再声明一个类，被声明的类称为嵌套类，声明嵌套类的类称为外层类
- 嵌套类的声明在外层类中，因此嵌套类的作用域受外层类限定，如嵌套类放在private中类外无法使用
- 嵌套类可以访问外层类的所有成员
- 外层类仅能访问嵌套类的公有成员
- 嵌套类访问外层类成员时**变量需要是静态**，函数不需要

**嵌套类写外层**

```C++
class Role {//外层类

	static void test() {
	}

public:
	int hp;
	int mp;
	Role() {
		Weapon::test1();
	}
	class Weapon;//声明，不用将Weapon类写入Role类中，防止Role类太臃肿
};

class Role::Weapon {//嵌套类
    
	Weapon* ReturnW();//声明函数
    
public:
	Weapon();//声明函数
	short lv;
	WeaponLv wlv;
};

Role::Weapon::Weapon() {//定义函数
	test();//嵌套类函数能调用外层类的静态函数，放在私有也是可以的
	std::cout << "weapon!" << std::endl;
}
Role::Weapon* Role::Weapon::ReturnW() {//定义函数，类指针需要加上外层类
	return this;
}
```

**嵌套类写内层**

```C++
class Role {//外层类

	static void test() {

	}

public:
	int hp;
	int mp;
	Role() {
		Weapon::test1();
	}
	class Weapon {//嵌套类
		Weapon* ReturnW();//声明函数
	public:
		Weapon() {
			hp++;//报错，只能用静态成员
			test();
		}
		short lv;
		WeaponLv wlv;
		static void test1() {//要放在公共才能让外层类访问

		}
	};
	Weapon leftHands;//嵌套类需要写在外层类里才能声明成功
};

Role::Weapon* Role::Weapon::ReturnW() {//定义函数，类指针需要加上外层类
	return this;
}
```



### 局部类

定义在函数内的类称为局部类

- 局部类的定义必须写在类内
- 局部类中不允许使用静态成员变量
- 局部类可以访问全局变量

```C++
int a1;
int main() {
	int x = 250;
	class T2 {//局部类，函数定义与对象生成不能放在外面
		//static int a;//报错，不能使用静态变量
		int hp;
		int mp;
		void GetHp(){
			a1++;//能访问全局变量
			x++;//报错，有些编译器可以使用，有些不能
		}
		static int GetCount() {}//有些编译器可以使用，有些不能
	};
	T2 t1;
}
```



### malloc和new本质区别

- 对于**普通的数据类型**来说malloc和new没有什么区别
- 但是对于**类**来说，malloc**仅仅是**分配内存，而new除了**分配内存以外还会调用构造函数**

```C++
class T {
public:
	T() {
		std::cout << "第" << ++count << "个T被构造" << std::endl;
	}
};

T* t1 = (T*)malloc(10 * sizeof(T));
std::cout << t1[2].test << std::endl;//-842150451，由此可以发现对于类来说在涉及析构函数、构造函数等时melloc不适用

int* pint = (int*)malloc(10 * 4);
pint[2] = 250;
std::cout << pint[2] << std::endl;//250

T* t2 = new T[100];//构造函数成功运行
```



### free和delete本质区别

- 对于**普通的数据类型**来说free和delete没有什么区别
- 但是对于类来说，free仅仅是**释放内存空间**，而delete不仅**释放内存空间，还会调用类的析构函数**

```C++
T* t1 = (T*)malloc(10 * sizeof(T));
int* pint = (int*)malloc(10 * 4);
T* t2 = new T[100];

delete[] pint;//成功销毁
free(pint);//成功销毁
free(t1);//没有调用析构函数
delete[] t2;//成功调用析构函数
```



### delete和delete[]本质区别

- 对于**普通数据类型**来说delete和delete[]没有什么区别
- 但是对于**类**来说，delete仅仅是释放**内存空间**，且**调用第一个元素的析构函数**
- 而delete[]不仅**释放内存空间**，还会**调用每一个元素的析构函数**

```C++
T* t2 = new T[100];//构造函数成功运行
T* t3 = new T;

delete t3;//成功销毁且调用析构函数
delete[] t3;//报错，释放第2个类时析构函数报错
delete t2;//报错，且仅仅销毁一个类
delete[] t2;//成功全部销毁且调用析构函数
```



### 底层理解类

#### 栈平衡

在调用函数前，如果需要传入参数，则需要把参数入栈然后call函数，当函数执行完毕后，就需要平衡参数入栈导致的栈变化

栈平衡分为**外平栈与内平栈**

**外平栈**指调用者平衡栈，即call函数后`add esp，参数存储的大小`

**内平栈**指被调用者要结束时使用`ret 参数存储的大小`来平衡栈

#### 函数调用约定

#####  _thiscall

_thiscall是C++中类的成员函数访问时定义的函数调用约定。**类的普通成员函数默认调用。**

1. 寄存器ecx用来存放类的指针
2. 参数由右到左入栈
3. 堆栈由被调用者负责恢复
4. __thiscall可以不写出来

类中的非静态成员函数都可以使用this指针，this指针本质上来讲就是**把对象的指针通过寄存器ecx传入成员函数**，因此类中成员函数访问其成员变量时，都是通过**指针+偏移**的形式来访问的，不管**是否明确使用this指针**，即

```C++
return hp + a + b;
//等于
return this->hp + a + b;

void __thiscall BeAct1(int damage, int damage2) {
	hp = damage2 - damage;
	std::cout << "BeAct" << hp << std::endl;
}
```



##### _stdcall

__stdcall的全称是standard call。是C++的标准调用方式。**需要函数特别声明才调用。**

**非静态函数使用：**

1. 参数由右到左入栈，被调用函数参数个数是固定的，传入参数必须严格按定义个数传入
2. 若是类调用，会把对象的this指针传入eax，然后把eax压入栈的参数的最高层
3. 被调用方清理堆栈
4. 函数返回使用retn x，x为调整堆栈的字节数，即自动清理堆栈，内平栈

**静态函数使用（无法传入对象的this指针）：**

1. 参数由右到左入栈，被调用函数参数个数是固定的，传入参数必须严格按定义个数传入
2. 被调用方清理堆栈
3. 函数返回使用retn x，x为调整堆栈的字节数，即自动清理堆栈，内平栈

```C++
void _stdcall BeAct1(int damage,int damage2) {//把eax当成第3个参数
	hp = hp - damage;
	std::cout << "BeAct" << damage << std::endl;
}
```



##### _cdecl

__cdecl的全称是C Declaration，即C语言默认的函数调用方式。**普通函数（包括静态函数）默认调用。**

**类的静态成员函数，本质上是采用的_cdecl约定**

参数由右到左入栈
由调用者恢复堆栈平衡，即需要调用者根据实际传入参数的个数手动平衡堆栈
返回指令为ret

因为类的静态成员函数本质上就是**一个普通的函数**，所以根本**没有传递对象的指针**，因此，也就**不能访问其成员变量**，而类的**静态成员变量本质上相当于一个全局变量**，有着**固定的内存地址**，与类对象并无关系，所以类的静态成员可以在**类没有实例**的情况下通过 **类:静态成员** 这样的形式来访问

**非静态函数使用：**

1. 参数由右到左入栈
2. 若是类调用，会把对象的this指针传入eax，然后把eax压入栈的参数的最高层
3. 堆栈由调用方清理，即需要调用者根据实际传入参数的个数手动平衡堆栈
4. 返回指令为ret，外平栈

**静态函数使用（无法传入对象的this指针）：**

1. 参数由右到左入栈
2. 堆栈由调用方清理，即需要调用者根据实际传入参数的个数手动平衡堆栈
3. 返回指令为ret，外平栈

```c++
void _cdecl BeAct2(int damage, int damage2) {//把eax当成第3个参数，会恢复栈
	hp = hp - damage;
	std::cout << "BeAct" << damage << std::endl;
}
```



#### 类的静态成员

类的静态成员函数，本质上是采用的_cdecl约定

1. 参数由右到左入栈
2. 由调用者恢复堆栈平衡

因为类的静态成员函数本质上就是一个普通的函数，所以根本**没有传递对象的指针**。因此，也就不能访问其成员变量，而类的静态成员变量**本质上相当于一个全局变量，有着固定的内存地址**，与类对象并无关系，所以类的静态成员可以在类没有实例的情况下通过**类：：静态成员**这样的形式来访问



#### 类构造函数

```C++
class T{
	int hp;
}
```

每个类都有一个默认构造函数，如果**没有显式指定**，编译器会为我们**添加一个空的默认构造函数T(){}**

但是类T大部分的候是**没有默认构造函数**的，这是因为C/C++标准委员会要求每一个类都有默认构造函数，而一个空的构造函数实际上没有任何意义，所以**某些情况下编译器会删掉没有意义的构造函数**，这是编译器优化的功劳，原则上来讲，每一个类都有默认构造函数

若没有赋值，编译器会删除空构造函数，**有赋值会保留构造函数**，通过构造函数赋值

```C++
class T {
	int hp=1;//若没有赋值，编译器会删除空构造函数，有赋值会保留构造函数，通过构造函数赋值
	int mp;
};
```



## 运算符重载

### 定义

运算符重载让编译器对对象进行+运算时不是默认的+运算，而是进入指定函数，称为运算符重载

语法：返回类型 operator运算符()

```C++
class Person {
    unsigned short Age;
    friend bool operator<(Person& psa, Person& psb);
    friend bool operator<(Person& psa, unsigned short Age);
public:
    Person(unsigned short _Age) :Age{ _Age } {};
    unsigned GetAge() { return Age; }//就可以不用友元函数来破坏封装

    bool operator>(Person& person);//声明bool operator>(Person& person)运算符重载函数
};

//非类的成员函数实现运算符重载
bool operator<(Person& psa, Person& psb) {//运算符重载，Man<Wonman => operator<(Man,Woman)
    return psa.Age < psb.Age;
}

bool operator<(Person& psa, unsigned short _Age) {//运算符重载，Man<Wonman => operator<(Man,Woman)
    return psa.Age < _Age;
}

//报错，参数的个数是根据运算符判断的
bool operator>(Person& person) {
    return person.Age;
}

bool Person::operator>(Person& person) {//定义bool operator>(Person& person)运算符重载函数
    return Age > person.Age;
}

Person Man(120);
Person Woman(50);
if(operator<(Man,Woman))std::cout << "你找富婆了！" << std::endl;//函数在外面声明，但是是Person的友元类
if (Man < Woman)std::cout << "小于！" << std::endl;
if (Man.operator>(Woman))std::cout << "大于！" << std::endl;//函数声明在类中
if (Man > Woman)std::cout << "大于！" << std::endl;//大于是Man的函数，Woman是参数
```



### 意义

1. 让类也支持原生的运算，比如+-*/
2. 提升对程序的控制权，比如重载 new delete new[] delete[]
3. 为了让目标代码更方便使用和维护，而不是提升开发效率，重载运算符未必能提升开发效率



### 限制

1. 不能自创运算符，比如===，=<>=只能重载现有运算符
2. 以下运算符不能重载
   - 对象访问运算符.，如user.hp
   - 作用域解析运算符::，如std::cout
   - 求大小的运算符sizeof，例如sizeof(int)
   - 条件运算符?:，如b=a>c?100:200
3. 不能修改运算符本身的优先级，相关性
4. 在C++17后，也不能修改运算符的操作数的计算循序，在C++17前，编译器可以自由选择如何计算（未定义行为）
5. 除了delete/delete[]和new/new[]以外，不能对原生数据类型的其他运算符进行重载，比如把char类型的+定义为-
6. 除了new和delete以外，其他运算符的arity（运算符关联的操作数的个数或者是关联的参数）一律不能修改



### 原则

1. 不要改变运算符本身的意义，比如把假发重载为减法
2. 不建议重载逻辑运算符&&||，取址运算符&逗号运算符

备注：重载后的逻辑运算符将不会进行短路测试，在C++17标准前，编译器可以只有决定先计算左操作数还是右操作数，在C++17后计算的顺序规定为先计算左再计算右



### 正确姿势

**二元运算符的重载**

| 利用方式         | 返回类型                                             | 运算解析 |
| ---------------- | ---------------------------------------------------- | -------- |
| 利用全局函数     | 返回类型operator运算符（类型左操作数，类型右操作数） | a+b      |
| 利用类的成员函数 | 返回类型operator运算符（类型右操作数）               | a.b      |

**一元运算符的重载**

| 利用方式         | 返回类型运算解析                     | 运算解析 |
| ---------------- | ------------------------------------ | -------- |
| 利用全局函数返回 | 返回类型operator运算符（类型操作数） | ~a       |
| 利用类的成员函数 | 返回类型operator运算符（）           |          |

- 有的运算符只能重载为类的成员函数，有些运算符只能重载为全局函数
- 有些运算符既可以重载为类的成员函数又可以重载为全局函数
- 如果一个运算符既可以重载为成员函数又可以重载为全局函数，一般推荐重载为类的成员函数
- 因为类的成员函数可以是虚函数，单全局函数不能是虚函数
- 如果这个运算符不修改对象，应该将这个成员函数限定为const
- 运算符重载的参数一般可以传递值或者引用，大部分情况下，能够传递引用就不要传递值
- 对于不会修改的值最好是限定为const，某些时候要擅用使用右值引用&&作为参数
- 运算符重载的返回值一般来说可以是任何类型，但是尽量要符合运算符的原意，比如把>运算符返回指针类型，把+返回bool类型，都不是很好的选择



### 重载赋值运算符

一个类中编译器会自动添加一个默认赋值运算符重载的函数

```C++
class Role {
public:
	int hp;
	int mp;
	
    //类型后加&可以返回对象的引用，例如a=b=c需要加&
	Role& operator=(const Role& role) {//编译器默认添加关于=的运算符构造函数，这是默认生成的函数
		hp = role.hp;
		mp = role.mp;
		return *this;
	}
	Role& operator=(const Role& role) {//可以自己添加该函数，并修改
		hp = role.hp / 2;
		mp = role.mp / 2;
		return *this;
	}
}

Role x, y, z;
x.hp = 100;
x.mp = 200;
y = x;//Role& operator=(const Role& role)
z = y = x;//z.operator=(y.operator=(x))
```

**赋值运算符的重载必须用成员函数的方式实现**

```c++
Role& operator=(const Role& role)//报错，该函数必须是成员函数
```

**类中删除运算符重载**

```C++
Role& operator=(const Role& role) = delete;//删除运算符重载
```

**类外定义函数**

```C++
class Role {
public:
	int hp;
	int mp;
	Role& operator=(const Role& role);
};
Role& Role::operator=(const Role& role) {//函数类型写为void的话会让结果赋值其他对象时失败
	hp = role.hp / 2;
	mp = role.mp / 2;
	return *this;
}
```

**副本构造函数与赋值运算符重载**

```C++
char strA[]{ "aaaaaabbbbbbbbcccccccc" };
hstring str{ strA };//使用副本构造函数，因为str还没创建
str="ABC";//使用赋值运算符重载，因为str已经创建
```

**hstring**

```C++
class hstring {
    unsigned short    usmlen;//缓冲区总大小
	unsigned short    uslen;//缓冲区内存占用长度
	char* cstr;//字符串的内容
public:
	char* c_str() { return cstr; }//返回字符串
	hstring(char clen = 0x32);//初始化缓冲区
	hstring(const char* str);//利用C字符串来构造hstring
	hstring(const hstring& str);//利用hstring来构造hstring
	hstring& operator=(const hstring& str);//对于hstring重载=运算符
};

strA[0] = 0;//赋值为0后，因为\0表示ASCII码值为0的字符，所以那个位置的内容变成\0，因此后面的内容消失
std::cout << str.c_str() << std::endl;//对于cstr = (char*)str，因为str的字符串不是自己的，是strA的，所以当strA改变，str的字符串也会改变

hstring str2{ "hello!" };
str1 = str2;
std::cout << str1.c_str() << std::endl;//hello!

str1 = "12414";//不需要对于char重载=运算符，因为会使用hstring::hstring(const char* str)构造函数临时构造对象，然后传递进来，也就是完成了一次类型转换
std::cout << str1.c_str() << std::endl;//12414
```



### 重载位移运算符

<<和>>可以重载为类的方法或者是全局函数

类的成员函数：返回类型 operator>>(类型 操作数);

全局函数：返回类型 operator>>(类型 左操作数，类型 右操作数)

```C++
//实现<<添加字符串
hstring& hstring::operator<<(const hstring& str)
{
	unsigned short slen = GetLength(str.cstr);
	slen = uslen + slen - 1;
	if (slen > usmlen) {
		delete[] cstr;//删除内存
		cstr = new char[slen];//重新分配内存
		usmlen = slen;//修正内存的长度
	}
	//cstr + uslen - 1指从cstr结尾-1开始，slen - uslen + 1要加1是因为uslen里面包含了\0，需要加上才能复制到新字符串的\0
	memcpy(cstr + uslen - 1, str.cstr, slen - uslen + 1);
	uslen = slen;//字符串长度修正
	return *this;
}

//打印字符串
std::ostream& operator<<(std::ostream& _cout, hstring& _str) 
{
	_cout << _str.c_str();
	return _cout;
}

std::ostream& operator<<(std::ostream& _cout, hstring&& _str)
{
	_cout << _str.c_str();
	return _cout;
}

//输入字符串
std::istream& operator>>(std::istream& _cin, hstring& _str)
{
	char _sRead[0x1FF]{};
	_cin >> _sRead;
	_str = _sRead;
	return _cin;
}

hstring str{ "大家好啊！" };
str << hstring("@@@@@@@@@@@@@1");
str + "123" + "1234353";
std::cout << str << "111" << "22" << std::endl;//operator<<(operator<<(operator<<(std::cout, str), "111"), "222");

std::cin >> str;
std::cout << str;
```



### 重载函数运算符

()只能重载为类的方法

**方法：**返回类型 operator()(类型 操作数);

- ()重载称为functor函数对象
- ()不限制参数个数
- ()可以拥有默认实参

```C++
hstring hstring::operator()(const unsigned short start, const unsigned short length) const
{
	if (start > uslen - 2) {//不支持截取\0
		//字符串起始位置不合理
		return hstring("");
	}

	unsigned slen = start + length > uslen - 1 ? uslen - start - 1 : length;//判断截取长度是否超过最大长度
	char* newstr = new char[slen + 1];//+1是因为字符串最后面需要加上\0
	memcpy(newstr, cstr + start, slen);
	newstr[slen] = 0;
	return hstring(newstr);
}

hstring str = "a123";
std::cout << str(1, 2);
```

如果`return *this`:

```C++
//前面要加const，返回常量指针常量
const A& const operator()(const int& c) const{
	//c = 2;
	std::cout << c << std::endl;
	return *this;
}
```



### 重载二元算术运算符

二元算术运算符+-*/%可以重载为类的成员函数或者是全局函数

类的成员函数：返回类型 operator 运算符(类型 操作数);

全局函数：返回类型 operator 运算符(类型 左操作数，类型 右操作数);

```C++
hstring& hstring::operator+(int val)
{
	char      str[0x20]{};//定义一段内存
	int       len{ 0x1F };//从后往前第一个数在内存的位置
	bool      bzs = (val >= 0);//是不是正数
	val = val * (bzs * 2 - 1);//取绝对值，bzs只有2个结果，一个1，一个0，0-1=-1，1-1=0，所以乘2，0*2-1=-1，1*2-1=1
	do str[--len] = val % 10 + 48; while (val = val / 10);
	str[len = len - (1 - bzs)] = '-' * (bzs + 1) * (1 - bzs) + str[len] * bzs;
	
	unsigned short slen = uslen + 0x20 - len - 1;

	if (slen > usmlen) {
		char* lstr = cstr;
		cstr = new char[slen];//重新分配内存
		usmlen = slen;//修正内存的长度
		memcpy(cstr, lstr, uslen);
		delete[] lstr;//删除内存
	}
	//cstr + uslen - 1指从cstr结尾-1开始，slen - uslen + 1要加1是因为uslen里面包含了\0，需要加上才能复制到新字符串的\0
	memcpy(cstr + uslen - 1, str + len, 0x20 - len);
	uslen = slen;//字符串长度修正

	return *this;
}

hstring str{ "123abc" };
str = str + -123456 + 123534;
std::cout << str;
```



### 重载类型转换运算符

1. 类型转换运算符只能**重载为类的成员函数**
2. 类型转换运算符**没有返回值**，返回值由类型转换的类型来决定
3. 语法：operator 类型() const

```C++
hstring::operator int()
{
	int val{ 0 };
	int i = (cstr[0] == '-');//标尺，有-为1，无则0，跳过负号
	while (cstr[i] >= '0' && cstr[i] <= '9') {
		val = val * 10 + cstr[i++] - 48;
	}
	val = val * ((cstr[0] != '-') * 2 - 1);//如果是负数则变成负数
	return val;
}
```

**注意：**

**（1）对于以下情况**

```C++
hstring str{ 123 };
int x = str + 123;
```

有2种转法：

- 第一种123转化为hstring与str相加，以下为转换翻译

  ```C++
  int x = int(str + hstring(123));
  ```

  这种情况需要对**operator int()函数的声明**加上explicit，禁止operator int()函数的隐式转换

- 第二种str转化为int变为123后与123相加得到246，以下为转换翻译

  ```C++
  int x = int(str) + 123;
  ```

  这种情况在上一种情况下只需要对str加上int()转型就行了，跟上面这条代码一样

**（2）对于if(str)**

- 可以使用重载void*()

  ```C++
  hstring::operator void*(){
  	return cstr;
  }
  ```

  这样就能判断了，但是这样写就会暴露cstr的值

- 也可以使用重载bool

  ```C++
  hstring::operator bool()
  {
  	return cstr != nullptr;
  }
  
  //重载hstring类==nullptr
  bool hstring::operator== (std::nullptr_t nptr) {
  	return cstr == nptr;
  }
  ```

  这个方法对于以下代码

  ```C++
  hstring strls{ (char)0 };
  short mon = strls;//执行了hstring::operator bool()，因为strls是0，被强转short，类型提升
  std::cout << mon << std::endl;//0
  ```
  
  - 因为此时的strls的cstr是0，在strls等于mon时会经过operator bool()的转换，有类型提升
  - 如果有operator int()函数会报错，因为有多个隐式转换，如果其中一个加上explicit就可以转换到另一个



### 重载递增递减运算符

**前置**++/--可以重载为类的方法或者是全局函数

方法

返回类型 operator++();
返回类型 operator--();

全局函数

返回类型 operator++(操作数类型 操作数);
返回类型 operator--(操作数类型 操作数);



**后置**++/--可以重载为类的方法或者是全局函数**（int一定要加，其他数据类型都不行）**

方法

返回类型 operator++(int);
返回类型 operator--(int);

全局函数

返回类型 operator++(操作数类型 操作数,int);
返回类型 operator--(操作数类型 操作数,int);



**方法**

```C++
hint& hint::operator++() {//这里返回引用是为了让后续能对该对象进行操作，如++a+1，不返回则无法+1
	*this = *this + 1;
	return *this;
}
hint& operator--(hint& val) {//这里用引用的原因和上面一样
	val = val - 1;
	return val;
}
```

**全局**

```C++
const hint operator--(hint& val, int)
{
	hint _val = val;
	--val;
	return _val;
}

const hint hint::operator++(int) {
	hint val = *this;//报错，因为调用了默认复制构造函数hint::hint(hint& val)，而该函数默认为空，且结束时会调用一次析构函数，最终结束时又会调用一次析构函数导致内存出错
	//hint val = 100;//报错，相比起上一条是100先转化为hint类调用了默认复制构造函数hint::hint(hint& val)
	*this = *this + 1;
	return val;
}

解决办法：
hint::hint(hint& val):hint((int)val) {//在复制构造函数之前运行构造函数
}
```



### 重载内存分配

- 重载new就要重载delete，不然会让系统自带的对接造成出错
- 要重载就要一起重载
- noexcept修饰的函数不会抛出异常
- new的函数都为静态函数，因为内存还没分配没有this指针
- delete的函数都为静态函数，因为已经调用了析构函数没有空间了，也就没有了this指针
- 定义了`void* operator new (size_t size)`或`void operator delete (void* adr) noexcept`后其他版本永不调用

#### new

**工作原理**

Role* user=new Role();

1. 通过new分配内存空间
2. 调用构造函数
3. 返回指针
4. 对象结束时不会自动diao'yong

**new六种声明形式**

- `void* operator new (size_t size);`**（定义这个版本后其他版本永不调用）**

- `void* operator new[] (size_t size);`

- `void* operator new(size_t size,const std::nothrow_t&) noexcept;`

- `void* operator new[](size_t size,const std::nothrow_t&) noexcept;`

  **禁止重载两种形式**

- `void* operator new(size_t size,void* p) noexcept;`
- `void* operator new[](size_t size,void* p) noexcept;`

```C++
void* blut::operator new(size_t size) {//size为要分配的内存空间，这里表示为类的大小，静态函数，因为内存还没分配没有this指针
	//return new blut;//会造成死循环
	//return ::operator new (size);//::表示全局命名空间，operator new 表示全局new
	//return mem;//分配到指定位置上
	blut* dat = (blut*)mem;
	for (int i = 0; i < 1000; i++) {
		if (!dat[i].bUse)return &dat[i];
	}
}

void* blut::operator new(size_t size, const char* txt) {
	blut* dat = (blut*)mem;
	for (int i = 0; i < 1000; i++) {
		if (!dat[i].bUse)return &dat[i];
	}
}
```



#### delete

**工作原理**

delete user;

1. 调用析构函数
2. 通过delete释放内存

**delete六种形式**

- `void operator delete (void* adr) noexcept;`**（定义这个版本后其他版本永不调用）**
- `void operator delete[] (void* adr) noexcept;`
- `void operator delete(void* adr,const std::nothrow_t&) noexcept;`
- `void operator delete[](void* adr,const std::nothrow_t&) noexcept;`
- `void operator delete(void* adr,void* p)noexcept;`
- `void operator delete[](void* adr,void* p)noexcept;`

```C++
void blut::operator delete(void* adr)noexcept {//adr表示释放的指针，静态函数，因为已经调用了析构函数没有空间了，也就没有了this指针
	
}

void blut::operator delete(void* adr, size_t size)noexcept {
	
}
```



#### 内存管理

```C++
class blut {
	bool bUse = true;//此个对象块被使用，只是提示
public:
	void* operator new(size_t size);//重载new
	void operator delete(void* adr)noexcept;//重载delete，要和new一起重载
	
	~blut();//此个对象块被释放，只是提示
}

char* mem = new char[1000 * sizeof(blut)] {};//申请1000个blut类
void* blut::operator new(size_t size) {//重载new，将内存划在mem中
	blut* dat = (blut*)mem;
	for (int i = 0; i < 1000; i++) {
		if (!dat[i].bUse)return &dat[i];
	}
}
void blut::operator delete(void* adr)noexcept {}//重载delete
blut::~blut() {//将当前内存划为被释放，只是提示
	bUse = false;
}

blut* shota1 = new blut;
blut* shota2 = new blut;
blut* shota3 = new blut;
blut* shota4 = new blut;
blut* shota5 = new blut;
blut* shota6 = new blut;
std::cout << (void*)mem << std::endl;//000001A2046A2010
std::cout << shota1 << std::endl;//000001A2046A2010
std::cout << shota2 << std::endl;//000001A2046A2024
std::cout << shota3 << std::endl;//000001A2046A2038
std::cout << shota4 << std::endl;//000001A2046A204C
std::cout << shota5 << std::endl;//000001A2046A2060
std::cout << shota6 << std::endl;//000001A2046A2074，从上得知，相差0x14字节

blut* shota1 = new blut;
blut* shota2 = new blut;
blut* shota3 = new blut;
delete shota1;
blut* shota4 = new blut;
blut* shota5 = new blut;
blut* shota6 = new blut;
std::cout << (void*)mem << std::endl;//000001A2046A2010
std::cout << shota1 << std::endl;//000001A2046A2010
std::cout << shota2 << std::endl;//000001A2046A2024
std::cout << shota3 << std::endl;//000001A2046A2038
std::cout << shota4 << std::endl;//000001A2046A2010，分配到了shota1的地址
std::cout << shota5 << std::endl;//000001A2046A204C
std::cout << shota6 << std::endl;//000001A2046A2060
```



### 重载数据类型

```C++
//第一个const禁止修改指针的指向（常量指针），第二个const让operator函数变为常函数，不能修改值
operator const sockaddr* () const {
	return (sockaddr*)&m_addr;
}
```



## 继承

### 概念

- 类monkey基于类animal被创造出来，那么**类animal是类monkey的基类(父类)**，**类monkey是类animal的派生类(子类)**，子类自动继承包含**父类的成员变量以及成员函数**
- 子类不能继承父类的**构造函数，析构函数，重载赋值运算符**，虽然不能继承，但是这些内容还是存在于父类中
- 基于类monkey创造的新类bigMonkey，类monkey是类bigMonkey的**直接基类**，类animal是类bigMonkey的**间接基类**

```C++
class Xobject {
public:
    char Name[0x20];
};

class object {
public:
    char Name[0x20];
};

class mapObject:public object {//继承object
public:
    int x;
    int y;
protected://外界用不了，只能让继承类使用
    int t;
private://外界与继承类都使用不了
    int a;
};

class MoveObject :public mapObject {//继承mapObject
public:
    int hp;
    int lv;
    
    Xobject X;//包含关系

    MoveObject() {  
        t = 100;//访问父类的protected属性变量
    }
};
```



### public、protected、private

- **在单个类里**

  public：全都可以访问

  protected：外界不能访问，继承类才使用

  private：外界与继承类都使用不了，**不添加属性的话是private**

  ```C++
  class mapObject:public object {//继承object
  public:
      int x;
      int y;
  protected://外界用不了，只能让继承类使用
      int t;
  private://外界与继承类都使用不了
      int a;
  };
  ```

- **继承类时**

  public：子类继承父类时父类属性不变

  protected：子类继承父类时父类public属性变protected，private不变

  private：子类继承父类时父类public、protected属性变private

**继承后的成员属性**

![image-20221108192751645](https://raw.githubusercontent.com/being1752/image/main/img/image-20221108192751645.png)

如果想修改继承类的protected变量，可以通过下面方式

```C++
class mapObject {
protected://外界用不了，只能让继承类使用
    int t;
    int getHP(int x2) {
        x = 2;
    }
private:
    int a;
};

class MoveObject : public mapObject {//继承mapObject
public:
    using mapObject::t;//将继承的protected属性变为public
    using mapObject::a;//报错，a是private属性
    using mapObject::getHP;//修改函数属性不需要后面的参数
};

MoveObject npc;
npc.t;
```

但是这种方式会破坏类的封装



**设置访问属性好处**

- 可以更好的封装父类成员
- 可以在子类作为基础进行派生类的时候提供继承控制
- 将保护属性为private的父类产生的子类作为基类的时候，**派生类继承但不能访问构建其父类的基类成员**
- 将保护属性为protected的父类产生的子类作为基类的时候，**派生类继承且可以访问其非私有成员**



**选择**

一般来说，**尽量设计类的成员变量为private**，如果需要访问这些成员变量，**应该提供setter以及getter函数**

```C++
class object {
public:
    int getMapId() { return mapId; }
    void setMapId(int val) { mapId = val; }
private:
    int mapId;
};
```



### 继承中的构造函数

**对于继承类**

```C++
class object {
public:
    int mapId;
    object() {
        std::cout << "object was created" << std::endl;
    }
};

class mapObject :public object {
public:
    int mapId;
    mapObject() {
        std::cout << "mapObject was created" << std::endl;
    }
};

class actObject :public mapObject {
public:
    int damage;
    actObject() {
        std::cout<<"actObject was created" << std::endl;
    }
};

 actObject obj;//先构造object，再构造mapObject，最后构造actObject
```

最终actObject类初始化对象会先构造object，再构造mapObject，最后构造actObject

**对于对象赋值**

```C++
class object {
public:
    int mapId;
    object() {
        std::cout << "object was created" << std::endl;
    }
    object(const object& obj) {
        std::cout << "object was created by copy" << std::endl;
    }
};

class mapObject :public object {
public:
    int mapId;
    mapObject() {
        std::cout << "mapObject was created" << std::endl;
    }
    mapObject(const mapObject& obj) {
        std::cout << "mapObject was created by copy" << std::endl;
    }
};

class actObject :public mapObject {
public:
    int damage;
    actObject() :mapObject{ 100 } {
        std::cout<<"actObject was created" << std::endl;
    }
    actObject(const actObject& obj)  {
        std::cout<<"actObject was created by copy" << std::endl;
    }
};

int main()
{
    actObject obj;
    actObject obj1 = obj;//先构造object，再构造mapObject，最后运行actObject副本构造函数
```

最终复制的对象会先构造object，再构造mapObject，最后运行actObject副本构造函数

**若要规定运行的函数**

```C++
 mapObject(int id) :mapId{ id } {//在mapObject类

 }

//在actObject类
actObject() :mapObject{ 100 } {//规定运行mapObject(int id) :mapId{ id }函数，mapId要放在函数内使用
   // actObject() : mapId{ 100 } {//父类的mapId要放在函数内使用
   std::cout<<"actObject was created" << std::endl;
}
```

**若想要调用父类的构造函数**

```C++
//mapObject
mapObject(int id) :mapId{ id } {
        std::cout << id << std::endl;
}

//actObject
using mapObject::mapObject;

actObject obj{10};//10
```

- 构造函数的属性是根据传过去的属性的
- 默认构造函数，副本构造函数等函数是无法传过去，自定义的构造函数才能传过去
- 如果子类有定义相同类型的构造函数，父类的就无法传过去



### 继承中的析构函数

```C++
class object {
public:
    int x;
    int y;
    ~object() {
        std::cout << "object was -" << std::endl;
    }
};
class mapObject :public object {
public:
    int mapId;
    ~mapObject() {
        std::cout << "mapObject was -" << std::endl;
    }
};
class actObject :public mapObject {
public:
    int damage;
    ~actObject() {
        std::cout << "actObject was -" << std::endl;
    }
};

actObject MAP;
```

析构顺序为actObject->mapObject->object



### 成员重复

指父类与子类有相同成员

```C++
class object {
public:
    int x;
};
class mapObject :public object {
public:
    int x;
    mapObject() {
        x = 2500;
        object::x = 3500;//父类的x，需要函数内访问
    }
};

mapObject MAP;
MAP.x = 250;//访问子类的x
MAP.object::x=20;//访问父类的x
```

**对于不同参数的函数**

```C++
class object {
public:
    int x;
    void Show() {
        std::cout << x << std::endl;
    }
};
class mapObject :public object {
public:
    int x;
    mapObject() {
        x = 2500;
        object::x = 3500;//父类的x，只能函数访问
    }
    void Show(int p) {//与父类的Show函数重名，继承父类的Show函数被覆盖
        std::cout << x << std::endl;
    }
    using object::Show;//引入父类的Show函数形成重载
};
```

**对于相同参数的函数**

```C++
class object {
public:
    void Show(int p) {
        std::cout << x << std::endl;
    }
};
class mapObject :public object {
public:
    void Show(int p) {//与父类的Show函数重名，继承父类的Show函数被覆盖
        std::cout << x << std::endl;
    }
    using object::Show;//引入父类的Show函数
};

mapObject MAP;
MAP.Show(150);//访问子类的show函数
MAP.object::Show(2500);//访问父类的show函数
```

**对于三层继承类**

```C++
obj.Show(222);//第三层
obj.mapObject::Show(22);//第二层
obj.mapObject::object::Show(2);//第一层
obj.objdec::show(2);//第一层
```



### 多重继承

**函数继承**

```C++
class Wolf {
public:
    void eat() {
        std::cout << "Wolf eat!" << std::endl;
    }
    void bite() {
        std::cout << "Wolf Bite!" << std::endl;
    }
};
class Man {
public:
    void eat() {
        std::cout << "Man eat!" << std::endl;
    }
};
class WolfMan :public Wolf, public Man {//多重继承
    
};

WolfMan Jack;
Jack.eat();//报错，冲突
```

Wolf的eat函数和Man的eat函数冲突，解决办法：

```C++
Jack.Wolf::eat();//指定Wolf的eat函数
Jack.Man::eat();//指定Man的eat函数
```

或

```C++
class WolfMan :public Wolf, public Man {//多重继承
    Wolf::eat();
};
```



**变量继承**

```C++
class MoveObject {
public:
    int x;
    int y;
};
class Wolf:public MoveObject {

};
class Man :public MoveObject {

};
class WolfMan :public Wolf, public Man {//多重继承

};

Jack.x = 250;//参数冲突
Jack.y = 250;//参数冲突

//解决
Jack.Wolf::x = 250;
Jack.Man::y = 250;
```



## 多态

多态是指一个事物表达出多种形态
在C++中多态是指一个函数的多种形态

### 对象多态

- 向下转型  父类->子类，虽然能调用派生类的函数，但是**派生类的变量没有初始化**
- 向上转型  子类->父类，安全



#### 向上转型与向下转型

- 使用对象直接向上转型会导致**内存切片**
- 使用对象直接向下转型会报错，因为Human类相比Animal有更多内容，大转小的话会访问不属于Human类的内存空间
- 可以通过**指针或引用**表达派生类中**基类的特征**
- 通过指针或引用向下转型，存在访问**不属于基类的内存地址**的风险，最好用来**访问不涉及子类的变量**访问的函数
- 通过指针或引用向下转型能调用派生类的函数，但是**派生类的变量都是未初始化**的，因为变量的内存地址不属于该基类
- 对于向下转型时使用dynamic_cast，需要有虚函数**表示多态**才不报错，若转换的类指针**指向基类**则返回nullptr表示转换失败，若转换的类指针**指向派生类**则返回地址表示转换成功，原因是因为基类转换，虽然不报错，但是有风险，基类转派生类有内存泄露的可能

```C++
class Animal {
public:
    int Age;
};
class Human :public Animal {
public:
    int Money;
    void doit() {
        std::cout << "Do it!" << std::endl;
    }
};

Human laow;
laow.Age = 350;
laow.Money = 250;
Animal anm = laow;//内存切片，子类->父类，向上转型
Human laowan = anm;//报错，父类->子类，向下转型，因为Human类相比Animal有更多内容，多转小的话会访问不属于Human类的内存空间

Animal* anm = &laow;//可以通过指针或引用表达laow中Animal的特征
Human* hman = (Human*)anm;//父类->子类，存在访问不属于该对象的内存地址的风险，最好用来访问不涉及子类的变量访问的函数
hman->doit();//向下转型调用子类函数，能正常调用

Animal hu;
Human laow1;
Animal& anm1 = laow1;//引用转换
Human& lao = dynamic_cast<Human&>(hu);//抛出异常，但是可以过编译，指针可以有空指针，引用不能有空引用，此时因为基类转派生类导致转换失败返回空指针，所以异常

Human* hman1 = (Human*)(new Animal);//强制转换
std::cout << hman1->Money << std::endl;//-33686019，基类强制转换派生类的话，虽然能调用派生类的函数，但是派生类的变量没有初始化
```



#### **类型转换**

**强制转换与隐式转换**

- 对于继承的基类，如果是通过private或虚基类的方式继承，无法进行强制转换或隐式转换
- 如果一个基类指针原本指向派生类，然后让派生类的类指针指向这个基类指针，编译器会报错，因为编译器不知道，向下转型不允许
- 如果是通过强制转换的方式让上面的基类指针指向派生类的类指针就不报错，因为经过思考，而不是不小心

```C++
class MoveObject{};
class MonsterObject:MoveObject{};
class MObject:public virtual MoveObject{};//虚基类

MonsterObject monster;
MoveObject* _move = &monster;//隐式类型转换，向上转型，若基类是private的话会报错
MoveObject* _move = (MoveObject*)&monster;//强制转换
MonsterObject* _pmonster = _move;//报错，虽然_move指向的对象原本就是MonsterObject类，但是编译器不知道，向下转型不允许

MonsterObject* _pmonster = (MonsterObject*)_move;//强制类型转换，因为经过思考，不是不小心
MonsterObject* _pmonsterA = static_cast<MonsterObject*>(_move);//若基类是private或基类是虚基类的话会报错

MonsterObject* _pmonsterB = (MObject*)_move;//强制转型报错，类是虚基类
MonsterObject* _pmonsterC = static_cast<MObject*>(_move);//显式转型报错，类是虚基类
```



**类指针之间转换**

- 类指针与类指针之间的类型转换才能转换信息
- 类与空指针之间的类型转换会让内存信息丢失，只能转内存地址

```C++
class WOLF :public MonsterObject {};
class MAN :public MonsterObject {};
class WOLFMAN :public WOLF,public MAN {};

WOLFMAN wolfMan;

void* ptr = &wolfMan;//WOLFMAN* -> void*，发生类型转换，变为空指针，内存信息丢失，只剩下内存地址

WOLF* pWlfA = ptr;//报错，类型丢失

WOLF* pWlf = (WOLF*)ptr;//void* -> WOLF*，因为空指针内存信息丢失，只能转内存
MAN* pMan = (MAN*)ptr;//void* -> MAN*，因为空指针内存信息丢失，只能转内存

WOLF* pWlf = &wolfMan;//WOLFMAN* -> WOLF*，能完整转换信息
MAN* pMan = &wolfMan;//WOLFMAN* -> MAN*，能完整转换信息

std::cout << ptr << " " << pWlf << " " << pMan << std::endl;//003CF9D3 003CF9D3 003CF9D4
```



**多重继承转换**

多重继承不能转换，因为有多个MoveObject类，编译器不知道转哪一个

```C++
class MoveObject{};
class MonsterObject:public MoveObject{};
class WOLF :public MonsterObject {};
class MAN :public MonsterObject {};
class WOLFMAN :public WOLF,public MAN {};

WOLFMAN wolfMan;
MoveObject* _move;

_move = (WOLFMAN*)&wolfMan;//多重继承不能转换
_move = (MoveObject*)&wolfMan;//多重继承不能转换
_move = static_cast<MoveObject*>(&wolfMan);//多重继承不能转换，因为有多个MoveObject类，编译器不知道转哪一个
```

以下为可以转换的方式，但是此时wolfMan指向的是第二个MoveObject类，而不是WOLF类里的MoveObject类，因为相对MoveObject类相对WOLF类是父类

```C++
class MoveObject{
    int x;
};
class MonsterObject:public MoveObject{};
class WOLF :public MonsterObject {};
class MAN :public MonsterObject {};
class WOLFMAN :public WOLF,public MoveObject {};

WOLFMAN wolfMan;
MoveObject* _move;

_move = (WOLFMAN*)&wolfMan;//不报错
_move = (MoveObject*)&wolfMan;//不报错
_move = static_cast<MoveObject*>(&wolfMan);//不报错，此时wolfMan指向的是WOLF类里的MoveObject类，而不是第二个MoveObject类

wolfMan.::MoveObject::x = 3500;//赋值成功
wolfMan.::WOLF::MonsterObject::MoveObject::x = 2500;//赋值失败

std::cout << _move->x << std::endl;//3500，即WOLFMAN类继承的类中相对底层的父类赋值能成功，也就是wolfMan类指向第二个MoveObject类
```



#### **static_cast**

**语法：**static_cast< new_type >(expression)

在编译的时候进行检查

1. 用于基类和派生类之间通过指针或引用的转换
   - 向上转型，即派生类转基类时，是安全的，因为内存切片，并且传递一个内存地址即可
   - 向下转型，即基类转派生类时，因为没有动态类型检查，是不安全的
2. 用于基本数据类型之间的转换，如int转char，int转enum
3. 把空指针转目标类型的空指针
4. 把任何类型的指针转换成void类型

**注意：**static_cast不能转换掉expression的const、volatile、或者__unaligned属性

**static_cast在类中不允许类型转换的情况**

- 通过private继承基类
- 通过虚基类继承基类



#### **dynamic_cast**

语法：dynamic_cast<目标类型>();

在运行时进行检查

1. dynamic_cast只能用于支持**方法多态类型**的指针，如果转换成功，返回指针，转换失败返回nullptr
2. 一般动态转换分为两种情况：
   - 向下转换 downcast
   - 跨类转换 crosscast
3. 不要转换this指针
4. 当dynamic_cast用于转换引用时，转换失败将会抛出异常，一般不推荐转换引用

**向下转换**

- 对于向下转型时使用static_cast，虽然不报错，但是有风险，尽管能调用派生类的函数，但是派生类的变量没有初始化，因为基类转派生类有内存泄露的可能
- 对于向下转型时使用dynamic_cast，需要有虚函数**表示多态**才不报错，若转换的类指针**指向基类**则返回nullptr表示转换失败，若转换的类指针**指向派生类**则返回地址表示转换成功，原因是因为基类转换，虽然不报错，但是有风险，基类转派生类有内存泄露的可能
- 如果使用引用接收dynamic_cast的转换结果，指针可以有空指针，引用不能有空引用，如果是向下转型，则会返回nullptr导致引用变为空引用，然后抛出异常

```C++
class MoveObject {
public:
    virtual void test(){}
};
class MonsterObject :public MoveObject {};
class NPCObject :public MoveObject {};

MonsterObject monster;
MoveObject* _move = (MoveObject*)&monster;//强制转换

_move = new MoveObject();
MonsterObject* _pmonsterA = static_cast<MonsterObject*>(_move);//静态类型转换，如果从基类MoveObject转换，虽然不报错，但是有风险，虽然能调用派生类的函数，但是派生类的变量没有初始化，因为基类转派生类有内存泄露的可能

MonsterObject* _pmonsterB=dynamic_cast<MonsterObject*>(_move);//需要有虚函数表示多态才不报错，若_move指向基类MoveObject则返回nullptr表示转换失败，若_move指向派生类MonsterObject则返回地址表示转换成功，原因还是因为基类MoveObject转换，虽然不报错，但是有风险，因为基类转派生类有内存泄露的可能

NPCObject& lNPC = dynamic_cast<NPCObject&>(monster);//抛出异常，但是可以过编译，指针可以有空指针，引用不能有空引用，此时因为基类转派生类导致转换失败返回空指针，所以异常
```

**正确使用（案例的类的情况与上面一样）**

- **new：**基类指针指向需要转换的派生类，然后就能将该基类指针转化为派生类

  ```C++
  MoveObject* _move = new MonsterObject();
  MonsterObject* _pmonsterB = dynamic_cast<MonsterObject*>(_move);
  ```

- **派生类对象转换：**派生类对象经过强制转换为基类后，将转换类型设置为该派生类就能重新转换回来

  ```C++
  MonsterObject monster;
  MoveObject* _move = (MoveObject*)&monster;//强制转换
  MonsterObject* _pmonsterB = dynamic_cast<MonsterObject*>(_move);
  ```



**跨类转换**

- 转换是否成功取决于pMove是否指向WOLFMAN，因为WOLFMAN类继承WOLF类和BOSS类
- 如果转换没有继承的类会返回空指针

```C++
class MoveObject {};
class MonsterObject :public MoveObject {};
class WOLF :public MonsterObject {};
class BOSS {};
class WOLFMAN :public WOLF, public BOSS {};

WOLFMAN wlfMan;
MoveObject* pMove = &wlfMan;
auto p = dynamic_cast<WOLFMAN*>(pMove);//转换是否成功取决于pMove是否指向WOLFMAN
std::cout << p << std::endl;
auto b = dynamic_cast<BOSS*>(pMove);//跨类型转换，WOLFMAN没有继承BOSS的话返回空指针
std::cout << b << std::endl;
```

**转换this指针**

不违反语法规定，但是逻辑非常依赖派生类，非常不推荐

```C++
class MoveObject {
public:
    virtual void move() {
        if(dynamic_cast<MonsterObject*>(this));//不违反语法规定，但是逻辑非常依赖派生类，非常不推荐
    }
};
class MonsterObject :public MoveObject {};
```



**过度使用dynamic_cast情况**

写法没错，但是没有利用到多态的特性，过度使用dynamic_cast

```C++
class MoveObject {
public:
    virtual void test(){}
};
class MonsterObject :public MoveObject {
public:
    void MonsterMove() {};
};
class NPCObject :public MoveObject {
public:
    void NPCMove() {};
};

MonsterObject monster;
MoveObject* _move = (MoveObject*)&monster;

//写法没错，但是没有利用到多态的特性，过度使用dynamic_cast
auto _pmonsterC=dynamic_cast<MonsterObject*>(_move);
auto _pNPCA = dynamic_cast<NPCObject*>(_move);
if (_pmonsterC != nullptr)_pmonsterC->MonsterMove();
if (_pNPCA != nullptr)_pNPCA->NPCMove();
```

**正确用法**

使用多态，在基类MoveObject添加move虚函数，派生类实现move函数，然后多态调用

```C++
class MoveObject {
public:
    virtual void move() {}
};
class MonsterObject :public MoveObject {
public:
    void move() {
        std::cout << "MonsterObject" << std::endl;
    }
};
class NPCObject :public MoveObject {
public:
    void move() {
        std::cout << "NPCObject" << std::endl;
    }
};

//使用多态，在基类MoveObject添加move虚函数，派生类实现move函数，然后多态调用
MonsterObject monster;
MoveObject* _move = (MoveObject*)&monster;
_move->move();//MonsterObject

NPCObject Npc;
_move = (MoveObject*)&Npc;
_move->move();//NPCObject
```







### 方法多态

**静态多态**

- 函数重载
- 函数模板

```C++
class Animal {
public:
    int Age;
};
class Human :public Animal {
public:
    int Money;
};

void BeAct(Human* R) {

}
void BeAct(Animal* anm) {
    
}

Human laowa;
Animal dog;

//静态多态，通过重载或模板实现，根据参数实现调用哪一个函数
BeAct(&laowa);
BeAct(&dog);
```



**动态多态**

- 需要加上virtual才能动态多态调用，不加上的话永远调用指针指向的类的beAct函数
- 派生类的函数不加virtual也能实现，因为指针bA原本就是基类，基类的beAct函数是虚函数

```C++
class Animal {
public:
    int Age;
    void virtual beAct(Animal* anm) {//需要加上virtual才能动态多态调用，不加上的话永远调用指针指向的类的beAct函数
        std::cout << "动物被攻击！" << std::endl;
    }
};
class Human :public Animal {
public:
    int Money;
    void virtual beAct(Animal* anm) {//需要加上virtual才能实现，但是这里不加virtual也能实现，因为指针bA原本就是Animal类，Animal类的beAct函数是虚函数
        std::cout << "人被攻击！" << std::endl;
    }
};

Human laowa;
Animal dog;

int id;
Animal* bA;//要实现动态多态，必须使用基类指针
std::cin >> id;
//动态多态
if (id) {//通过id输入来决定Animal类指针bA指向什么类
    bA = &dog;
}
else bA = &laowa;

bA->beAct(&dog);
```



### 虚函数

基类设置虚函数，通过基类指针指向不同派生类实现同一函数不同操作，就算派生类之间互为父子类也能实现

**注意：**

- 一个函数**不能既是虚函数，又是模板函数**
- 一个模板类可以有虚函数

#### 原理

在多态类中，第一个存放的地址是该多态类虚函数表的地址，然后才是变量。该多态类虚函数表里又存放了虚函数的地址。
所以调用虚函数的路径是先访问多态类虚函数表查找目标虚函数地址，然后根据该地址调用该函数。

```C++
class AIM {
public:
    //[unknown]
    int HP;//+0x0
    virtual void Eat() {
        std::cout << "AIM" << std::endl;
    }
    virtual void Die() {
        std::cout << "AIM-DIE" << std::endl;
    }
};
class WOLF :public AIM {
public:
    virtual void Eat() {
        std::cout << "WOLF" << std::endl;
    }
    virtual void Die() {
        std::cout << "WOLF-DIE" << std::endl;
    }
    void Sound() {
        std::cout << "aoaoaoao~" << std::endl;
    }
};

AIM* wolf = new WOLF();
wolf->Die();//WOLF-DIE

std::cout << sizeof(AIM) << std::endl;//8
std::cout << wolf << " " << &wolf->HP << std::endl;//在X86下，00C7F4A0 00C7F4A4，类地址与变量地址之间相差4字节，由此可以得知之间存在一个指针

unsigned* vtable = (unsigned*)wolf;//该值是wolf内存地址存放的值，也就是VTABLE的内存地址
std::cout << std::hex << "VTABLE:" << vtable[0] << std::endl;//VTABLE:f1ab60，该值是wolf内存地址存放的值，也就是VTABLE的内存地址
unsigned* func = (unsigned*)vtable[0];//虚表第一位的值

std::cout << std::hex << "eat:" << func[0] << std::endl;//eat:f110e1
std::cout << std::hex << "die:" << func[1] << std::endl;//die:f11163
```



#### 虚表性质

1. 同一个类的多个实例都指向同一个虚函数表

2. 通过修改虚函数表的数据可以实现劫持

   ```C++
   void Hack() {//攻击函数
       std::cout << "sadfsaf" << std::endl;
   }
   
   unsigned* vtable = (unsigned*)wolf;//该值是wolf内存地址存放的值，也就是VTABLE的内存地址
   unsigned* func = (unsigned*)vtable[0];//虚表第一位的值
   
   //解除修改限制
   DWORD old;
   VirtualProtect(func, 8, PAGE_EXECUTE_READWRITE, &old);
   
   //修改表值
   func[0] = (unsigned)Hack;
   func[1] = (unsigned)Hack;
   func[2] = (unsigned)Hack;
   ```

3. 只有通过指针访问函数才会调用虚函数表

   ```C++
   WOLF aWolf;//初始化对象
   aWolf.Sound();//该函数是普通函数
   aWolf.Eat();//该虚函数没有被上面的劫持所更改
   ```



#### virtual

- virtual告诉派生类该函数是虚的，**可以被重写覆盖**

  ```C++
  virtual MoveObject* Move() {//virtual告诉派生类该函数是虚的，可以被派生类重写覆盖
      x++;
      y++;
      return this;
  }
  ```

- virtual只能写在**类的内部声明或者定义**，不能把virtual写在**类的外部定义中**

  ```C++
  class MoveObject {
  public:
      void virtual Move() {//在内部定义，可以内部virtual声明外部定义
      }
  };
  void virtual MoveObject::Move() {}//报错，不能定义在外部
  ```

- 调用**类的对象**是无法使用虚函数的，必须使用基类指针来实现，**用对象会造成内存切片**

  ```C++
  void Move(MoveObject* obj) {//要实现动态多态，必须使用基类指针，引用也可以
      obj->Move();
  }
  ```

- 虚函数的调用虚函数在**派生类和基类**中必须具有**相同的及参数列表**

- 虚函数在派生类和基类中返回值要求基本一致，但是当返回类型为类类型的指针和引用时除外

  ```C++
  //例外情况
  class MoveObject {
  public:
      virtual MoveObject* Move() {
          return this;
      }//返回MoveObject类指针
  };
  class NPCObject :public MoveObject {
  public:
      virtual NPCObject* Move() {
          return this;
      }//返回NPCObject类指针
  };
  class MonsterObject :public MoveObject {
  public:
      virtual MonsterObject* Move() {
          return this;
      }//返回MonsterObject类指针
  };
  //MoveObject和NPCObject是父子关系，所以NPCObject的Move函数能返回MoveObject类的指针，而MonsterObject和NPCObject是兄弟关系，所以NPCObject的Move函数不能返回MonsterObject类的指针
  ```

- 虚函数不能是函数模板

- virtual不能定义变量

- 基类虚函数的属性能影响派生类相同函数的属性，即通过基类指针能调用派生类private属性的函数，这样就破坏了封装

  ```C++
  class MoveObject {
  public:
      virtual MoveObject* Move() {
          return this;
      }
  };
  class NPCObject :public MoveObject {
  privata:
      virtual NPCObject* Move() {
          return this;
      }
  };
  
  MonsterObject snake;
  NPCObject zsf;
  
  MoveObject* obj = &zsf;
  obj->Move();//基类指针obj根据指向的类进行判断函数执行，如指向类MonsterObject就执行类MonsterObject的Move函数，就算有多个派生类都是一样的结果
  ```



#### final

可以终止函数的重载，能与override配合使用

```C++
MonsterObject* Move() final {
        return this;
}
```



#### override

可以强制要求检查函数是重载，能与final配合使用

```c++
MonsterObject* Move() override {
        return this;
}
```



#### 构造函数

- 构造顺序：MoveObject->MonsterObject
- this指针的地址一样，snake地址就是this指针的地址，只是因为先传入基类MoveObject而已

```C++
class MoveObject {
public:
    int x;
    int y;
    MoveObject() {//构造函数
        std::cout << this << std::endl;
        this->Move();//MoveObject Moving~,调用MoveObject的Move函数
    }
    virtual void Move() {
        std::cout << "MoveObject Moving~\n";
    }
    void test() {
        this->Move();
    }
};
class MonsterObject:public MoveObject {
public:
    MonsterObject() {//构造函数
        std::cout << this << std::endl;
        this->Move();//MonsterObject Moving~,调用MoveObject的Move函数
        MoveObject::test();//通过基类MoveObject的test函数调用MoveObject的Move函数，因为MoveObject的Move是虚函数，所以调用MonsterObject的Move函数
        MoveObject::Move();//调用基类MoveObject的Move函数
        
    }
    void Move() override {
        std::cout << "MonsterObject Moving~\n";
    }
};

MonsterObject snake;//this指针的地址一样，snake地址就是this指针的地址，只是因为先传入基类MoveObject而已
snake.test();//MonsterObject Moving~，因为MoveObject类的Move函数是虚函数
```



#### 析构函数

- 析构函数加上virtual后，析构顺序：MonsterObject->MoveObject
- 若没有对基类的析构函数加上virtual，会只析构了基类，因为p指针是MoveObject类，且此时因为没有完全释放有内存泄露

```C++
class MoveObject {
public:
    int x;
    int y;
    virtual void Move() {
        std::cout << "MoveObject Moving~\n";
    }
    void test() {
        this->Move();
    }
    ~MoveObject() {
        std::cout << "~MoveObject" << std::endl;
        Move();//MoveObject Moving~，因为派生类已经析构，所以Move函数做静态绑定，调用MoveObject的Move函数
    }
};
class MonsterObject:public MoveObject {
public:
    ~MonsterObject() {
        std::cout << "~MonsterObject" << std::endl;
        Move();//MonsterObject Moving~
    }
};

MoveObject* p = new NPCObject();
delete p;//~MoveObject，只析构了基类，因为p指针是MoveObject类，且此时因为没有完全释放有内存泄露
delete p;//析构函数加上virtual后，先~MonsterObject，再~MoveObject
```



#### 默认参数

默认实参以**基类为原则**，派生类的默认参数无效果，因为编译的时候已经决定了默认参数是多少

```C++
class MoveObject {
public:
    virtual void AutoMove(int step = 2) {//默认实参以基类为原则，派生类的默认参数无效果，因为编译的时候已经决定了默认参数是多少
        std::cout << "auto move" << step << std::endl;
    }
};
class MonsterObject:public MoveObject {
public:
    void AutoMove(int step = 3) override {
        std::cout << "auto move" << step << std::endl;
    }
};
```



#### 类指针

```C++
class MoveObject {
public:
    ~MoveObject() {
        std::cout << "~MoveObject" << std::endl;
    }
};
class MonsterObject:public MoveObject {
public:
    ~MonsterObject() {
        std::cout << "~MonsterObject" << std::endl;
    }
};

MoveObject* p = new MonsterObject();

delete p;//~MoveObject，只析构了基类，因为p指针是基类MoveObject，且此时因为没有完全释放有内存泄露
```

因此需要给析构函数加上virtual，变成虚函数

```C++
class MoveObject {
public:
    virtual ~MoveObject() {
        std::cout << "~MoveObject" << std::endl;
    }
};
class MonsterObject:public MoveObject {
public:
    ~MonsterObject() {
        std::cout << "~MonsterObject" << std::endl;
    }
};

MoveObject* p = new MonsterObject();
p->Move();
delete p;//先~MonsterObject，再~MoveObject
```

析构函数变虚函数后就能正常一级一级析构



### typeid

普通数据类型

```C++
int a=1;
std::cout << typeid(a).name() << std::endl;//int,typeid(a)返回引用

long long b;
std::cout << typeid(b).name() << std::endl;//__int64
```

类指针

```C++
class MoveObject{
public:
    virtual void test(){}
};
class Monster :public MoveObject{};

MoveObject* _move = new MoveObject();
_move = new Monster();
std::cout << typeid(*_move).name() << std::endl;//class MoveObject *，不是class Monster的原因是，首先需要*对指针解除引用，其次类不是多态
std::cout << typeid(*_move).name() << std::endl;//class Monster，加上前提后成功
```

引用

```C++
Monster wolf;
MoveObject& lMove = wolf;
std::cout << typeid(lMove).name() << std::endl;//class Monster
```

判断

```C++
if (typeid(lMove) == typeid(Monster))
    std::cout << "Right!" << std::endl;
```



### 抽象类

- 拥有纯虚函数的类称为抽象类，因为该类的函数没有实现，因此不能创建抽象类的实例，但是却可以使用抽象类的指针和引用作为返回或者参数
- 抽象类的构造函数因为不能实际使用，所以一般推荐把抽象类的构造函数定义为protected
- 抽象类的派生类如果没有定义纯虚函数，则该派生类依然是抽象类

```C++
class Animal {
protected:
    Animal() {}
    void virtual Move() = 0;//纯虚函数，只有声明
    void virtual Fly() = 0;//如果抽象类的派生类没有定义所有的纯虚函数，那么该派生类也算是抽象类
};
class Dog :public Animal {
    void virtual Move(){}
};
class Cat :public Animal {
    void virtual Move() {}
};

Animal anml;//抽象类不能实例化
Animal* panml = new Cat();//指针可以
```



### 接口类

类中所有（极大部分）函数定义为纯虚函数的类称为接口类

```C++
class Animal {
    void virtual Move() = 0;
    void virtual Fly() = 0;
    void virtual Eat() = 0;
};
```



### 类的成员函数的函数指针

```C++
class WOLF;
typedef void(WOLF::* PGROUP)();//定义成员函数的函数指针
typedef void(*COUNT)();//定义类静态成员函数的函数指针

class WOLF {
public:
    static void Count() {//静态函数

    }
    WOLF() {
        pGroup= &WOLF::GrouUP0;//构造函数调用函数指针，因为类成员函数不能像普通函数那样进行隐式转换，从WOLF::GrouUP0到&WOLF::GrouUP0，所以要手动标识
        (this->*pGroup)();//构造函数调用函数指针
    }
    void GrouUP0() {
        std::cout << "狼成长到一阶段！" << std::endl;
    }
    void GrouUP1() {
        std::cout << "狼成长到二阶段！" << std::endl;
    }
    void GrouUP2() {
        std::cout << "狼成长到三阶段！" << std::endl;
    }
    PGROUP pGroup;
};

int main()
{
    PGROUP pFunction=&WOLF::GrouUP1;//主函数调用函数指针，固定写法，初始化
    WOLF* pWolf = new WOLF();//主函数调用函数指针，固定写法，初始化
    (pWolf->*pFunction)();//主函数调用函数指针，固定写法，实现多态

    COUNT _count = &WOLF::Count;//类静态成员函数指针，初始化
    _count();
}
```



### 模板类实现虚函数功能

```C++
template<class T>
class Edoyun {
public:
    void show() {
        //本类强制转换为T*
        T* p = static_cast<T*>(this);
        //如果类T实现了Name，那么调用类T的，如果没有调用本类的
        p->Name();
    }
protected:
    void Name() { std::cout << "Edoyun" << std::endl; }
};

//将Sub填入模板类Edoyun中，然后让Sub继承这个模板类，这样就能让父类函数调用子类重写了的函数
class Sub :public Edoyun<Sub> {
public:
    void Name() { std::cout << "Sub" << std::endl; }
};
class Sub2 :public Edoyun<Sub2> {};

Sub sub;
Sub2 sub2;
sub.show();//Sub
sub2.show();//Edoyun
```

 l
