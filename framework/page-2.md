# 스프링 STOMP

### STOMP

STOMP(Simple Text Oriented Messaging Protocol)는 원래 스크립트 언어가 메시지 브로커에 연결하기 위해 만들어졌다.

> 메시지 브로커: 메시지 발신자, 수신자 사이에서 메시지를 받아서 적절한 대상에게 전달하는 역할

stomp는 일반적으로 사용되는 메시징 패턴을 최소화한 집합을 처리하기 위해 설계되었다. http같은 복잡한 메시징 패턴보다 간단한 메시징 패턴을 사용하기위해 만들어졌다는 의미다.\
tcp, web socket 과 같은 신뢰할 수 있는 양방향 스트리밍 네트워크 프로토콜을 통해 사용 가능하다.\
stomp는 텍스트 중심의 프로토콜이지만 페이로드는 텍스트 또는 이진 데이터 모두 가능하다.\
stomp는 http프레임을 기반으로 모델리되어 있어서 http와 유사하게 보인다.

```sh
COMMAND
header1:value1
header2:value2

Body^@
```

### Spring STOMP

스프링의 stomp를 지원하는 경우 Spring Websocket 애플리케이션은 stomp의 브로커 역할을 한다.\
컨트롤러에서 메시지에 대한 처리를 수행할 수 있고, 인메모리 브로커를 구독한 사용자에게 메시지를 브로드캐스트 할 수 있다.\
stomp 전용 브로커를 작동할 수도 있다. 이 경우 spring과 브로커에 tcp 연결을 유지하고 브로커에게 메시지를 전달해 브로커가 연결된 web socket 클라이언트로 전송한다.

destination 헤더의 경우 정해진 규칙은 없고, 전적으로 애플리케이션의 설계대로 동작 방식을 정한다.\
하지만 일반적으로 `/topic/..`의 경우 publish-subscribe(일대다)를 의미하고, `/queue/..`의 경우 point-to-point(일대일)를 의미할 수 있다.

서버에서 임의의 사용자에게 메시지를 보낼 수 없고, 이용자의 subscribe 요청의 가입 id 헤더와 서버 응답 메시지의 subscription 헤더의 값이 같아야 한다.

[전문](https://stomp.github.io/stomp-specification-1.2.html)

#### Message Flow

*   메모리 `Message Broker`를 사용할 경우

    <div align="left">

    <figure><img src="../.gitbook/assets/stomp message flow nobroker.png" alt="" width="563"><figcaption></figcaption></figure>

    </div>
*   외부 `Message Broker`를 사용할 경우

    <div align="left">

    <figure><img src="../.gitbook/assets/stomp message flow withbroker.png" alt="" width="563"><figcaption></figcaption></figure>

    </div>

외부 message broker를 사용할 때는 stomp tcp를 통해 message broker에게 메시지를 전달하고 message broker가 이용자에게 메시지를 보낼 때 `StompBrokerRelay`를 사용한다.\
위 이미지의 상황에서 이용자가 `/app`에 메시지를 보낸다면 `@MessageMapping`이 존재하는 **컨트롤러 메서드**로 라우팅되고 `/topic`에 메시지를 보낸 경우 **직접 message broker**로 라우팅 된다.\
컨트롤러에서 `Broker channel`을 통해 message broker에 직접 메시지를 보낼 수 있고, message broker는 `ClinetOutboundChannel`을 통해 일치하는 구독자에게 메시지를 보낼 수 있다.\
동일한 컨트롤러가 http 요청에 응답할 수 있으므로 http post 요청을 수행한 다음 `@PostMapping` 메서드가 meesage broker에게 메시지를 전송할 수 있다.

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/portfolio");
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.setApplicationDestinationPrefixes("/app");
        registry.enableSimpleBroker("/topic");
    }
}

@Controller
public class GreetingController {

    @MessageMapping("/greeting")
    public String handle(String greeting) {
        return "[" + getTimestamp() + ": " + greeting;
    }
}
```

위 코드의 동작 흐름은

1. 이용자가 `/portfolio`에 웹 소켓 연결을 성공하면 stomp 프레임은 이 연결로 흐르기 시작한다.
2. 이용자가 `/topic/greeting`으로 subscribe 메시지를 보낸다면 수신, 디코딩이 완료되고 메시지는 `ClinetInboundChannel`로 전송된 후 구독을 저장하는 message broker로 라우팅 된다.
3. 이용자가 `/app/greeting`으로 send 메시지를 보낸 경우 `/app` 접두사는 컨트롤러로 라우팅 되는데 도움이 된다. `/app` 접두사가 제거된 나머지 부분은 `@MessageMapping` 메서드로 매핑된다.
4. 컨트롤러에서 반환 값은 기본 반환값에 기반한 페이로드와 이용자 헤더의 접두사(/app)를 message broker 접두사(/topic)로 변환 후 spring message로 변환된다. 결과 메시지는 broker channel로 전송되어 message broker에 의해 처리된다.
5. message broker는 해당 메시지의 목적지(토픽)에 구독자들을 모두 찾는다. message broker는 구독자에게 메시지 프레임을 전송한다. 이때 `ClientOutboundChannel`을 통해 메시지를 전송하고, 전송된 메시지는 stomp 프레임으로 인코딩 된다. stomp 프레임으로 인코딩 된 메시지는 실제 web socket 연결을 통해 구독자에게 전송된다.

#### Controller

`@Controller`가 선언된 컨트롤러에서 이용자의 메시지를 처리할 수 있다. `@MessageMapping`, `@SubscribeMapping`, `MessageExceptionHandler`를 메서드에 선언해야 한다.

1.  @MessageMapping&#x20;

    메시지의 목적지(destination)를 기반으로 메시지를 라우팅한다.\
    메서드 레벨, 타입(클래스) 레벨에 모두 적용 가능하다. 타입 레벨에 지정될 경우 컨트롤러 클래스 내의 모든 메서드에 공통으로 매핑된다.\
    `/thing/*`, `/thing/**` 과 같은 Ant-Style의 경로 패턴을 지정할 수 있고, `/thing/{id}`와 같은 템플릿 변수를 지정할 수 있다. `@DestinationValue`를 통해 템플릿 변수 참조가 가능하다.\
    지원 메서드

    | Method argument                                                                 | Description                                                                                                                                                                                                                                                                                                                                                                                                                                    |
    | ------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | `Message`                                                                       | For access to the complete message.                                                                                                                                                                                                                                                                                                                                                                                                            |
    | `MessageHeaders`                                                                | For access to the headers within the `Message`.                                                                                                                                                                                                                                                                                                                                                                                                |
    | `MessageHeaderAccessor`, `SimpMessageHeaderAccessor`, and `StompHeaderAccessor` | For access to the headers through typed accessor methods.                                                                                                                                                                                                                                                                                                                                                                                      |
    | `@Payload`                                                                      | <p>For access to the payload of the message, converted (for example, from JSON) by a configured <code>MessageConverter</code>.</p><p>The presence of this annotation is not required since it is, by default, assumed if no other argument is matched.</p><p>You can annotate payload arguments with <code>@jakarta.validation.Valid</code> or Spring’s <code>@Validated</code>, to have the payload arguments be automatically validated.</p> |
    | `@Header`                                                                       | For access to a specific header value — along with type conversion using an `org.springframework.core.convert.converter.Converter`, if necessary.                                                                                                                                                                                                                                                                                              |
    | `@Headers`                                                                      | For access to all headers in the message. This argument must be assignable to `java.util.Map`.                                                                                                                                                                                                                                                                                                                                                 |
    | `@DestinationVariable`                                                          | For access to template variables extracted from the message destination. Values are converted to the declared method argument type as necessary.                                                                                                                                                                                                                                                                                               |
    | `java.security.Principal`                                                       | Reflects the user logged in at the time of the WebSocket HTTP handshake.                                                                                                                                                                                                                                                                                                                                                                       |

    기본적으로 `@MessageMapping`의 반환값은 일치하는 MessageConvertor를 통해 페이로드에 직렬화 되고 broker channel에 메시지로 전송되어 구독자들에게 브로드캐스트 된다. 전송 대상은 요청 메시지의 대상에서 접두사만 브로커 접두사로 변경된다.
2.  `@SubscribeMappgin`&#x20;

    `@MessageMapping`과 유사하지만 매핑 범위를 subscribe 메시지 범위로 줄인다.\
    기본적으로 응답값이 message broker가 아닌 직접 ClientOutboundChannel로 응답한다.
3.  `@MessageExceptionHandler`&#x20;

    `@MessageMapping`에서 발생한 예외를 처리하기 위한 애노테이션.\
    애노테이션 자체에 예외 인스턴스를 선언하거나 메서드의 인수를 통해 예외를 선언할 수 있다.\
    스프링 MVC의 `@ExceptionHandler`와 유사하다.

#### 메시지 보내기

애플리케이션 어디에서든 연결된 사용자에게 메시지를 전송할 수 있다.\
모든 컴포넌트는 broker channel을 이용할 수 있지만 SimpMessagingTemplate을 사용하여 쉽게 메시지를 전송할 수 있다.

```java
@Controller
public class GreetingController {

    private SimpMessagingTemplate template;

    @Autowired
    public GreetingController(SimpMessagingTemplate template) {
        this.template = template;
    }

    @RequestMapping(path="/greetings", method=POST)
    public void greet(String greeting) {
        String text = "[" + getTimestamp() + "]:" + greeting;
        this.template.convertAndSend("/topic/greetings", text);
    }

}
```

#### 브로커 연결

외부 브로커를 사용할 경우 stomp broker relay는 브로커와 하나의 tcp 연결을 사용한다.\
웹 소켓 클라이언트와 stomp broker relay는 각각의 tcp 연결을 맺는다.

#### authentication

스프링의 stomp를 지원할 경우 일반적으로 첫 웹 소켓 연결을 맺을 때 http 요청에서 웹 소켓으로의 업그레이드 요청일 수도 있고 sockJs http 요청일 수도 있다.\
이때 http 요청에서 이미 spring security 같은 보안 컨텍스트를 거치기 때문에 이후 통신에서는 액세스할 수 있는 인증된 사용자가 존재한다. 그러므로 별도의 보안 절차를 추가할 필요는 없다.\
stomp 프레임에는 로그인, 비밀번호 헤더가 존재하긴 하지만 이는 tcp를 통한 연결을 위한 헤더이다.\
웹 소켓을 통한 stomp 통신은 이미 http 전송 단계에서 인증을 한 것으로 간주한다.\
그럼에도 불구하고 웹 소켓 통신에 인증 과정이 필요하다면 스프링 시큐리티의 [웹소켓 시큐리티](https://docs.spring.io/spring-security/reference/servlet/integrations/websocket.html)를 참고하자.

#### token authentication

jwt와 같은 token 기반 인증 과정을 수행해야 할 경우가 있다.\
하지만 웹 소켓 핸드 셰이크 절차에는 서버가 클라이언트를 인증할 수 있는 특별한 방법을 규정해놓지 않았다.\
클라이언트도 표준 인증 헤더, 쿠키만 사용할 수 있다. 토큰을 전달하는데 쿼리 매개변수로 사용할 수도 있겠지만 이때 토큰이 노출될 위험이 존재한다.\
결국 토큰 인증 절차를 수행하기 위해 두 가지 단계가 필요하다.

1. stomp 연결 시 인증 헤더 추가
2. `channelInterceptor`에서 인증 헤더로 인증 절차 수행

```java
@Configuration
@EnableWebSocketMessageBroker
public class MyConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureClientInboundChannel(ChannelRegistration registration) {
        registration.interceptors(new ChannelInterceptor() {
            @Override
            public Message<?> preSend(Message<?> message, MessageChannel channel) {
                StompHeaderAccessor accessor =
                        MessageHeaderAccessor.getAccessor(message, StompHeaderAccessor.class);
                if (StompCommand.CONNECT.equals(accessor.getCommand())) {
                    Authentication user = ... ; // access authentication header(s)
                    accessor.setUser(user);
                }
                return message;
            }
        });
    }
}
```

`channelInterceptor`의 인증 절차가 spring security의 인증 절차보다 먼저 수행되어야 잘 작동하기 때문에 `@Order(Ordered.HIGHEST_PRECEDENCE + 99)`와 같이 순서를 지정해주면 좋다.



***

#### 참고

[https://docs.spring.io/spring-framework/reference/web/websocket/stomp.html](https://docs.spring.io/spring-framework/reference/web/websocket/stomp.html)















