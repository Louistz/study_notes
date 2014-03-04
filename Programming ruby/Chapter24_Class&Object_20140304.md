类和对象
========
[Program ruby:第二十四章]()

Ruby只有一个底层的类和对象结构。  
一个ruby对象包含三个部分： 一组标志，一些实例变量，以及相关联的类。  
ruby类是Class的一个对象，包括一个对象所具备的所有内容、外加方法列表和一个超类的引用。  

ruby中的所有方法调用必须指派一个接收者（默认为self,当前对象）。 Ruby通过在接收者的类中查找方法列表，来找到要调用的方法。如果在其中没有找到，则在其包含的模块中查找，然后是父类，父类的模块，继而是父类的父类。
如果最终都没有找到，Ruby调用最终接收者的 method_missing方法

####特定于Object的类
Ruby允许创建一个和特定对象绑定的类。
	
	a = "Hello"
	b = a.dup

	class <<a 
		def to_s
			"The value is #{self}"
		end
		def two_times
			self + self
		end
	end

	a.to_s  -> "The value is Hello"
	a.two_times -> "hello"
	b.to_s  -> "hello"
`class <<ojb` 意思是“为对象obj构建一个新类”， 也可以改写为：
	
	def a.to_s
		"The value is #{self}"
	end
	def a.two_times
		self + self
	end
效果相同：想对象a中添加一个类。  
那么，这就有了一个提示： 创建虚拟类并插入作为a的直接类。a的原有类String作为虚拟类的超类。  
Ruby中的类是**永不关闭**的；随时可以打开一个类并添加新的方法。这对虚拟类也适用。如果对象的`klass`引用已经指向了一个虚拟类，则并不会创建一个新的虚拟类。这意味着，前面示例中定义的第一个方法创建了一个虚拟类，而第二个围棋添加了一个方法。  

object#extend 方法将其参数中的方法添加到调用该方法的接收者中。因此，如果需要的话，它也创建一个虚拟类。  
	
	obj.extend(Mod)    
	等价于
	class <<ojb
		include Mod
	end

####Mixin模块
当包含一个模块，模块的实例方法就变味类的实例方法。Ruby创建了一个指向该模块的匿名代理类，并将这个代理插入到实施包含的类中作为其直接超类。 代理类包含有指向模块实例变量和示例方法的引用。  
这个代理类指向唯一的底层模块：改变模块中某个方法的电脑怪异，它就会改变所有包含该模块的类。
####扩展对象
就像使用 `class <<ojb`一样，可以使用Object#extend将一个模块混入到对象中。如：
	
	module Humor
		def tickle
			"Hee,hee!"
		end
	end

	a = "Grouchy"
	a。extend Humor
	a.tickle      -> "Hee,hee"

######将模块中的方法添加到类的级别
	
	module Humor
		def tickle
				"Hee,hee!"
			end
		end
	end

	class Grouchy
		include Humor
		extend Humor
	end

	Grouchy.tickle          -> "Hee,hee!"
	a = Grouchy.new         
	a.tickle                -> "Hee,hee!"

####类和模块的定义
不同java或c++语言：类定义是在编译期处理的，编译器创建符号表，计算出需要分配多少存储空间，构造分发表，已经其他事情。  
作为一个动态语言，在ruby中，类和模块的定义是可执行的代码。虽然是在编译器进行解析，但当你遇到定义是，类和模块实在运行时创建的（方法的定义也是如此）。
如果类定义是可执行的代码，这意味着它是在某个对象的上下文中执行的：self必定指向了**某个东西**。
	
	class Test
		puts "Class of self = #{self.class}"
		puts "Name of self = #{self.name}"
	end

	->Class of self = Class
	->Name of self = Test
这意味着类定义在执行时就是以这个类作为当前对象。从而表明，metaclass及其超类中的方法，在执行方法定义时时可用的。

####类实例变量
如果类定义是在某个对象的上下文中执行的，这意味着这个类可能具有实例变量
	
	class Test
		@cls_var = 123
		def Test.inc
			@cls_var+=1
		end
	end
	Test.inc -> 124
	Test.inc -> 125
如果类有它们自己的实例变量，就可以使用 attr_reader访问了。对于一般的实例变量来说，属性访问时在类一级定义的。对类实例变量来说，必须在metaclass中定义访问函数。
	
	class Test
		@cls_var = 123
		class <<self
			attr_reader :cls_var
		end
	end

	Test.cls_var     ->   123