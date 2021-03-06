#Matlab



###解方程 - 多项式求解

对于多项式**p(x) = x^3 - 6x^2 - 72x - 27**，求多项式p(x)=0的根;    
可用多项式求根函数roots（p）, 其中p为多项式系数向量，即         

	>> p = [1, -6, -72, -27]  
	p = 
		1    -6   -72   -27
		
p是多项式的MATLAB描述方法，我们可用poly2str(p,'x')函数，来显示多项式的形式:
	>>px = poly2str(p,'x')
	px =
		x^3 - 6 x^2 - 72 x - 27
多项式的根解法如下：    
	
	>> format rat %以有理数显示
	>> r=roots(p)
	r =
		2170/179 
		-648/113 
		-769/1980 
	
###解方程 - 数学表达式
在MATLAB中，求解用符号表达式表示的代数方程可由函数solve实现，    
其调用格式为：   
solve(s,v): 求解符号表达式s的代数方程，求解变量为v。  
用单引号把包含字母的方程写出来，后面指定未知数是什么字母。  
例如，求方程(x+2)*x=2的解,解法如下：    

	>> x = solve('(x+2)^x=2','x')
	
	x =
	
	0.69829942170241042826920133106081
	
得到符号解，具有缺省精度。    
如果需要指定精度的解，则: 
	
	>> x=vpa(x,3)
	
	x =
	
	.698

###求解方程组    

     >> [pxpy, zx, zy, cx, cy]=solve('cy=f*zy','cx=e*zx^2','z=zx+zy','pxpy=f/(2*zx*e)','pxpy*(b/cy)-(a/cx)=0','pxpy,zx,zy,cx,cy')
     
用逗号分隔每个方程，最后一个字符串中用逗号分隔未知数。    
函数返回结果是一个列向量。

多元多次方程组
	
	x1 + 2*x2 = 8
	2*x1 + 3*x2 = 13

方法1:
	
	>> [x1,x2] = solve('x1+2*x2 = 8','2*x1 + 3*x2=13', 'x1,x2')
	x1 =
	
	2
	
	x2 =
	
	3

方法2:
1. x=inv(A)*b 采用求逆运算解方程组；    
2. x=A\b 采用左除运算解方程组。

	>> A=[1,2;2,3]; b=[8;13];	
	>> x=inv(A) x) b??
				
	x =
		2
		3  

即二元一次方程组的解x1和x2分别是2和3。
