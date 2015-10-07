
### wiredTiger 사용 conf 파일 예

```
// mongodb_wt.conf
storage:
    dbPath: "/data/mongod_wt"
    engine: "wiredTiger"
    directoryPerDB: true
    wiredTiger:
        engineConfig:
            cacheSizeGB: 1
            directoryForIndexes: true
            statisticsLogDelaySecs: 0
        collectionConfig:
            blockCompressor: "snappy"
        indexConfig:
            prefixCompression: true
    journal:
         enabled: true
systemLog:
   destination: file
   path: "/var/log/mongodb/mongodb_wt.log"
   logAppend: true
processManagement:
    fork: true
```


### storage설정 항목 사례
- dbpath
    - 데이터베이스 파일 생성 디렉토리
- journal
    - 저널 파일 작성 여부
- directoryPerDB
    - 기본 false. 데이터베이스 마다 디렉토리를 생성 할지
- engine
    - 스토리지 엔진의 선택
    - mmapv1: 기존 스토리지 엔진
    - wiredTiger: wiredTiger 스토리지 엔진
    - 3.0 이라면 wiredTiger를 사용하는 것이 좋다



### wiredTiger 설정 항목 사례
#### engineConfig
- cacheSizeGB
    - 기본적으로 시스템 메모리의 절반 or 1GB.
    - wiredTiger에서 실제 데이터를 캐시 하기 위한 메모리 영역 지정.
    - 쓰레드와 Index도 메모리를 먹으므로 처음에는 시스템 메모리의 6~7할 정도가 좋다고 생각한다
- directoryForIndexes
    - MySQL의 innodb_file_per_table과 비슷
- statisticsLogDelaySecs
    - 기본 값은 0(출력하지 않는다)
    - wiredTiger 통계 정보 파일을 몇 초 만에 출력하까를 설정.
    - 0 으로 하면 출력하지 않는다.
    - 출력 시에 부하가 생기는 경향이 있으므로 테스트 시는 30초 정도로 설정하고 실 서비스 때는 10분에 한번이나 안 나오게 설정하는 것이 좋다.

####collectionConfig
- blockCompressor
    - 데이터 압축 여부. 기본 값은 snappy. 압축 시의 CPU 부하 등이 신경이 쓰이면 none 으로
    - none: 압축하지 않는다
    - snappy: 적당한 압축률로 고속으로 압축
    - zlib: snappy 보다 압축률은 좋지만 저속


#### indexConfig
- prefixCompression
    - 기본 True. index 압축 여부



### wiredTiger 캐시 사이즈 확인
- 현재 설정 확인
```
db.serverStatus().wiredTiger
```

- 캐시 설정
```
db.serverStatus().wiredTiger.cache["maximum bytes configured"]
```


### 참고
- http://qiita.com/kuwa_tw/items/0a5704e9e505cffeae34
- http://qiita.com/takiuchi/items/309bfa7475d1c4666389
