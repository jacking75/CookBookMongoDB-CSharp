http://qiita.com/SHUAI/items/fc154ce42d6944bcedfe
http://qiita.com/SHUAI/items/e1f9bde6eb118de0fb88


## 기본 추가
- 객체 맵핑으로 추가
```
var newData = new DBBasic()
{
    _id = "test01",
    Level = 1,
    Exp = 0,
    Money = 1000,
    Costume = new List<int>(Enumerable.Repeat(0, 12)),
};

// Basic 컬렉션에 추가한다
var collection = Common.GetDBCollection<DBBasic>("Basic");
await collection.InsertOneAsync(newData);
```

- BsonDocument를 사용하여 추가.
```
Int64 money = 1000;
var Costume = new List<int>(Enumerable.Repeat(0,12));
var document = new BsonDocument { { "_id", userID }, { "Level", 1 }, { "Exp", 0 }, { "Money", money }, { "Costume", new BsonArray(Costume) } };

var collection = GetDBCollection<BsonDocument>("Basic");
await collection.InsertOneAsync(document);
```



## 한번에 복수의 document 추가
- 한번의 insert 요청에 복수의 document를 추가한다.
```
var newItemList = new List<DBUserItem>();

foreach (var itemID in ItemIDList)
{
    var newData = new DBUserItem()
    {
        _id = UniqueSeqNumberGenerator(),
        UserID = userID,
        ItemID = itemID,
        AcquireDateTime = DateTime.Now,
    };

    newItemList.Add(newData);
}

var collection = GetDBCollection<DBUserItem>("Item");
await collection.InsertManyAsync(newItemList);
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

```
public class DBUserItem
{
    public Int64 _id; // Unique ID

    public string UserID;

    public int ItemID;

    [MongoDB.Bson.Serialization.Attributes.BsonElement("AD")]
    public DateTime AcquireDateTime; // 아이템 입수 시간
}
```
