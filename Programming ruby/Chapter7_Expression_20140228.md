表达式
=====
[Program ruby:第七章]()

####连等和链式语句
	
	a = b= c = 0                 ->  0
	[3,1,7,0].sort.reverse       ->  [7,3,1,0]
那些在c和java中的普通语句，在ruby中也是表达式。例如 if和case语句都返回最后执行的表达式的值

	song_type =  if song.mp3_type == MP3:Jazz
					if song.written < Data.new(1935,1,1)
						Song::TradJazz
					else
						Song::Jazz
					end
				else
					Song::Other
				end
	rating = case votes_cast
				when 0...10   then  Rating::SkipThisOne
				when 10...50  then  Rating::CouldDoBetter
				else                Rating::Rave
				end
ruby提供基本的运算符集（如+、-、*、/）等.  
实际上，ruby中的许多运算符是由方法调用来实现的。如：
	
	a * b + c    等价于
	(a.*(b)).+(c)
因为任何东西都是对象，而且可以重新定义实例方法，因此可以重新定义任何不满足需求的基本算术方法。
	
	class Fixnum
		alias old_plus +    #将 + 引用为 old_plus

		def +(other)
			odl_plus(other).succ
		end
	end 

####杂项

 * 如果用反引号或者以`%x`为前缀的分界形式，括起一个字符串，默认情况下它会被当做底层操作系统的命令执行  

如：

	`data`                   ->  "Fri Feb 28 09:48:39 CST 2014\n"  
	%x{echo "Hello there"}   ->  "Hello there\n"  

 * 重定义反引号

如：

	alias old_backquote `  
	def `(cmd)  
		result = old_backquote(cmd)  
		if $? != 0  
			fail "Command #{cmd} faild: #$?"  
		end  
		result  
	end  
	print 'date'  
	print 'data'   

####赋值
链式、并行  
**陷阱**：   
类的可写属性有个陷阱。通常，类中的方法可以通过函数形式（即带一个隐式self作为接受者）调用同一个类的其他方法和它的父类的方法。
然而，这并不适用于属性赋值函数： Ruby看到赋值语句是，会认为左边的名字是局部变量，而不是一个为属性复制的方法调用。

	class BrokenAmplifier
		attr_accessor :left_channel, :right_channel
		def volume=(vol)
			left_channel = self.right_channel = vol
		end
	end

	ba = BrokenAmplifier.new
	ba.left_channel = ba.right_channel = 99
	ba.voluem = 5
	ba.left_channel    -> 99
	ba.right_channel   -> 5

叠起和展开数组  

嵌套赋值
 
ruby不支持 `--`， `++`，可以用 `+=`，`-=`代替之

####条件执行
ruby的“真值”,任何不是nil或者常量false的值都为真。

######Defined? 、与 、或 和非  
新操作符 defined?  
and、&&、or、|| 、not、！ and和or的优先级相同，而 &&的优先级高于||  
defined? ，如果参数未被定义，返回nil；否则返回对参数的一个描述。  
	
	==              测试值相等与否
	===             用来比较case语句的目标和每个when从句的项
	<==>            通用比较操作符。根据接收者小于、等于和大于其参数，返回 -1、0和 1
	<,<=,>=,>
	=~              正则表达式模式匹配操作符
	eql?            如果接受者和参数有相同的类型和相等的值，则返回真。 如 1==1.0为真，但 1.eql?(1.0)
	equal?          如果接受者和参数具有相同的对象ID，则返回真

==和=~都有相反的形式： ！=和！~.但 ruby在读取程序的时候，会进行行转换： a!=b   ->  !(a==b).这意味着，如果自定义的类 重载了 ==或者=~,那么也自动得到！=和！~。   

######if 和 unless  
如果if 分布到多个行上，那么可以不用then
	
	if song.artis == 'Gil'
		handle = "Dizzy"
	elsif song.artis == "Parker"
		handle = "Bird"
	else
		handle = "unknown"
	end

	if song.artis == 'Gil' then handle = "Dizzy"
	elsif song.artis == "Parker" then handle = "Bird"
	else handle = "unknown"
	end
也可以使用冒号`：`代替then
	
	if song.artis == 'Gil' : handle = "Dizzy"
	elsif song.artis == "Parker" : handle = "Bird"
	else handle = "unknown"
	end

表达式用法
	
	handle = if song.artis == 'Gil' then 
				"Dizzy"
			elsif song.artis == "Parker" then 
				"Bird"
			else 
				"unknown"
			end
否定形式,unless是if的否定形式。

	unless song.duration > 180
		cost = 0.25
	else
		cost = 0.35
	end

三元表达式
	
	cost = song.duration > 180 ? 0.35 : 0.25
Ruby和perl都有一个特性： 语句修饰符允许将条件语句附加到常规语句的尾部
	
	mon,day,year = $1,$2,$3 if date =~ /(\d\d)-(\d\d)-(\d\d) /
	puts "a= #{a}" if debug

####case表达式
	case 相当于多路if.而它有两种形式，使得它更加强大。

	leap = case
		when year%400 ==0 : true
		when year%100 ==0 : false
		else year%4 ==0
		end

	case input_line
	when "debug"
		dump_debug_info
		dump_symbols
	when /p\s+(\w+)/
		dump_variable($1)
	when "exit","quit"
		exit
	else 
		print "Illegal comman:#{input_line}"
	end

case的测试时通过 `comparison === target`来完成。只要一个类为===提供有意义的语句，那么该类的对象就可以在case表达式中使用

	case shape
	when Square,Retangle
		#...
	when Circle
		#...
	when Triangle
		#...
	else
		#...
	end

######循环
	
	while ...
	...
	end
until和while相反，执行循环体知道条件为真
	
	until ...
	...
	end
用 while和until做语句修饰符时，不过被修饰的语句是一个 begin/end块，那么不管表达式的布尔值时什么，快内的代码至少执行一次。
	
	print "Hello!" while false
	begin
		print "Goodbye" 
	end while false                 ->  Goodbye


####迭代器
	
	3.times do 
		print "Ho"
	end

	0.upto(9) do |x|
		print x," "
	end

	0.step(12,3){ |x| print x," "}

	[1,2,4,65,2].each {|val| print val," "}

loop循环一直执行到给定的块，直到跳出循环
	loop do
		#block...
	end

######For...in
只要类支持each方法，就可以使用for...in 去遍历
######break,redo,next
break 终止最接近的封闭循环体，然后继续执行block后面的语句。  
redo充循环头重新开始执行循环，但不 重新计算循环条件表达式或者获得迭代中的下一个元素。  
next跳出本次循环的末尾，开始下一次迭代。 
######retry
retry 语句使得一个循环重新执行当前的迭代。retry在重新执行之前会重新就散传递给迭代的所有参数