* 如果程序是从try代码块或者catch代码块中返回时 ,finally语句在return语句执行之后return返回之前执行的。
* 当finally有返回值时，会直接返回。不会再去返回try或者catch中的返回值。   
* 如果try和catch的return是一个变量时且函数的是从其中一个返回时，后面finally中语句即使有对返回的变量进行赋值的操作时，也不会影响返回的值。
 
