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