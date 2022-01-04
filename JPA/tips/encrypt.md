### 개인정보 암호화 하기
- 프로젝트 과정에서 개인정보를 암호화하자는 의견이 있어서 이메일과 전화번호와 같은 개인정보는 데이터베이스에 암호화하여 저장하기로 하였다.
- 비밀번호에 경우 spring security가 지원하는 PasswordEncoder를 사용할 수 있지만 단방향암호화라서 이 방법은 값을 조회해 올 수가 없었다. 이메일과 전화번호같은경우에는 조회할 수 있어야하기 때문에 string값을 암호화하고 조회할 때는 복호화하여 조회해야한다.
- spring에서 yml파일로 key값을 관리할 수 있는 암호화 방법을 사용하는 것이 관리하기 좋고 보안에도 좋을 것 같아서 해당 방법을 찾아봤다.

### JPA Attribute Converter 사용하기
```
public class User {
    ...

    @Convert(converter = CryptoConverter.class)
    private String email;

    @Convert(converter = CryptoConverter.class)
    private String phoneNum;

    ...
}
```
- 위의 코드처럼 @Convert(converter = CryptoConverter.class) 사용하여 조회할 때 저장될 때 암호화 복호화를 자동으로 해줄 수 있다.
```
@Converter
public class CryptoConverter implements AttributeConverter<String, String> {
    private static final String ALGORITHM = "AES/ECB/PKCS5Padding";
    @Value("${crypto.key}")
    private String key;

    @Override
    public String convertToDatabaseColumn(String attribute) {
        if (attribute == null) {
            return null;
        }
        byte[] KEY = key.getBytes();
        Key key = new SecretKeySpec(KEY, "AES");
        try {
            Cipher cipher = Cipher.getInstance(ALGORITHM);
            cipher.init(Cipher.ENCRYPT_MODE, key);
            return new String(Base64.getEncoder().encode(cipher.doFinal(attribute.getBytes())));
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    @Override
    public String convertToEntityAttribute(String dbData) {
        if (dbData == null) {
            return null;
        }
        byte[] KEY = key.getBytes();
        Key key = new SecretKeySpec(KEY, "AES");
        try {
            Cipher cipher = Cipher.getInstance(ALGORITHM);
            cipher.init(Cipher.DECRYPT_MODE, key);
            return new String(cipher.doFinal(Base64.getDecoder().decode(dbData)));
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```
- CryptoConverter 클래스를 만들어 AttributeConverter<?,?> 의 메서드를 오버라이딩하여 사용할 수 있다. 이렇게 하면 User 엔티티에 @Convert(converter = CryptoConverter.class) 사용한 컬럼들이 해당 클래스를 사용하여 DB에 저장될 때는 암호화되고 조회할 때는 복호화되어 조회되도록 할 수 있다.

### AES 알고리즘이란???
- AES는 고급 암호화 표준이라는 의미로 암호화 및 복호화 시 동일한 키가 사용되는 대칭키 알고리즘이다.

- Secret Key
    - Secret Key는 평문을 암호화하는 데 사용되며 외부에 노출되어서는 안된다.
    - AES 종류에 따라 Secret Key의 길이가 달라진다.

- Block Cipher
    - AES는 16바이트의 고정도니 블록 단위로 암호화를 수행한다. 암호화를 수행할 때  Block Cipher Mode 를 선택할 수 있으며 CBC,ECB 등이 있다. 권장하는 방식은 CBC 방식이다.
    - AES는 16바이트의 블록단위로 암호화를 수행하는데 16바이트보다 작은 블록이 생길 경우 부족한 부분을 특정 값으로 채워야한다. 이러한 작업을 패딩이라고 하며 PKCS5, PCKS7 방식이 있다.

### 같은 값은 암호화 값이 같은 이슈 발생
- AES 16바이트의 고정된 블록 단위로 암호화를 수행하는데 CBC는 블록을 그대로 암호화하지 않고 이전에 암호화했던 블록과 XOR 연산을 한 뒤 암호화를 수행한다. 첫번째 블록은 암호화 블록이 없기 때문에 IV(initialization vector)를 이용한다. IV도 16바이트 크기를 가져야하고 IV를 매번 다르게 생성하면 같은 평문이라도 다른 암호문을 생성할 수 있다.
- 지금까지 매번 같은 IV를 사용하고 있었기 때문에 같은 평문이면 같은 암호문이 나온 것이다.
- 매번 IV를 다르게 하면 복호화할 때도 IV를 알 수 있어야 하기 때문에 IV도 암호문속에 같이 저장해둔다.
- 암호화를 할 때 IV를 매번 다르게 생성하고 암호화한 값 앞에 16자리를 IV로 저장한다. 복호화할 때는 앞에 16자리를 IV로 사용하여 복호화하면 문제를 해결할 수 있다.