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