# HMAC을 이용한 데이터 검증
API 통신을 할 때 클라이언트는 데이터를 알맞게 생성해서 서버에 전송한다.

서버는 메시지에 포함된 값이 유효한 클라이언트가 생성해서 보낸 값인지 확인해야 한다. 중간에 누군가가 데이터를 위변조해서 보낼 수 있기 때문이다.

공격에 대응하려면 메시지가 위변조되지 않았다면 것을 확인할 수단이 필요하며 이때 HMAC을 주로 사용한다.

HMAC은 Hash-based Message Authentication Code의 약자로 메시지의 무결성과 인증을 보장하기 위해 사용하는 암호화 기술이다. HMAC은 해시 함수와 비밀키를 이용해서 다음 2가지를 보장한다.

- 메시지 무결성 : 메시지가 중간에 위변조되지 않음
- 인증 : 메시지 발신자를 인증할 수 있음(발신자만 비밀 키 접근)

메시지의 발신자와 수신자는 둘만 알고 있는 비밀 키를 공유한다. 이 비밀 키는 외부에 노출되면 안된다. 메시지 발민사는 메시지를 비밀 키로 해싱해서 생성한 MAC를 원본 메시지와 함께 수신자에게 전송한다. 수신자는 수신한 메시지와 비밀 키를 이용해 MAC을 다시 생성한 뒤 발신자가 보낸 MAC과 비교한다.

두 값이 같으면 메시지가 변경되지 않았음을 보장할 수 있다. 반대로 두 값이 다르면 메시지는 유효하지 않은 것으로 판단한다.

HMAC의 장점은 단순함과 효율성에 있다. 발신자와 수신자가 비밀 키만 공유하면 정해진 해시 알고리즘을 이용해서 MAC을 생성할 수 있기 때문에 낮은 비용으로 인증 보안을 구현할 수 있다.

사용하는 해시 알고리즘에 따라 보안성동 향상된다.

단점은 비밀 키 관리다. 비밀 키가 외부에 유출되면 보안에 취약해진다. 유출 위험이 있는 만큼 비밀 키 교체도 까다로울 수 있다.

## HMAC 예제 코드
```java
public static class HMAC{
    private String secretKey;
    public HMAC(String secretKey){
        this.secretKey = secretKey;
    }

    public String hmac(String message){
        try{
            Mac mac = Mac.getInstance("HmacSHA256");
            SecretKey secretKeySpec = new SecretKeySpec(secretKey.getByte(), "HmacSHA256");
            
            mac.init(secretKeySpec);
            byte[] hash = mac.doFinal(message.getBytes("UTF-8"));
            return Base64.getEncoder().encodeToString(hash);
        }catch(Exception e){
            throw new RuntimeException(e);
        }
    }
}
```
