块（blocks）和迭代器(iterators)
===========================
[Program ruby:第四章]()

##引子
现有SongListL类，实现一个 with_title 方法，方法接受一个字符串，返回以此为歌名的歌曲。

第一种实现：

	class SongList
		def with_title(title)
			for i in 0...@songs.length
				return @songs[i] if title == @songs[i].name
			end
			return nil
		end
	end

更自然的方式：
	
	calss SongList
		def with_title(title)
			@songs.find {|song| title == song.name}
		end
	end

这里的`find`就是一种**迭代器**，它反复调用block中的代码（block就是`{}`中的内容）。

##实现迭代器
Ruby的迭代器只不过是可以调用block的方法而已。

####block
block是处在`{...}`或者`do ... end`之间的一组代码（通常单行用前者，多行用后者），它只和方法调用一起出现。  
在遇到block时，并不立即执行其中代码。Ruby会记住block出现时的上下文（局部变量，当前对象等）然后执行方法调用。  

方法中，block可以像方法一样被yield语句调用。执行一次yield，就会调用block中的代码。当block执行结束，控制返回紧随yield之后的那条语句。  
例如：

	def tree_times
		yield
		yield
		yield
	end
	three_times (pust "hello")
输出：
	
	hello
	hello
	hello

带参数的例子,实现小于某个值的所有Fibonacci数列：

	def fib_up_to(max)
		i1,i2=1,1
		while i1<max
			yield i1
			i1,i2=i2,i1+i2
		end
	end
	fib_up_to(1000) {|f| print f, " "}
输出：
	
	1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987 

如果传递给block的参数是已经存在的局部变量，那么这些变量即为block的参数，它们的值可能会因block的执行而改变。  
如果block内的变量时第一次出现的，那么它们就是block的局部变量。如果它们出现在block外，那么就与外部共享这些变量。  
block也可以返回值给的方法：

	class Array
		def find
			for i in 0...size
				value = self[i]
				return value if yield(value)
			end
			return nil
		end
	end

	[1,2,4,5,7,8,9].find {|v| v*v > 30 } -> 7

####iterator
一些迭代器是 Ruby 的许多 collections类型所共有的。
例如： each、collect. each是最简单的迭代器,它所做的就是连续访问收集的所有元素。
	
	[1,3,4,6].each {|i| puts i}
另外一个就是collect了，它从集合中获得各个元素并传递给block。block的返回结果被用来生成一个i性能的数组。例如：  

	['H','A','L'].collect { |x| x.succ}    ->   ['I','B','M']
再有 inject方法可以遍历集合的所有成员以累计出一个只。
	
	[1,3,5,7].inject(0) {|sum,element| sum+element }         -> 16
	[1,3,5,7].inject(1) {|product,element| product*element}  -> 105
工作流程：

* 第一次被执行是，sum被置为inject的参数，而element为集合第一个元素
* 接下来每次执行block时，sum被置为上次block的返回值。  

如果inject 没有参数，则默认集合的第一个元素为初始值，并且从第二个元素开始迭代。

####内迭代器和外迭代器
迭代器位于集合内部和外部。

##事务Blocks
例如：打开文件后需要关闭文件，尽管攒在传统的方式来实现（打开->操作->关闭）。但也存在"应该由文件负责自身的关闭"这样的观点。如：

	class File
		def File.open_andprocess(*args)
			f=File.open(*args)
			yield f
			f.close()
		end
	end

	File.open_and_process("testfile.txt","r") do |file|
		while line=file.gets
			pusts line
		end
	end
改写open,使其自动判定block存在与否，自动做不同的动作,Kernel有个方法：block_given? 方法，当某个方法和block关联在一个，他将返回真。

	class File
		def File.my_open(*args)
			result = file = File.new(*args)
			if block_given?
				result = yield file
				file.close
			end
			return result
		end
	end

##Blocks作为闭包
回到点唱机的例子。某些时候，用户选择歌曲，然后点击`start`和`pause`按钮来播放和暂停。
通常的做法是这样：

	start_button = Button.new("start")
	pause_button = Button.new("pause")

	class StartButton < Button
		def initialize
			super("start")
		end
		def button_pressed
			#do start actions..
		end
	end
	...
	start_button = StartButton.new
	paust_button = PauseButton.new
这样的做法会造成大量的子类。而且如果Button类的接口发生了变化，维护代价会很高。其次，按下按钮的动作所处的层次不当：它们不是按钮的功能，而是点唱机的功能。  
而是用 block 就可以解决这些问题：

	songList = SongList.new

	class JukeboxButton < Button
		def initialize(label,&action)
			super(label)
			@action =action
		end
		def button_pressed
			@action.call(self)
		end
	end

	start_button = JukeboxButton.new("Start") {songlist.start}
	pause_button = JukeboxButton.new("Pause") {songlist.pause}

上面的实现的秘密就在initialize函数的第二个参数。 &action，那么当调用该方法是，ruby会寻找一个block,
这个block将会被转化为Proc类的一个对象，并赋值给参数。
