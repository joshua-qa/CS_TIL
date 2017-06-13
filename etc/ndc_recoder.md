## 실시간 게임 레코더 개발사례 소개

* 블랙박스는? 게임 플레이 영상을 녹화하는 애플리케이션
  * 특이점 : 상시 녹화
  * 서비스되는 게임에 한해 사전에 등록, 사전에 등록된 게임만 녹화
  * 구동되는 것을 감시하다가 실행 감지시 자동으로 녹화가 시작
  * Native DLL / 버튼 누를 필요 없이 게임 화면 항상 녹화
  * JIRA에 이슈 보고 등
* 어디에 씀?
  * 증거로서의 역할에 충실
  * 크래시 발생 시 다잉메시지 역할
  * FGT, 게임 플레이 분석, 개발자가 버그를 직접 눈으로 볼 수 있음
  * 유용한 성능 분석 자료
  * 효율적인 대화의 재료
* 사실 상시 레코딩 기능만 보면 더 나은 성능의 앱도 있지만, 따로 만든 이유
  * 레코딩을 위해 획득한 화면을 가공해서 응용할 수 있는 가능성이 있음
* 블랙박스 데모
  * 단축키 누르면 JIRA에 올릴 수 있도록 이런저런 기능 제공.. (스샷 편집이라거나 등등)
* 화면 캡쳐 -> 인코딩/먹싱 -> 파일 생성 (mp4)
  * 코덱은 H.264.. (브라우저 및 html5 호환성 고려)
  * 처음엔 Video for windows 썼는데 이런저런 단점으로 media foundation으로 변경

#### 캡쳐 소스 획득
* GDI : 성능 보장 힘듬
* 다이렉트3d 9 : 성능 보장이 힘들다거나, 어떤 창이 가린다거나 하면 게임화면이 가려져서 녹화
* 게임 직접 연동해서 백버퍼 취득 : 다수의 프로젝트 연동 불편, 커뮤니케이션 비용 증가
* windows media 9 sdk : wmv 쓸일이 없어 따로 고려하진 않았음..
* desktop duplication api : 윈8부터 지원, 성능은 우월했지만 게임 영상 촬영에는 부적합 (창이 가린다거나 하는 문제)
* 다이렉트x 버전마다 후킹 : 할 수 있다면 좋겠지만.. 유지보수 어려움, 런타임 에러 메시지 박스 녹화는?
* DWM(desktop window manager)으로부터 캡쳐 소스 획득 : 윈8 이후로는 항상 상주, DWM API를 이용해서 일부 컨트롤 가능 (순식간에 게임화면 가져올 수 있음)
  * 알트탭 누르면 활성화된 창 썸네일을 실시간 갱신
  * DXGI의 화면전송이 DWM을 통해서 벌어짐
  * 썸네일 렌더링은 되는데 캡쳐 기능은 제공하지 않음..
  * msdn에 나오지 않는 몇몇 함수로 텍스쳐를 순식간에 획득.. dwmgetdxsharedsurface()
  * dx7로 돌아가는 클래식 게임도 레코딩 가능해지는 현상 발견

* 모바일도 가능할거라는 가능성 추측

* 오디오 캡쳐는 WASAPI

#### JIRA와 통합
* 그냥 인앱 브라우저에 띄워줌
  * 내장 크로미움 브라우저에 띄우되, 자동 로그인 해주고 이슈 작성 화면까지 이동
  * 웹페이지의 보고 버튼을 누르면 블랙박스 애플리케이션이 상황에 맞는 작업 (URL 획득이라거나..)
  * REST API, JQL 활용