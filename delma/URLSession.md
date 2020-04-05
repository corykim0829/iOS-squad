# URLSession

관련된 네트워크 데이터 전송 태스크의 그룹을 조정하는 객체



### Declaration

```swift
class URLSession: NSObject
```



### Overview

URLSession 클래스와 관련된 클래스는 URL로 표시된 엔드포인트에서 데이터를 다운로드 하고 업로드하기 위해 API를 제공한다. 또한 앱이 실행중이 아니거나 일시 중단된 상태에서 이 API를 사용해 백그라운드 다운로드를 수행할 수 있다. 관련된 URLSessionDelegate 및 URLSessionTaskDelegate를 사용해 인증을 지원하고 리디렉션 및 태스트 완료와 같은 이벤트를 수신할 수 있다. 



> Note
>
> URLSession API는 꽤 복잡한 방식으로 함께 작동하는 많은 다른 클래스를 포함하며, 참조 문서를 독자적으로 읽으면 분명하지 않을 수 있다. API를 사용하기 전에, URL Loading System 항목을 읽어보라. Essentials, Uploading and Downloading 섹션의 글은 URLSession을 사용하여 일반적인 작업을 수행하는 예를 제공한다. 



데이터 전송 태스크와 관련된 그룹을 조정하는 각각의 URLSession 인스턴스를 하나 이상 생성한다. 예를들어, 웹 브라우저를 만든다면, 앱이 탭이나 윈도우 세션 혹은 인터랙티브 사용을 위한 세션과 백그라운드 다운로드를 위한 다른 세션을 만들 수 있다. 각각의 세션에서, 앱은 일련의 작업을 추가하며 **각 태스크는 특정 URL에 대한 요청을 나타낸다**(필요한 경우 HTTP 리디렉션에 따름)



### Types of URL Sessions

지정된 URL 세션 내의 태스크는 **하나의 호스트에 대해 수행할 최대 동시 연결 수**, **연결이 셀룰러 네트워크를 사용할 수 있는지 여부** 등과 같은 연결 동작을 정의하는 공통 세션 구성 개체를 공유한다. 

URLSession은 기본 요청에 대한 싱글톤 `shared`(구성 개체 없음) 세션이 있다. 사용자가 생성하는 세션만큼 커스터마이징 할 수 있는 건 아니지만, 요구사항이 매우 제한적인 경우 좋은 출발점으로 작용할 수 있다. shared class method를 호출함으로 세션에 접근한다. 다른 종류의 세션에서, 다음 세가지 구성 중 하나를 사용해 URLSession을 생성한다. 

- 디폴트 세션은 shared 세션과 매우 유사하게 동작하지만, 이를 구성할 수 있다. (커스터마이징 할 수 있다?) 또한 디폴트 세션에 델리게이트를 할당해 데이터를 점진적으로 가져올 수 있다.
- 수명이 짧은(ephemeral) 세션은 shared 세션과 유사하지만 캐시, 쿠키나 credential을 디스크에 쓰지 말아라. 
- background 세션은 앱이 실행되지 않는 동안 백그라운드에서 컨텐츠 업로드 및 다운로드를 수행할 수 있다. 

각 구성 유형을 생성하는 자세한 내용은 [Creating a Session Configuration Object](https://developer.apple.com/documentation/foundation/urlsessionconfiguration#1660412) , [URLSession Configuration](https://developer.apple.com/documentation/foundation/urlsessionconfiguration) 을 참고





### Types of URL Session Tasks

세션 내에서 데이터를 서버로 선택적으로 업로드한 다음 서버에서 디스크의 파일 또는 메모리에 있는 하나 이상의 NSData 객체로 데이터를 검색하는 태스크를 생성한다. URLSession API는 네가지 유형의 태스크를 제공한다. 

- Data tasks : NSData 객체를 사용해 데이터를 주고받는다. 데이터 태스크는 서버에 대한 짧고 빈번한 인터랙티브 요청을 위한것.
- Upload tasks: 데이터 태스크와 비슷하지만 데이터(흔히 파일형태로 표시됨)도 보내고, 앱이 실행되지 않는 동안 background 업로드를 지원한다. 
- Download tasks: 파일 형식으로 데이터를 검색하고, 앱이 실행되지 않는 동안 백그라운드 다운로드 및 업로드를 지원한다.
- WebSocket tasks: RFC 6455에 정의된 WebSocket 프로토콜을 사용해 TCP 및 TLS를 통해 데이터를 교환한다.



### Using a Session Delegate

세션의 태스크는 또한 공통의 델리게이트 객체를 공유한다. 이 델리게이트를 구현해 아래와 같은 다양한 이벤트가 발생할 때 정보를 얻을 수 있다.

- 인증 실패시
- 서버로부터 데이터 도착시
- 데이터를 캐싱에 사용할 때

델리게이트가 제공하는 기능이 필요하지 않은 경우, 세션을 만들 때 nil을 전달하는 것을 제공받지 않고도 해당 API를 사용할 수 있다.

> Important
>
> 세션 객체는 앱이 세션을 종료하거나 명시적으로 세션을 무효화 할 때까지 델리게이트에 대한 strong reference를 유지한다. 세션을 무효화하지 않으면 앱이 종료될때까지 메모리가 유출된다.



### Asynchronicity and URL Sessions

대부분의 네트워킹 API와 마찬가지로 URLSession API도 비동기성이 매우 높다. 다음 두가지 방법 중 하나로 데이터를 앱에 반환한다.

- 전송이 성공적으로 완료되거나 오류가 발생했을 때 completion handler block을 호출
- 데이터가 도착하고 전송이 완료될 때 세션의 델리게이트 메소드를 호출

URLSession은 이러한 정보를 델리게이트에게 전달하는 것 외에도(언제나 그 상태가 변경될 수 있다는 주의사항과 함께) 태스크의 현재상태에 기초해 프로그래밍 방식의 의사 결정을 해야하는 경우 쿼리할 수 있는 상태 및 progress properties를 제공한다. 



### Protocol Support

URLSession 클래스는 사용자의 시스템 기본 설정에 구성한대로 프록시 서버와 SOCKS 게이트웨이를 지원하며 data, file, ftp, http, https URL schemes 를 기본적으로 지원한다. 

URLSession은 HTTP/1.1, HTTP/2 프로토콜을 지원한다. [RFC 7540](https://tools.ietf.org/html/rfc7540)에서 설명한 HTTP/2 지원은 ALPN(Application-Layer Protocol Trade)을 지원하는 서버를 필요로 한다. 

또한 URLProtocold을 서브클래싱해 사용자화 네트워킹 프로토콜 및 URL 구성(앱 전용)에 대한 지원을 추가할 수 있다. 



### App Transport Security(ATS)

iOS 9.0 및 macOS 10.11 이상에서는 URLSession을 통해 만들어진 모든 HTTP 연결에 App Transport Security(ATS)를 사용한다. ATS는 HTTP 연결이 HTTPS([RFC2818](https://tools.ietf.org/html/rfc2818))을 사용하도록 요구한다.

더 많은 정보는 [NSAppTransportSecurity](https://developer.apple.com/documentation/bundleresources/information_property_list/nsapptransportsecurity)



### Foundation Copying Behavior

세션 및 태스크 객체는 다음과 같이 [NSCopying](https://developer.apple.com/documentation/foundation/nscopying) 프로토콜을 준수한다. 

- 앱이 세션이나 작업 객체를 복사하면 동일한 객체를 다시 얻는다.

- 앱이 구성 객체를 복사할 때 독립적으로 수정할 수 있는 새 복사본을 얻는다.

  

### Thread Safety

The URLSession API is thread-safe. 어떤 쓰레드 컨텍스트에서도 세션 및 태스크를 자유롭게 생성할 수 있다. 델리게이트가 제공한 completion handler를 호출하면 작업이 올바른 델리게이트 큐에 자동적으로 스케쥴된다. 







### Reference
https://developer.apple.com/documentation/foundation/urlsession







