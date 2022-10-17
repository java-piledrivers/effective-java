규칙8에서 자원반환의 안정망으로 finalizer가 제시되었는데요 
finalizer가 믿을만하지 못하기에 자바7이전에는 try-finally 문이 사용되었습니다.
(finally 구문은 예외처리 구분 상관없이 무조건 구문내의 코드를 실행합니다.)

하지만 반환해야하는 자원이 많아질수록 코드의 가독성이 떨어진다는 문제점이 있습니다.

``` java
	static void copy(String src, String dst) throws IOException {
		final int BUFFER_SIZE = 1024;
		
	    InputStream in = new FileInputStream(src);
	    try {
	        OutputStream out = new FileOutputStream(dst);
	        try {
	            byte[] buf = new byte[BUFFER_SIZE];
	            int n;
	            while ((n = in.read(buf)) >=0)
	                out.write(buf, 0, n);
	       } finally {
	           out.close();
	       }  
	   } finally {
           in.close();
      }

}
```
자바7 부터는 해당 문제를 해결할 수 있는 try-with-resources 을 지원합니다

해당 기능을 사용하려면 반환이 필요한 자원에 AutoCloseable 인터페이스를 구현한뒤
try문에 해당 자원을 명시하면 됩니다.

[예제2] try-with-resources 문을 사용했을때
```java
       
	static void copy(String src, String dst) throws IOException {
	    // 반환해야할 자원 목록을 명시
	    try (InputStream in = new FileInputStream(src);
	          OutputStream out = new FileOutputStream(dst)) {

	         byte[] buf = new byte[BUFFER_SIZE];
	         int n;
	         while ((n = in.read(buf)) >=0)
	             out.write(buf, 0, n);
	        }
	 }
```

자바 라이브러리와 서드파티 라이브러리들의 수많은 클래스와 인터페이스들이 
이미 AutoCloseable 인터페이스를 구현해두었으며, 따로 추가 구현없이 해당 기능을 사용할 수 있습니다. 

[예제3]  오라클jdk 1.8 의 AutoCloseable인터페이스를 구현한 Closeable 인터페이스

```java
package java.io;

import java.io.IOException;

public interface Closeable extends AutoCloseable {

    public void close() throws IOException;
}
```

[예제4] 오라클jdk 1.8 의 Closeable  인터페이스를 구현한 InputStream
```java
public abstract class InputStream implements Closeable {

...아래 생략
```

Closeable 인터페이스는 1.7 이전에 생긴 인터페이스로 하위호환성을 위해 유지되고 있는 인터페이스로 
현재 신규 코드등을 짤때는  AutoCloseable 인터페이스를 구현하는것을 권장되고 있습니다.
* 비록 Closeable 인터페이스도 AutoCloseable 인터페이스를 구현하는식으로 하위호환성을 유지하였기에 try-with-resources문에 사용할 수 있습니다.
* Closeable 인터페이스를 구현할경우 처리하는 Exception이 IOException만 고정됩니다. (AutoCloseable은 Exception을 throws 합니다)

