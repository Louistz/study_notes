
ruby中的类
=========
[Program ruby:第三章]()

类名首字母大写，而方法名通常以一个小写字母开始

####创建
initialize 方法，new时传入的参数，类似于java中的构造函数

	calss Song
		def initialize(name,artist,duration)
			@name=name
			@artist=artist
			@duration=duration
		end
	end

	song = song.new("music","artist",100)
	song.inspect -># <Song :0x1c8ac8, @duration=100,@artist="artist",@name="music">
	song.to_s    -># <Song :0x1c8ce4>

####继承

	class KaraokeSong < Song
		.....
		def to_s
			super + " [#@lyrics]"
		end
	end

####Getter

1.自定义方式：

	def duration
		@duration
	end

ruby 默认函数返回最后一行的字符串
2.ruby快捷方式：

	class Song
		attr_reader :name, :artist, :duration
	end

####Setter

1.自定义方式：
	
	def name=(new_name)
		@name = new_name
	end

敢信？! 方法的名字可以是“name=” ...，调用时后面可以直接跟参数： song.name="newname"

2.ruby快捷方式：

	class Song
		attr_writer :duration
	end

####类变量和类方法

类变量有两个“@”前缀,使用前必须初始化，只存在一份拷贝

	class Song
		@@plays = 0
		def initialize(...)
		...
		end
	end

类需要提供不束缚于任何特定对象的方法。例如 ： Song.new(...),File.delete(...)

	class Example
		def instan_method      #instance method
		...
		end
		
		def Example.class_method    #class method
		...
		end
	end

#####单例和内部类等等其他

####访问控制

* Public ：可以被任何人调用，没有限制。方法默认是public的，initialize除外，它是private的
* Protected: 只能被定义了该方法的类或者其子类访问。整个家族均可访问
* Private： 方法不能被明确的接受者调用，其接受者只能是 *self*。意味着只能在对象的上下文中被访问

######定义
	
	class MyClass
		def method1  #default is public
			#...
		end
	  protected
	  	def method2
	  	...
	  	end
	  private
	  	def method3
	  	...
	  	end
	  public
	  	def method4
	  	...
	  	end
	end

或者

	class MyClass

		def mehtod1
		....
		..
		.
		end

		public :method1, :method2, :mehod3
		private :method4, :method5
		protected :method6
	end
	