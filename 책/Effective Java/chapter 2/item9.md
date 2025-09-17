## try-finally 보다는 try-with-resources를 사용하라

---

코드가 간결해지고 예외 정보도 훨씬 유용

close()메소드를 호출해 직접 닫아줘야 하는 자원들
ex) I/O,Connection

단 AutoCloseable을 구현하거나 확장되있어야 함.

```java
static void copy(Sttring src, String dst) throws IOException{
  try(InputStream in = new FileInputStream(src);
    OutputStream out = new FileOutputStream(dst)){
      byte[] buf = new byte[BUFFER_SIZE];
      int n;
      while((n=in.read(buf)) >= 0){
        out.write(buf,0,n);
      }
    }
}

static String firstLineOfFile(String path, String defaultVal){
  try(BufferReader br = new BufferedReader(new FileReader(path))){
    return br.readLine();
  }catch(IOException e){
    return defaultVal;
  }
}

```
