태그: #Java 
연결문서: [Java 심화 - 5](Java%20심화%20-%205.md)

# Java 심화

## 파일 입출력(I/O)

### InputStream, OutputStream

> 자바에서는 입출력을 다루기 위한 InputStream, OutputStream을 제공한다. 스트림은 단방향으로만 데이터를 전송할 수 있기에, 입력과 출력을 동시에 처리하기 위해서는 각각의 스트림이 필요하다.
> 
> 입출력 스트림은 어떤 대상을 다루느냐에 따라 종류가 나뉘며, File을 다룰 때는 FileInputStream / FileOutputStream을 사용하고, 프로세스를 다룰 때는 PipedInputStream / PipedOutputStream을 사용한다.

#### FileInputStream

-   터미널을 이용한 파일 생성
    -   code라는 문자열이 입력된 codestates.txt 라는 이름의 파일을 생성

```
echo code >> codestates.txt
```

-   파일 읽기(기본)
    -   실행할 코드와 codestates.txt 파일은 같은 디렉토리에 위치해야 함

```
import java.io.FileInputStream;

public class FileInputStreamExample {
    public static void main(String args[])
    {
        try {
            FileInputStream fileInput = new FileInputStream("codestates.txt");
            int i = 0;
            while ((i = fileInput.read()) != -1) { //fileInput.read()의 리턴값을 i에 저장한 후, 값이 -1인지 확인합니다.
                System.out.print((char)i);
            }
            fileInput.close();
        }
        catch (Exception e) {
            System.out.println(e);
        }
    }
}
```

-   파일 읽기(보조 스트림 사용)
    -   BufferedInputStream 이라는 보조 스트림을 사용하여 성능 향상
    -   버퍼란 바이트 배열로서 여러 바이트를 저장하여 한 번에 많은 양의 데이터를 입출력할 수 있또록 도와주는 임시 저장 공간

```
import java.io.FileInputStream;
import java.io.BufferedInputStream;

public class FileInputStreamExample {
    public static void main(String args[])
    {
        try {
            FileInputStream fileInput = new FileInputStream("codestates.txt");
                        BufferedInputStream bufferedInput = new BufferedInputStream(fileInput);
            int i = 0;
            while ((i = bufferedInput.read()) != -1) {
                System.out.print((char)i);
            }
            fileInput.close();
        }
        catch (Exception e) {
            System.out.println(e);
        }
    }
}
```

#### FileOutputStream

-   code라는 문자열이 입력된 codestates.txt 피알을 생성

```
import java.io.FileOutputStream;

public class FileOutputStreamExample {
    public static void main(String args[]) {
        try {
            FileOutputStream fileOutput = new FileOutputStream("codestates.txt");
            String word = "code";

            byte b[] = word.getBytes();
            fileOutput.write(b);
            fileOutput.close();
        }
        catch (Exception e) {
            System.out.println(e);
        }
    }
}
```

### FileReader / FileWriter

> File 입출력 스트림은 바이트 기반 스트림으로 입출력 단위가 1byte이다. 그러나 Java에서 char 타입은 2byte이기 때문에 자바에서는 문자 기반 스트림을 별도로 제공한다.
> 
> 문자 기반 스트림에는 FileReader와 FileWriter가 있고 문자 데이터를 다룰 때 사용한다.
> 
> 문자 기반 스트림과 그 하위 클래스는 여러 종류의 인코딩(encoding)과 자바에서 사용하는 유니코드(UTF-16)간의 변환을 자동으로 처리한다.
> 
> 문자 기반 스트림에서는 일반적으로, 바이트 기반 스트림의 InputStream이 Reader로, OutputStream이 Writer로 대응된다.
> 
> FileReader는 인코딩을 유니코드로 변환하고, FileWriter는 유니코드를 인코딩으로 변환한다.

#### FileReader

-   예제 (기본)

```
public class FileReaderExample {
    public static void main(String args[]) {
        try {
            String fileName = "codestates.txt";
            FileReader file = new FileReader(fileName);

            int data = 0;

            while((data=file.read()) != -1) {
                System.out.print((char)data);
            }
            file.close();
        }
        catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

-   예제 (보조 스트림 사용)

```
public class BufferedReaderExample {
    public static void main(String args[]) {
        try {
            String fileName = "codestates.txt";
            FileReader file = new FileReader(fileName);
            BufferedReader buffered = new BufferedReader(file);

            int data = 0;

            while((data=buffered.read()) != -1) {
                System.out.print((char)data);
            }
            file.close();
        }
        catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

#### FileWriter

-   예제

```
public class FileWriterExample {
    public static void main(String args[]) {
        try {
            String fileName = "codestates.txt";
            FileWriter writer = new FileWriter(fileName);

            String str = "written!";
            writer.write(str);
            writer.close();
        }
        catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```