模块
=============
[Program ruby:第九章]()

模块是一种将 方法、类与常量组织在一起的方式。两个好处：

1. 命名空间（namespace）防止命名冲突
2. 模块实现了 mixin 功能

####模块机制
定义

	module Trig
		PI = 3.141592654
		def Trig.sin(x)
			...
		end
		def Trig.cos(x)
			...
		end
	end
引用
	
	require 'trig'
	y = Trig.sin(Trig::PI/4)
使用模块名和两个冒号来引用常量

####Mixin
mixin功能，以雷霆之势，极大地消除了对多重继承的需要。  
模块并没有实例，因为模块不是类。但可以在类中用*include*来包含一个模块。当包含发生时，模块所有的实例方法瞬间在类中也可使用了。这种`mixin(混入)`的行为就像是一个超类。
	
	module Debug
		def who_am_i?
			"#{self.class.name}(\##{self.object_id}):#{self.to_s}"
		end

	class EightTrack
		include Debug
		#...
	end

	et = EightTrack.new("sfasdfas")
	et.who_am_i?       -> "EightTrack(#123456):sfasdfas"
ruby产生一个指向指定模块的引用。如果模块位于外部文件，必须先require再include。include并非简单地将模块的实例方法拷贝到类中，而是建立一个由类到所包含模块的引用，一处变动，所有引用的东西都会变动。  
Mixin的真正力量在于，当**mixin中的代码和使用它的类的代码开始交互式，他们会一起迸发出来**。例如：  
Ruby mixin--comparable,cimparable mixin向类中添加比较操作符（<,<=,==,>=,>）以及between？方法。为了使其能过工作，comparable假定任何时候使用它的类都定义了<=>操作符。我们定义一个方法，在包含comparable，就可以免费得到6个比较函数。
####迭代器和可枚举模块
	
####组合模块
######Mixin中的实例变量

	module Obserbable
		def observers
			@observer_list ||= []
		end

		def add_observer(obj)
			observers << obj
		end

		def notify_observers
			observers.each {|o| o.update}
		end
	end

mixin中的变量可能会和它的宿主变量名字冲突。所以，多数时候，mixin并不带有他们自己的实例数据，它们只是使用访问方法从客户对象中取得数据。如果一定要用，确保名字的唯一性。  
######歧义名字的解析过程
	首先是从直属类中查找，然后是类中包含的mixin，之后是超类以及超类的mixin.如果类中有多个模块，那么最后一个包含的模块将会第一个搜索
######文件包含
	
	load "filename.rb" 或者
	require "filename"
加载文件中的局部变量不会蔓延到加载它们所在的范围中。  
require有额外的功能：它可以加载共享的二进制文件。 因为load可以无条件的包含源文件，可以使用它来重新加载一个在程序开始执行后可能更改的源文件。