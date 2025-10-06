# Item 9. try-finally 보다는 try-with-resources를 사용하라.

자바 라이브러리에는 `close` 메서드를 호출해 직접 닫아줘야 하는 자원들이 많다.

- InputStream
- OutputStream
- java.sql.Connection

자원 닫기는 클라이언트가 놓치기 쉽기 때문에 예측할 수 없는 성능 문제로 이어지기도 한다.

이런 자원 중 상당수가 안전망으로 `finalizer`를 사용하고 있지만, `finalizer`는 믿을만 하지 못하다.

## try-finally의 문제점

전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 `try-finally`가 쓰였다.

```java
static String firstLineOfFile(String path) throws IOException {
	BufferedReader br = new BufferedReader(new FileReader(path));
	try {
		return br.readLine();
	} finally {
		br.close();
	}
}
```

만약 자원을 두 개 사용한다면 어떨까?

```java
static void copy(String src, String dst) throws IOException {
	InputStream in = new FileInputStream(src);
	try {
		OutputStream out = new FileOutputStream(dst);
		try {
			byte[] buf = new byte[BUFFER_SIZE];
			int n;
			while ((n = in.read(buf)) >= 0)
				out.write(buf, 0, n);
		} finally {
			out.close();
		}
	} finally {
		in.close();
	}
}
```

위처럼 자원이 둘 이상이면 `try-finally` 방식은 너무 지저분하다.

또한 실제 시스템에서의 디버깅을 몹시 어렵게 한다.

## try-with-resources

`try-with-resources`는 자바 7에서 시작되었으며, `try-finally`의 문제점을 모두 해결할 수 있다.

```java
static String firstLineOfFile(String path) throws IOException {
	try (BufferedReader br = new BufferedReader(
				new FileReader(path))) {
			return br.readLine();		
		}
}
```

```java
static void copy(String src, String dst) throws IOException {
	try (InputStream in = new FileInputStream(src);
			OutputStream out = new FileOutputStream(dst)) {
		byte[] buf = new byte[BUFFER_SIZE];
		int n;
		while ((n = in.read(buf)) >= 0)
			out.write(buf, 0, n);
	}
}
```

`try-finally`보다 `try-with-resources`가 훨씬 간결하다.

<br>

또한 보통의 `try-finally`와 같이 `catch` 절을 사용할 수 있다.

```java
static String firstLineOfFile(String path, String defaultVal) {
	try (BufferedReader br = new BufferedReader(
			new FileReader(path))) {
		return br.readLine();		
	} catch (IOException e) {
		return defaultVal;
	}
}
```