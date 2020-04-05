# Essentials of URLSession



## URLSession Overview

URLSession은 HTTP 기반 요청을 처리하기 위한 클래스다.



![02-URLSession-Diagram-650x432](/Users/adorabledy/Desktop/02-URLSession-Diagram-650x432.png)



` URLSession`은 요청을 보내고 받는 핵심 개체로 `URLSessionConfiguration`을 통해 생성할 수 있다. 



### URLConfiguration 

URLSession 객체의 업로드, 다운로드 시 정책과 행동을 정의한다. 

데이터를 업로드나 다운로드 할 때 configuration 객체를 만드는 것은 가장 먼저 할 일이다. timeout 값, 캐싱 정책, 연결 요구사항과 URLSession 객체를 사용하며 의도하는 다른 타입의 정보를 구성한다.

 `URLSessionConfiguration` 의 종류

- shared : 기본 요청을 위한 세션. 

- defualt : 디스크 사용 글로벌 캐시와 인증, 쿠키 저장소 객체를 이용해 defualt configuration 객체를 생성한다.
- Ephemeral : 모든 세션 관련 데이터를 메모리에 저장한다는 점을 제외하면 default configuration과 유사. private session이라 생각하면 됨.                                                                                                                                                                                                                                                                                                                                                                
- background : 세션이 업로드와 다운로드 태스크를 백그라운드에서 수행하게 함. 애플리케이션 자체가 시스템에 의해 일시 중단되거나 종료된 경우에도 전송이 계속 됨.

구성 옵션의 전체 목록은 [Apple Docs](https://developer.apple.com/documentation/foundation/urlsessionconfiguration) 참조



`URLSessionTask` 는 dataTask를 나타내는 추상클래스다. 세션은 데이터를 가져오고 파일을 다운로드하거나 업로드하는 실제 작업을 수행하는 하나 이상의 태스크를 생성한다.





## Understanding Session Task Types

URLSessionTask 클래스는 URLSession 의 태스크에 대한 기본 클래스이다.

태스크는 항상 세션의 일부로, 태스크 생성 방법 중 하나를 인스턴스에 호출해 태스크를 생성한다. 호출하는 방법에 따라 태스크의 유형이 결정된다. 

구체적인 세션 작업에는 세가지 유형이 있다.

- URLSessionDataTask: GET 요청을 사용해 서버에서 메모리로 데이터를 가져오도록 요청을 보낸다.
- URLSessionUploadTask: POST, PUT 메소드를 통해 디스크에서 웹 서비스로 파일을 업로드 한다.
- URLSessionDownloadTask: 이 작업을 통해 리모트 서비스에서 임시 파일 위치로 파일을 다운로드 할 수 있다. 

![02-URLSession-Diagram-650x432](/Users/adorabledy/Desktop/03-Session-Tasks.png)



작업을 일시 중단, 재개 및 취소할 수도 있다. `URLSessionDownloadTask` 는 미래의 복구를 위해 일시중지할 수 있는 추가적인 능력이 있다. 

일반적으로, **URLSession은 아래의 두가지 방법으로 데이터를 리턴**한다.

- 작업이 성공적으로 끝나거나 오류가 발생한 경우, **completion handler를 통해** 
- **세션을 생성할 때 설정한 델리게이트의 메소드를 호출하면서 **





### References

- https://developer.apple.com/documentation/foundation/url_loading_system
- https://developer.apple.com/documentation/foundation/urlsession
- https://developer.apple.com/documentation/foundation/urlsessionconfiguration
- https://www.raywenderlich.com/3244963-urlsession-tutorial-getting-started



