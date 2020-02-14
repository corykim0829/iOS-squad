# The Nib Object Life Cycle

'nib' 파일이 메모리에 로드될 때, nib-loading 코드는 nib 파일의 객체가 올바르게 생성되고 초기화되도록 여러 단계를 수행합니다. 이러한 단계를 이해하면 사용자 인터페이스를 관리하기 위해 더 나은 컨트롤러 코드를 작성할 수 있습니다.


## The Object Loading Process

NSNib 또는 NSBundle의 메소드를 사용하여 nib 파일에서 객체를 로드하고 인스턴스화 할 때 기본 nib-loading 코드는 다음을 수행합니다. 

* nib 파일의 내용과 참조 된 리소스 파일을 메모리에 로드합니다.
  * 전체 'nib'객체 그래프(object graph)의 원시 데이터는 메모리에 로드되지만 아카이브(archive)되지는 않습니다.
  * nib 파일과 관련된 모든 custom image resources가 로드되어 Cocoa 이미지 캐시에 추가됩니다.

