异常、捕获和抛出
=============
[Program ruby:第八章]()
类似于 try/catch throw
在 `begin/end`中，用`rescue`语句告诉ruby希望处理的异常类型。
	
	op_file = File.open(filename,"w")
	begin
		while data = socket.read(512)
			op_file.write(data)
		end

	rescue SystemCallError
		$stderr.print "IO failed:" +＄！
		op_file.close
		File.delete(filename)
		raise
	end
####善后 ensure 
类似于 finally
	
	f=File.open("testFile")
	begin
		#..process
	rescue
		#..handle error
	ensure
		f.close unless f.nil?
	end
####引发异常
	raise
	raise "some Exception"
	raise InterfaceException ,"keyboard failure",caller

...