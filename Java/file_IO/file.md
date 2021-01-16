## 파일 입출력
---
### 파일 생성
```
 File dir = new File("C:/Temp2/Dir");
 File file = new File(new URI("file:///C:/Temp2/file3.txt"));
 if(!dir.exists()){
            dir.mkdirs();
        }
if(!file.exists()){
    file.createNewFile();
 }
```
- 파일 생성은 new File("파일경로")로 생성한다. 
- 파일 생성할 때 파일이 우선 존재하는 지 exists()으로 확인을 하고 createNewFile()로 생성한다.
- 디렉토리를 생성할 때 mkdirs()을 사용하면 경로에 없는 디렉토리 모두 생성한다.
### 디렉토리에 있는 파일 조회
```
File temp= new File("C:/Temp2");
File[] contents=temp.listFiles();
```
- contents에 있는 file들을 반복문을 사용해서 length()로 파일의 크기를 구할 수 있고 getName()을 사용해서 file 이름을 알 수 있다.

### FileInputStream
- 파일로부터 바이트 단위로 읽어 들일 때 사용한다.
- 모든 종류의 파일을 읽을 수 있다.
```
FileInputStream fis=new FileInputStream("파일경로");
```
- read() 사용으로 한 바이트씩 읽을 수 있다.
```
 while((data=fis.read())!=-1){  
    System.out.write(data);
}
```
### FileOutputStream
- 모든 종류의 파일을 복사하기 위해서는 바이트기반으로 해야한다.
- 파일 복사하기
```
     String originalFileName = "복사할 파일경로";
     String targetFileName = "복사한 파일이 위치할 경로";

     FileInputStream fis = new FileInputStream(originalFileName);
     FileOutputStream fos = new FileOutputStream(targetFileName);

     int readByteNo;
     byte[] readBytes = new byte[100];
     while ((readByteNo = fis.read(readBytes)) != -1) {
        fos.write(readBytes, 0, readByteNo);
     }
     fos.flush(); //파일에 FileOutputStream을 사용하여 파일에 추가하였다면 flush를 해야한다.
    ```
### FileReader
- 텍스트 파일로부터 데이터를 읽어 들일 때 사용한다.
```
 FileReader fr = new FileReader("읽어들일 파일 경로");
 int readCharNo;
 char[] cbuf = new char[100];
 while ((readCharNo = fr.read(cbuf)) != -1) {
     String data = new String(cbuf, 0, readCharNo);
     System.out.println(data);
 }
 fr.close();
```

### FileWriter
- 텍스트 파일에 문자 데이터를 저장할 때 사용
- write() 사용하여 데이터를 추가한다.
```
 File file = new File("저장할 파일 경로");
 FileWriter fw = new FileWriter(file);
 fw.write("문자 데이터 저장");
 fw.flush();
```