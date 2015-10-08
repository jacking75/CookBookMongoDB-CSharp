### MongoDB 3.0.6 Windows에서 설치 및 실행
- [MongoDB 공식 다운로드](https://www.mongodb.org/downloads#production) 에서 'Windows 64-bit 2008 R2+'를 선택해서 다운로드 한다.
    - 기본 설치 위치는 C:\Program Files\MongoDB\Server\3.0\bin
    - 실행 파일을 Path에 추가한다.
    - cmd에서 mongod --version 가 실행되면 설치는 성공이다.
- 데이터와 로그를 저장할 디렉토리를 만든다(이름은 마음대로 해도 괜찮다.)
    - 예)데이터. D:\mongodb\data
    - 예)로그.  D:\mongodb\logs
- 실행
    - cmd에서 'Mongod --dbpath D:\mongodb\data --logpath D:\mongodb\logs\mongodb.log'


### 서비스에 등록하기
- 관리자 권한으로 --install 옵션을 붙여서 MongoDB를 실행한다.
- mongod --remove로 서비스에서 제거할 수 있다.


### 설정파일 사용
- 매번 실행할 때마다 설정 옵션을 입력하기 귀찮으면 설정 파일을 사용한다.
- .conf 파일을 만든다.
- 관리자권한으로 'mongod --install --config D:\mongodb\mongodb.conf'
- conf 파일은 utf-8 로 한다.
- 설정 예

  ```
  storage:
    dbPath: D:\mongodb\data
    engine: wiredTiger
    journal:
      enabled: true

  systemLog:
    destination: file
    path: D:\mongodb\logs\mongodb.log

  net:
    bindIp: 127.0.0.1

  processManagement:
     windowsService:
        serviceName: MongoDB
        displayName: MongoDB
        description: MongoDB 서비스
  #     serviceUser: <string>
  #     servicePassword: <string>
  ```


### 참고
- 이 글은 [이것](http://qiita.com/moto_pipedo/items/c3bb40370ba2792143ad)을 정리한 것이다.
