
### 검색 시 첫 번째 도큐먼트만 가져온다
- 클래스 맵핑 사용     

```
var collection = GetDBCollection<DBBasic>("Basic");

// 기본으로는 Find 라는 메소드는 없다. Find는 확장 메소드로 사용하고 싶다면
//using MongoDB.Driver.Core.Misc;
//using MongoDB.Driver;
//을 선언해야 한다.

// 첫 번째 값 또는 없으면 null을 반환한다.
var document = await collection.Find(x => x._id == userID).FirstOrDefaultAsync();
```


### 조건에 맞는 모든 도큐먼트를 가져온다
```
var collection = GetDBCollection<DBBasic>("Basic");
var documents = await collection.Find(x=> x.Level >= level).ToListAsync();
return documents;
```


### BsonDocument를 사용하여 검색
- 도큐먼트의 필드를 수동으로 지정해야 한다.

```
var collection = GetDBCollection<BsonDocument>("Basic");

// useID와 동일한 도큐먼트들 검색.
var filter = new BsonDocument("_id", userID);
var documents = await collection.Find(filter).ToListAsync();

if (documents.Count > 0)
{
   return documents[0]["Level"].AsInt32; // 도큐먼트에 저장된 타입과 다르면 예외 발생
}

return 0;
```
```
var collection = GetDBCollection<DBBasic>("Basic");
// Level이 2 이상인 도큐먼트를 검색
var filter = new BsonDocument("Level", new BsonDocument("$gte", 2));
var documents = await collection.Find(filter).ToListAsync();
return documents;
```

- 복수의 조건으로 검색한다

```
// Builders를 사용할 때는 Collection은 BsonDocument를 사용해야 한다.

var collection = GetDBCollection<BsonDocument>("Basic");

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



### 특정 필드의 데이터만 가져오기
- 도큐먼트의 모든 필드의 데이터가 아닌 일부 필드의 값만 가져온다.

```
var collection = GetDBCollection<BsonDocument>("Basic");

// Level만
var documents = await collection.Find(new BsonDocument()).Project(BsonDocument.Parse("{Level:1}")).ToListAsync();

// Level, Money만
//var documents = await collection.Find(new BsonDocument()).Project(BsonDocument.Parse("{Level:1, Money:1}")).ToListAsync();
```

```
var collection = GetDBCollection<DBBasic>("Basic");
var projection = Builders<DBBasic>.Projection.Include("Exp").Include("Level");
var documents = await collection.Find(x => true).Project(projection).ToListAsync();
```


### 도큐먼트의 특정 필드를 제외한 데이터만 가져오기

```
var collection = GetDBCollection<BsonDocument>("Basic");

// Level만 제외한다
var documents = await collection.Find(new BsonDocument()).Project(BsonDocument.Parse("{Level:0}")).ToListAsync();
```


### Expression. 지정된 필드만, 필드를 다른 이름으로

```
var collection = GetDBCollection<DBBasic>("Basic");
var projection = Builders<DBBasic>.Projection.Expression(x => new { X = x.Level, Y = x.Exp });
var documents = await collection.Find(x => true).Project(projection).ToListAsync();

//var projection = Builders<Widget>.Projection.Expression(x => new { X = x.X, Y = x.Y });
//var projection = Builders<Widget>.Projection.Expression(x => new { Sum = x.X + x.Y });
//var projection = Builders<Widget>.Projection.Expression(x => new { Avg = (x.X + x.Y) / 2 });
//var projection = Builders<Widget>.Projection.Expression(x => (x.X + x.Y) / 2);
```
```
// sort + projection + skip + limt
//var userID = "jacking";
//int userLevel = 0;

var projection = BsonDocument.Parse("{Level:1, Momey:1, Exp:1}");
var sort = BsonDocument.Parse("{Level:1}");
var options = new FindOptions
{
    AllowPartialResults = true,
    BatchSize = 20,
    Comment = "funny",
    //CursorType = CursorType.TailableAwait,
    MaxTime = TimeSpan.FromSeconds(3),
    NoCursorTimeout = true
};

var collection = GetDBCollection<DBBasic>("Basic");
var documents = collection.Find(x => x.Exp >= 0, options).Project(projection)
									.Sort(sort)
									.Limit(30)
									.Skip(0).ToListAsync();

```


### Builders 사용

```
var collection = GetDBCollection<DBBasic>("Basic");

var builder = Builders<DBBasic>.Filter;
var filter = builder.Eq(x => x._id, "jacking") & builder.Lt(x => x.Money, 1100);
// or
//var filter = builder.Eq("X", 10) & builder.Lt("Y", 20);
// or
//var filter = builder.Eq("x", 10) & builder.Lt("y", 20);
// or
//var filter = Builders<DBBasic>.Filter.Where(x => x._id == "jacking" && x.Money < 1100);

// 첫 번째 값 또는 null
var document = await collection.Find(filter).FirstOrDefaultAsync();
```


### 배열 요소 조건 검색
- 도큐먼트의 필드 중 배열이 있으 때 배열 요소 조건 검사.
-
```
var collection = GetDBCollection<DBBasic>("Basic");

// Costume 리스트의 요소 중 0 보다 크거나 같은 것이 있는 도큐먼트 검색
var filter = Builders<DBBasic>.Filter.AnyGt(x => x.Costume, 0);
var documents = await collection.Find(filter).ToListAsync();

// 비동기로 검색 후 데이터를 비동기로 바로 사용
var filter = new BsonDocument("x", new BsonDocument("$gte", 100));
await collection.Find(filter).ForEachAsync(async document =>
{
	await ProcessDocumentAsync(document);
});
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
