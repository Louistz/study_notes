标准类型
======
[Program ruby:第五章]()

##数字
ruby支持整数和浮点数。整数可以是任意长度（其最大值取决于内存）。  
一定范围内整数（通常是-2^30 ~ 2^30-1 或 -2^62 ~ 2^62-1）在内部以二进制形式存储，是`Fixnum`类的对象。
范围之外整数存储在`Bignum`类的对象中。ruby自动管理它们之间的切换

前导符号：
	
	0b  : 二进制
	0   : 八进制
	0d  : 十进制（默认）
	0x  : 十六进制
控制字符（x的control版本，`x & 0x9f`）的整数值可以使用 `?\C-x` 和 `?\cx` 生成，元字符（`x | 0x80`）可以使用 `?\M-x`
生成。 元字符和控制字符的组合是用 `?\M-\C-x`生成。 `?\\`反斜线字符的整数值
	
	?a       => 97
	?\n      => 10
	?\C-a    => 1
	?\M-a    => 255
	?\M-\C-a => 129
	?\C-?    =>  127
所有数字都是对象，并且以各种形式的消息作出响应。常用的迭代器：
	
	3.times {print "X "}                       =>  X X X
	1.upto(5) {|x| pirnt x," "}                =>  1 2 3 4 5
	99.downto(95) {|x| print x," "}            =>  99 98 97 96 95
	50.step(80,5) {|x| print x," "}            =>  50 55 60 65 70 75 80
字符转整数
	
	a = "100"
	Integer(a)     =>  100

##字符串
ruby的字符串是8比特字节的序列。字符串是String类的对象

在字符串中是用 `#{expr}` 可以把ruby代码的值放进字符串中。如果代码知识全局变量、类变量或者实例变量的话，花括号可以省略。
	
	"Seconds/day: #{24*60*60}"    ->  86400
	"#{'Ho! ' *3} Merry Christmas" -> Ho! Ho! Ho Merry Charistmas
	"This is line #$"              -> This is line 3
要进行插入替换的代码可以是一条或多条语句，而不仅仅是一个表达式。

	puts "now is #{
		def the(a)
			"the " + a
		end
		the("time")
		} for all good coders..."
	->  new is the time for all good coders...

 也可以用 `%q` 或者 `%Q` *here documents* 来构建字符串字面量。
 跟在q或Q后面的字符是分界符。读取分界符知道发现下匹配的字符，否则就是下一个相同的分界符。分界符可以是任何一个非字母数字的单字节字符.如： `[]`,`{}`,`()`,`//`等
	
	用法占位。。。
####操作字符串
字符串可能是ruby中最大的内建类，有75以上的[标准方法](http://www.ruby-doc.org/core-1.9.3/String.html#method-i-split)

 * String#chomp : 去除字符串末尾的 \n,\r,\r\n ,返回一个新的字符串
 * String#split(pattern-$;,[limit]) -> anArray 按pattern分割，以limit为限，返回一个Array
 * String#squeeze([other_str]) 去重复，留一个
 * String#scan         类似split根据模式把字符串分成几块,但scan可以指定 希望这些块匹配的模式
 * ...  

##区间（Range）
区间无处不在：1月到12月，0到9等等。ruby使用区间实现3中不同的特性： `序列（sequence）`,`条件（condition）`和`间隔（interval）`。  
Range是一种类型~！
####区间作为序列

	1..10
	'a'...'z'
	...
`to_a`把区间转为列表

	(1..10).to_a         -> [1,2,3,4,5,6,7,8,9,10]
	('bar'..'bat').to_a  -> ['bar','bas','bat']
有很多方法可供迭代

	digits=0..9
	digits.include?(5)    -> true
	digits.min            -> 0
	digits.max            -> 9
	digits.reject {|i| i<5}   -> [5,6,7,8,9]
	digits.each {|digit| dial(digit)  -> 0..9

ruby 可以根据所定义的对象来创建区间。唯一的限制就是这些对象必须返回 在序列中的下一个对象 作为对succ的响应，而且这些对象必须是可以使用`<=>`来比较的。  
`<=>`比较两个值，并根据 第一个值是否小于、等于、大于第二个值返回 -1,0,1  
下面实现一个对点唱机音量控制的例子：
	
	class VU
		include Comparable
		attr :volume
		
		def initialize(volume)     #0..9
			@volume = volume
		end
		
		def inspect
			#' * @volume
		end

		#Support for ranges
		def <=>(other)
			self.volume <=> other.volume
		end

		def succ
			rise(IndexEror, "Volume too big") if @volume >=9
			VU.new(@volume.succ)
		end
	end
因为VU实现了`succ`和`<=>`方法，因此它就可以作为区间使用了。
	
	medium_volume = VU.new(4)..VU.new(7)
	medium_volume.to_a                  -> [####,#####,######,#######]
	medium_volume.include?(VU.new(3))   -> false

####区间作为条件
区间也可以作为双向表达式来使用。表现就像某种双向开关：  
当区间的第一部分的条件为true时，它就打开，当区间的第二部分的条件为true时，它就关闭。  
如下面的例子，打印从标准输入得到的行的集合，每组的第一行包含start这个词，最后一行包含end这个词：
	
	while line = gets
		puts line if line=~ /start/ .. line=~ /end/
	end
它的表现就是：
	
	当我们的输入的第一行带了start，它就打开了，每次收入的行都会被打印出来。直到输入的某行中包含了end从而导致它被关闭

####区间作为间隔
用作间隔测试：看看一些纸是否落入区间表达的间隔内。使用 `===`
	
	(1..10) === 5       -> true
	(1..10) === 15      -> false
	(1..10) === 3.14    -> true
	('a'..'j') === 'c'  -> true


##正则表达式
	
	占位。。。。候补