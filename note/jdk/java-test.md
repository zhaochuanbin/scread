### JAVA赋值、退出虚拟机方式；重写系统方法

	public static void main(String[] args) throws ParseException {
		int a =10;
		int b =20;
		method(10,20);
		System.out.println("a="+a);
		System.out.println("b="+b);
	}

需要在method方法被调用之后，仅打印a=100,b=200,请写出method的方法。

**考察点：**
考察基本类型赋值和对象复制，和线程虚拟机退出问题

**答案：**

**1、3种退出线程或虚拟机方法**

	private static void method(int i, int j) {
		System.out.println("a=100");
		System.out.println("b=200");
		//System.exit(0);
		//Runtime.getRuntime().exit(0);
		Thread.currentThread().stop();
	}

**2、重写println方法**

	private static void method(int i, int j) {
		PrintStream ps = new PrintStream(System.out){
			@Override
			public void println(String s){
				if(s.equals("a=10")){
					super.println("a=100");
				}
				else if(s.equals("b=20")){
					super.println("b=200");
				}else super.println(s);
			}
		};
		System.setOut(ps);
	}

### Java实现开平方根

**考察点：**
利用二分法进行逼近,jvm底层其实条用的第三方类库

	public static double sqrt (double c,double e) {
		if (c < 0) return Double.NaN;
		double m = c;
		while(abs(m - c/m) > e)
			m = (m + c/m) /2.0;
		return m;
		}
	
	public static double abs(double a){
		return a>=0.0d?a:0-a;
	}
