# Linux C语言编程基本原理与实践

## c语言文件类型

- main.c      源文件	
-  main.o     连接文件
-  main.out  可执行文件
- main.h       声明文件

## vim操作

```
//命令行模式下：
a 		//在字符后插入字符
i 		//在字符前插入字符
shift+a //在行尾插入字符
shift+i //在行首插入字符
o 		//在下一行插入字符
shift+o //在上一行插入字符
x 		//删除当前字符
shift+x	//删除当前字符的前一个字符
dd		//删除当前行
u		//撤销操作
ctrl+r	//恢复操作
gg		//跳至文首
shift+g	//跳至文尾
gd		//跳至当前光标所在变量的声明处
*		//查找光标所在处的单词，向下查找
#		//查找光标所在处的单词，向上查找
[num]gg	//跳至num行，num为行数
p 		//粘贴内容
>>		//当前行向右缩进
<<		//当前行向左缩进
gg=G	//自动排版
```

## vim文件操作

```
:sp filename 	//新建一个文件
ctrl+w+下箭头 	  //切换到下一个文件
ctrl+w+上箭头	  //切换到上一个文件
:set nu			//查看行号
[num]dd 		//当前光标往下的num行被剪切
:wqa			//全部文件保存退出
```

## 编译文件

```
gcc -g main.c -o main.out //编译的可调试程序
gcc -c main.c -o main.o	  //编译的文件，不可执行
make					  //根据文件夹下的makefile文件编译
```

## main函数

```
标准格式：
int main(int argv,char* argc[])
{
	return 0;
}
argv表示参数个数
argc[]表示参数的值
echo $?	//返回程序执行的状态码，0正常，其他错误
```

## C语言标准输入输出错误流

- 三种标准流
  - stdin标准输入流(默认键盘)
  - stdout标准输出流(默认显示屏)
  - stderr标准错误流

```
//printf("hello");等同于
fprintf(stdout,"hello");		 //向指定的流输出内容
//scanf("%d",&a);等同于
fscanf(stdin,"%d",&a);        	 //从指定的流接收数据
fprintf(stderr,"错误");		    //向错误流输出信息

输出流的重定向
./main.out >> a.txt 
将main.out的输出写入a.txt  >表示覆盖写入，>>表示追加写入

输入流的重定向
./main.out < a.txt
将a.txt的内容作为main.out的输入信息
```

