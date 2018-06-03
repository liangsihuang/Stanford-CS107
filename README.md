# Stanford-CS107
## Lec2
### bit pattern
1 byte = 8 bits, so 1 byte has 2^8=256 bit patterns. Each pit pattern can be mapped to one number.  
For most computers, 256 bit patterns(1 byte) are mapped to 0-255.
### variable(type) in c/c++
type  | how many bytes|
--------- | --------|
bool  | 1 byte |
char  | 2 bytes |
short  | 4 bytes |
int  | 4 bytes |
long  | 4 bytes |
float  | 4 bytes |
double  | 8 bytes |
pointer | 4 bytes |
### pointer
`pointer of 4 bytes` has 4GB addressable range. There is actually very little distinction between a pointer and a 4 byte unsigned integer. They both just store integers— the difference is in whether the number is interpreted as a number or as an address.  
`dereference a pointer` see in [wikipedia](https://en.wikipedia.org/wiki/Dereference_operator)
__dereference简单来说，就是根据reference取得资源__
### char
The ASCII code difines 128 characters and a mapping of those characters onto the numbers 0-127.  
所以bit pattern的头一个必然是0. （剩下的127-255怎么利用，不同计算机的处理不同）
'A' = 65 = 2^6+2^0 = 0100,0001
### short
可以有不同的映射规则(protocol)，只要保持一致。
2 bytes 可以表示 2^16个不同的值。为了表示负数，`头一个bit表示正负号（0正1负）`，所以  
2 bytes 表示的数字： 2^(-15) - 2^(15)-1  （中间有个0破坏了对称性）  
实际应用：不能仅仅改变头一个bit来得到负数的bit pattern，原因：  
`because we want addition and subtraction to actually follow very simple rules`  
比如：short型的数字7+(-7)应该等于0  
    //0000,0000 0000,0111  
    + 1000,0000 0000,0111  
    = 1000,0000 0000,1110（不是0）  
实际表示：15+(-15)=0（domino效应，多出来的1被丢掉）  
    //0000,0000 0000,1111 （15）   
    + 1111,1111 1111,0001  （-15是这样表示的，不仅头一个bit变为1，其他bit全部反转，再加1）  
    = 0000,0000 0000,0000  
### cast
```cpp
char ch='A';
short s=ch;
cout<<s<<endl; //Print out: 65
```
1 byte to 2 bytes: copy the bit pattern, extra space set to all 0s.  
(hidden check: first bit is 0, positive number)
```cpp
short s=67;
char ch=s;
cout<<ch<<endl; //Print out: C
```
2 byte to 1 bytes: only copy the lower bit pattern(0100,0011), no room for higher bit pattern(0000,0000)  
类似的（int to short）: 会丢失高位的bit pattern，表示另一个数！
```cpp
short s = -1; //bit pattern: 1111,1111 1111,1111
int i = s; //bit pattern: 1111,1111 1111,1111 1111,1111 1111,1111
// sign extend! 延申1到头bit，这样才有domino效应，才能保留符号
```
### float
* 4 bytes = 32 bits  
* 实际的标准：
    * 1 bit: 正负号 (-1)^s
    * 8个bit: exp=0-255, 8个bit全为1，exp=255
    * 23个bit：.xxxxxx ， 2^(-1)+2^(-2)+...2^(-23)
* 表示的数字：
    * (-1)^s乘1.xxxxx乘2^(exp-127), 2的指数范围-127~128
```cpp
int i = 5; //bit pattern: 0000,0000 0000,0000 0000,0000 0000,0101
float f=i;
cout<<f<<end; //print out: 5.0
//暗地里，计算机做了一次转换：转换成float的bit pattern: 1.25*2^（2）,exp=129,.xxxx=0.25

int i = 37;
float f=*(float *)&i; //没有做转换，所以f不是37.0
```

## Lec3
### brute force reinterpertation of the address
```cpp
short s=45;
double d = *(double*)&s;
```
### struct and array
```cpp
int arr[5];
arr[3]=128;
((short*)arr)[6]=2;
cout<<arr[3]<<endl; // 不是128了, arr[3]前半部分（2个bytes）的bit pattern被改成了2

struct student{
    char * name; // char pointer 有多少 bytes??? 4
    char suid[8];
    int numUnits;
    };
student pupils[4];
pupils[2].name=strdup("Adam"); //string duplication, 在heap里动态分配内存 string "Adam\0"
```
### Write generic function in C
```c
void swap(int *ap, int *bp)
{
    int temp=*ap;
    *ap=*bp;
    *bp=temp;
};
int x=7;
int y=117;
swap(&x, &y);
```
如何对所有类型都通用？ 在C++里面可以用template实现，在C里面怎么做？？
## Lec4
### Continue to write generic function in C
`very powerful very dangerous`
```c
void swap(void * vp1, void * vp2, int size) //generic pointer
{
    char buffer[size]; // 用char来计算byte，直接操作memory
    memcpy(buffer, vp1, size);
    memcpy(vp1, vp2, size);
    memcpy(vp2, buffer, size);
}
int x=17, y=37;
swap(&x, &y, sizeof(int));
```
C++的template: 每针对一种类型，都会复制一次代码。有什么副作用？？？
generic优势:更节约？
坏处：wrong call 一样 complie，得到错误的结果。特别是对于比swap更复杂的函数
```c
int i=44;
short s=5;
swap(&i, &s, sizeof(short)); //编译出错误的结果 sizeof(int) even worse
```
当要交换指针时，更加疑惑：要传入什么参数?非常烧脑！
```c
char * husband = strdup("Fred");
char * wife = strdup("Wilma");
swap(&husband, &wife, sizeof(char *)); //正确做法。
swap(husband, wife, sizeof(char *)); //错误做法。一样能编译
//变成husband指向 Wilm\0 , wife指向 Freda\0。 swap 4 bytes figure!
```
example : linear search
```
int lsearch(int key, int array[], int size)
{
    for(int i=0; i<size; i++){
        if (array[i]==key){
            return i;
        }
    }
    return -1;
}
```
generic version 1
```c
void *lsearch(void *key, void *base, int n, int elemSize)
{
    for(int i=0; i<n; i++){
        void *elemAddr = (char *)base + i*elemSize;
        if (memcmp(key, elemAddr, elemSize)==0) return elemAddr; //只能比较原始的类型，不能比较里面存储的是地址的指针类型
    }
    return Null;
}
```
## Lec5
generic version 2
```c
void *lsearch(void *key, void *base, int n, int elemSize, int (* cmpfn)(void *, void *)) //cmpfn前面的*可以没有？
{
    for(int i=0; i<n; i++){
        void *elemAddr = (char *)base +i*elemSize;
        if (cmpfn(key, elemAddr) == 0) return elemAddr;
    }
    return Null;
}
```
example: int domain
```c
int array[] = {4,2,3,7,11,6};
int size=6;
int number=7; 
int *found = lsearch(&number, array, 6, sizeof(int), intCmp); //&number为指向number的指针，因为intCmp传入的参数是指针
//found==Null 说明没找到
int intCmp(void * elem1, void * elem2)
{
    int * ip1 = elem1;
    int * ip2 = elem2;
    return *ip1-*ip2; // 相等即返回0
}
```
example: c-string domain
```c
char * notes[]={"Ab", "F#", "B", "Gb", "D"};
char * favoriteNote = "Eb";
char ** found = lsearch(&favoriteNote, notes, 5, sizeof(char *), StrCmp); // 两个**是因为array本身存储的是指针??
int StrCmp(void * vp1, void * vp2)
{
    char *s1 = * (char **) vp1; // *(char **) 是和favoriteNote前面的&配套，和下一行的代码对称
    //
    //问题：为什么不直接 char *vp1？ dereference不是可以抵消一个*吗？？
    //我的见解：char ** 和char *效果上完全相同（指针都是4 bytes）。但给人的理解不同。
    //dereference一定要，因为strcmp要求传入C-string的指针，对应于 &favoriteNote的reference
    char *s2 = * (char **) vp2;
    return strcmp(s1,s2); //C原生函数：比较C-string
}
//也可以写成不对称的形式：
char ** found = lsearch(favoriteNote, notes, 5, sizeof(char *), StrCmp); 
int StrCmp(void * vp1, void * vp2)
{
    char *s1 = (char *) vp1; //不用dereference
    char *s2 = * (char **) vp2;
    return strcmp(s1,s2); 
}
```
function pointer是什么？？学到大概33：00时，讲到了member function(method), free function的区别, 没听懂！！！

从generic algorithm 到 generic data structure  
stack.h
```c
typedef struct{
    int *elems;
    int loglength; //老师第一次写：int logicallen;
    int alloclength;
} stack; //struct是C里面有的类型，但数据完全暴露（都是public）。假装是private，用函数操作

void StackNew(stack *s);
void StackDispose(stack *s);
void StackPush(stack *s, int value);
int StackPop(stack *s);
```
implement
```c
void StackNew(stack *s)
{
    s->logicallen=0;
    s->alloclen=4;
    s->elems=malloc(4*sizeof(int)); //C function: 在heap里面找到16 bytes的block，返回地址
    assert(s->elems!=Null); //C function: 没有找到内存就中止！
}
```
## Lec6
continue to implement of stach in C
```c
void StackDispose(stack *s)
{
    free(s->elems);
    // free(s); 错误：s只是个局部变量
}

void StackPush(stack *s, int vaule)
{
    if (s->loglength == s->alloclength{
        s->alloclength *= 2;
        s->elems = realloc(s->elems, s->alloclength*sizeof(int)); //C++ 没有对应的函数，以后再讲为什么。 C++要copy，然后到另外一个地方分配内存
        assert(s->elems!=Null);
    }
    s->elems[s->loglength]=value;
    s->loglength++;
}

int StackPop(stack *s)
{
    assert(s->loglength>0);
    s->loglength--;
    return s-elems[s->loglength];
}
```
generic stack: stack.h
```c
typedef struct{
    void *elems;
    int elemSize;
    int loglength;
    int alloclength;
} stack;

void StackNew(stack *s, int elemSize);
void StackDispose(stack *s);
void StackPush(stack *s, void *elemAddr);
void StackPop(stack *s, void *elemAddr);
```
stack.c
```
void StackNew(stack *s, int elemSize)
{
    assert(s->elemSize>0);
    s->elemSize = elemSize;
    s->loglength = 0;
    s->alloclength=4;
    s->elems=malloc(4*elemSize);
    assert(s->elems!=NULL);
}

void StackDispose(stack *s)
{
    //这里有东西省略了！
    free(s->elems);
}

static void StackGrow(stack *s) //static 表示只能在本文件中使用此function，相当于C++的private
{
    s->alloclength*=2;
    s->elems=realloc(s->elems, s->alloclength*s->elemSize);
}
void StackPush(stack *s, void *elemAddr)
{
    if(s->loglength==s->alloclength) StackGrow(s);
    void *target = (char *)s->elems+s->loglength*s->elemSize;
    memcpy(target, elemAddr, s->elemSize);
    s->loglength++;
}

void StackPop(stack *s, void *elemAddr)
{
    void *source = (char *)s->elems+(s->loglength-1)*s->elemSize;
    memcpy(elemAddr, source, s->elemSize);
    s->loglength--;
}

## Lec7
储存c-string的stack
```c
int main()
{
    const char *friends[]={"Al", "Bob", "Carl"};
    stack stringStack;
    StackNew(&stringStack, sizeof(char *));
    for(int i=0; i<3; i++){
        char *copy=strdup(friends[i]); //注意：strdup里面是malloc，会在heap里面分配内存
        StackPush(&stringStack, &copy); //注意：必须有&，没有也会compile
    }
    char *name;
    for (int i=0; i<3; i++){
        StackPop(&stringStack, &name); //注意：必须有&，没有也会compile
        printf("%s\n",name);
        free(name); //name只指向的是动态的内存，必须free，否则成为orphan
    }
    StackDispose(&stringStack);
}
```
假如stack里面还有c-string，没有被free。stack被StackDispose后这些c-string都成为orphan。所以需要改写
```c
void StackNew(stack *s, int elemSize, void (*freefn)(void *));
typedef struct{
    //加一项
    void (*freefn)(void *)
} stack;
void StackDispose(stack *s)
{
    if(s->freefn!=NULL){        //不是储存c-string的stack不用定义freefn，参数传个NULL
        for(int i=0; i<s->loglength;i++){
            s->freefn((char *)s-elems+i*s->elemSize);
        }
    }
    free(s->elems);
}

Stack stringStack;
StackNew(&stringStack, sizeof(char *), StringFree);
void StringFree(void *elem)
{
    free(*(char **) elem);
}
```
实现一个rotate函数，类似swap
```c
void rotate(void *front, void *middle, void *end)
{
    int frontSize = (char *)middle - (char *)front;
    int backSize = (char *)end - (char *)middle;
    char buffer[frontSize];
    memcpy(buffer, front, frontSize);
    memmove(front, middle, backSize); //因为有可能overlap，所以不能用memcpy
    memcpy((char *)end-frontSize, buffer, frontSize); //理论上memcpy都可以用memmove，但效率低
}
```
## Lec8 Heap Management
How information about allocations are stored in the heap? 每一个blob前面有一个header记录blob的大小    
还讲了 stack 为什么叫 stack? 就是因为函数的层层调用，后调用先返回。  
ALU(Arithmetic Logic Unit)到register(寄存器？)再到RAM(Random Access Memory)??? 不懂
## Lec9 
 How a Code Snippet is Translated into Assembly Instructions (汇编指令)  
 到底怎么从code到汇编语言的？不懂。汇编有什么用？为什么不直接code到机器码？










