ruby中的变量
==========
[Program ruby:第三章]()

每个变量保存一个对象的引用：
	
	person = "Tim"
	person.object_id -> 9446456
	person.class     -> String
	person           -> "Tim"
变量知识对象的引用。对象存在一个很大pool中（大多时候是heap），并由变量指向它们。
测试：
	person1=“Tim”
	person2=person1

	person2[0]='J'
	person2 -> "Jim"
	person1 -> "Jim"

如何避免呢，使用 String 的dup	
	
	person1="Tim"
	person2 = person1.dup
	person2[0]='J'
	person2 -> "Jim"
	person1 -> "Tim"

也可以 freeze 一个对象阻止其他人对其修改：
	
	person1="Tim"
	person2 = person1
	person1.freeze
	person2[0]='J'	#引发（raise）一个异常
	