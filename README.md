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
## Lec3
