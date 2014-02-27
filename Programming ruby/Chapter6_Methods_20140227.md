更多关于`方法`的细节
============
[Program ruby:第六章]()

##定义方法
使用 def 定义个方法。  
方法名必须以一个小写字母开头。表示查询的方法通常以`?`结尾，如 intance_of?。“危险的”或者会修改接收者对象的方法，可以用`!`结尾，如
String#chop!。可以被复制的方法以一个`=`结尾。只有 `?`,`~`和`!`这几个怪异的字符能够作为方法名的后缀。
	
	def new_method(arg1,arg2)
		...
	end
方法还可以指定参数的默认值。如果调用者没有明确指定参数值，将使用默认值。
	
	def new_method(arg1="arg1",arg2="arg2")
	...
	end
####可变长的参数列表
如果想传人可变个数的参数是，在“普通”参数名前放置一个星号（`*`）即可。
	
	def new_method(*arg)
		puts "#{arg.join(",")}"
	end
####方法和block
我们已经知道当方法和一个block关联时，可以用yield从方法内部调用这个block。  
不过，如果方法定义的最后一个参数前缀为`&`，那么所关联的block会被转换为一个**Proc**对象，然后赋值给这个参数。
	
	class TaxCalculator
		def initizlize(name,&block)
			@name,@block = name,block
		end
		def get_tax(amount)
			"# @name on #{amount} = #{@block.call(amount)}"
		end
	end

	tc = TaxCalculator.new("sales tax") {|am| am*0.075}

	tc.get_tax(100)    ->  "sales tax on 100 = 7.5"
	tc.get_tax(250)    ->  "sales tax on 250 = 18.75"

##调用方法
可以通过指定接收者、方法名、可选的参数及block，来调用一个方法。
	
	onnection.download_MP3("music") {|p| show_progress(p)}
如果省略了接收者，其默认为`self`，也就是当前的对象。

	self.calss      -> Object
	self.frozen?    -> false
	frozen?         -> false
	self.object_id  -> 7837920
	object_id       -> 7837920
Ruby正是用这种默认机制实现私有方法的调用的。我们无法调用某个接收者的私有方法，它们只能在当前对象（self）中是可用的。  
在没有二义性的情况下，方法调用的括号是可以不写的，有参数也是如此。
	
	new_method(arg1,arg2)
	new_method arg1,arg2

####方法的返回值
每个方法的调用都会返回一个值。方法的值，是方法执行中最后一个语句执行饿结果。  Ruby有个return语句,return的返回值就是其参数的值。
如果不需要return，就省略之。
	
	def hello
		"hello"
	end
如果return了多个参数，就以数组的形式返回。

当调用一个方法是，可以分解一个数组，这样每个成员都被视为单个单独的参数。使用 `*`可以达到这个目的。
	
	def three(a,b,c)
		puts a,b,c
	end

	three(1,2,3)
	three(*[1,2,3])  #效果一样

为方法调用关联一个block
	
	print "(t)imes or (p)lus: "
	times = gets
	print "number: "
	number = Integer(gets)
	if times =~ /^t/
		calc = lambda {|n| n*number}
	else
		calc = lambda {|n| n+number }
	end

	puts((1..10).collect(&calc).join(", "))
输出：

	(t)imes or (p)lus: t
	number: 2
	2,4,6,8,10,12,14,16,18,20
可以当看到，如果方法的最后一个参数前有`&`符号，ruby将其认为i是一个Proc对象。它会将其从参数列表中删除，并将Proc对象转换为一个block,然后关联到该方法。

####散列表收集参数
某些语言支持关键字参数，也就是你可以以任意顺序传入参数的名称和值，而非按指定顺序传入参数。
我们可以用散列表实现同样的效果。  
例如，为SongList实现一个强大的按名搜索功能：
		
	class SongList
		def create_search(name,params)
			#...
		end
	end
	list.create_search("short jazz songs",
					 {
					 	'genre' => "jazz",
					 	'duration_less_than' => 270
					 	})
第一个参数是搜索的名字，第二个就是一个散列式。使用散列式模拟关键字： 查询类型->爵士，时长->小于270s.  
但这种方式比较笨重，且花括号容易被误解是个block。Ruby有一种快捷方式：只要在参数列表中，散列数组在正常的参数之后，并位于任何数组或block参数之前，就可以使用 `key=>value` 键值对。所有这些键值对会被集合到一个散列表中，作为方法的一个参数传入。
	
	list.create_search("short jazz songs",
					  :genre               => :jazz,
					  :duration_less_than  => 270)