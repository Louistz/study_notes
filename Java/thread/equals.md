Equals
===
[Core JAVA]()

java 规范要求 equals 方法应当具有以下特性：

1. 自反性： x.equals(x)
2. 对称性： x.equals(y),y.equals(x)
3. 传递性： x.equasl(y),y.equals(z) ==> x.equals(z)
4. 一致性： 如果x,y引用的对象没有发生变化，则反复调用 x.equals(x)应当返回同样的结果
5. 对于任意的非空引用x, x.equals(null) 返回 false

编写完美`equals`的建议：

1. 显示命名参数名为 otherObject
2. if(this == otherObject) return true;   //引用同一个对象
3. if(otherObject == null) return false;
4. if(this.getClass() != otherObject.getClass()) return false;   //是否同属一个类
	如果所有的子类都拥有统一的语义，就使用 instanceof 检测： if（！otherObject instanceof ClassName） return false;
5. 将 otherObject 转换为相应的类型，然后比较属性：
	ClassName other = (ClassName) otherObject;
	return field1 == other.field1
		&& Objects.equals(field2,other.field2)
		&& ...;
6. 如果在子类中重新定义 equals,就要在其中包含调用 super.equals(other)



