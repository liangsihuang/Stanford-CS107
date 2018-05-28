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
`pointer of 4 bytes` has 4GB addressable range.There is actually very little distinction between a pointer and a 4 byte unsigned integer. They both just store integers— the difference is in whether the number is interpreted as a number or as an address.
`dereference a pointer` see in [wikipedia](https://en.wikipedia.org/wiki/Dereference_operator)
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
char ** found = lsearch(&favoriteNote, notes, 5, sizeof(char *), StrCmp); // 两个**是因为array本身存储的是指针
int StrCmp(void * vp1, void * vp2)
{
    char *s1 = * (char **) vp1; // char ** 是和favoriteNote前面的&配套，和下一行的代码对称
    char *s2 = * (char **) vp2;
    return strcmp(s1,s2); //C原生函数：比较C-string
}
```







