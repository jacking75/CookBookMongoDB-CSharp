
## 검색 시 첫 번째 도큐먼트만 가져온다

```
var collection = GetDBCollection<DBBasic>("Basic");

// 기본으로는 Find 메소드는 없다. Find는 확장 메소드로 사용하고 싶다면
//using MongoDB.Driver.Core.Misc;
//using MongoDB.Driver;
//을 선언해야 한다.

// 첫 번째 값 또는 없으면 null을 반환한다.
var document = await collection.Find(x => x._id == userID).FirstOrDefaultAsync();
```


### 조건에 맞는 모든 도큐먼트를 가져온다

```
var collection = Common.GetDBCollection<DBBasic>("Basic");
var documents = await collection.Find(x=> x.Level >= level).ToListAsync();
return documents;
```


### BsonDocument를 사용하여 검색

```
var collection = Common.GetDBCollection<BsonDocument>("Basic");
var filter = new BsonDocument("_id", userID);
var documents = await collection.Find(filter).ToListAsync();

if (documents.Count > 0)
{
   return documents[0]["Level"].AsInt32;
}

return 0;
```

```
var collection = Common.GetDBCollection<DBBasic>("Basic");
var filter = new BsonDocument("Level", new BsonDocument("$gte", 2));
var documents = await collection.Find(filter).ToListAsync();
return documents;
```

```
// Builders를 사용할 때는 Collection은 BsonDocument를 사용해야 한다.

var collection = Common.GetDBCollection<BsonDocument>("Basic");

var builder = Builders<BsonDocument>.Filter;
var filter = builder.Gte("Level", 2) & builder.Eq("Money", 1000);
var documents = await collection.Find(filter).ToListAsync();

var IDList = new List<string>();

foreach (var document in documents)
{
   IDList.Add(document["_id"].AsString);
}
return IDList;
```



### 데이터 정의
- 객체 맵핑은 class만 가능하다
```
public class DBBasic
{
    public string _id; // 유저ID
    public int Level;
    public int Exp;
    public Int64 Money;
    public List<int> Costume; // 캐릭터 복장 아이템ID. 개수는 무조건 12
}
```
