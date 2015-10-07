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


### 정렬

```
var collection = GetDBCollection<DBBasic>("Basic");

// 내림차순
//var documents = await collection.Find(x=> x.Level >= 1).SortByDescending(d => d.Level).FirstOrDefaultAsync();
// or
//var documents = await collection.Find(x=> x.Level >= 1).SortByDescending(d => d.Level).FirstOrDefaultAsync();
// or
//var documents = await collection.Find(x=> x.Level >= 1).SortByDescending(d => d.Level).ToListAsync();
// or 조건 없이
var documents = await collection.Find(x => true).SortByDescending(d => d.Level).ToListAsync();
```

```
var collection = GetDBCollection<BsonDocument>("Basic");

// 올림 차순
//var documents = await collection.Find(new BsonDocument()).Sort(new BsonDocument("$Level", 1)).ToListAsync();

// 내림 차순
//var documents = await collection.Find(new BsonDocument()).Sort(new BsonDocument("Level", -1)).ToListAsync();
// or
var documents = await collection.Find(new BsonDocument()).Sort("{Level: -1}").ToListAsync();
```

```
var sort = Builders<BsonDocument>.Sort.Ascending("borough").Ascending("address.zipcode");
var result = await collection.Find(filter).Sort(sort).ToListAsync();
```



#### FindOneAndReplaceAsync

```
var collection = GetDBCollection<DBBasic>("Basic");

var newData = new DBBasic()
{
_id = "jacking3",
Money = 3333,
Costume = new List<int>(Enumerable.Repeat(0, 12))
};

// 변경 되기 이전 값을 반환한다. 실패하면 null
var documents = await collection.FindOneAndReplaceAsync(x => x._id == "jacking5", newData);
```


#### FindOneAndReplaceAsync

```
var collection = GetDBCollection<BsonDocument>("Basic");

var filter = new BsonDocument("_id", "jacking3");

// _id와 Level 필드만 도큐먼트에 남게된다.
var replacement = BsonDocument.Parse("{Level: 12}");
//var projection = BsonDocument.Parse("{x: 1}");
//var sort = BsonDocument.Parse("{a: -1}");
var options = new FindOneAndReplaceOptions<BsonDocument, BsonDocument>()
{
	IsUpsert = false,
  	//Projection = projection,
  	//ReturnDocument = returnDocument,
  	//Sort = sort,
  	MaxTime = TimeSpan.FromSeconds(2)
};

var documents = await collection.FindOneAndReplaceAsync<BsonDocument>(filter, replacement, options, CancellationToken.None);
```


#### FindOneAndUpdateAsync

```
var collection = GetDBCollection<BsonDocument>("Basic");

var filter = new BsonDocument("_id", "jacking3");
var update = BsonDocument.Parse("{$set: {Level: 3}}");
//var projection = BsonDocument.Parse("{x: 1}");
//var sort = BsonDocument.Parse("{a: -1}");
var options = new FindOneAndUpdateOptions<BsonDocument, BsonDocument>()
{
  	//IsUpsert = isUpsert,
  	//Projection = projection,
  	//ReturnDocument = returnDocument,
  	//Sort = sort,
  	MaxTime = TimeSpan.FromSeconds(2)
};


var document = await collection.FindOneAndUpdateAsync<BsonDocument>(filter, update, options, CancellationToken.None);
```


#### 동적 기능 사용하기

```
dynamic person = new System.Dynamic.ExpandoObject();
person.FirstName = "Jane";
person.Age = 12;
person.PetNames = new List<dynamic> { "Sherlock", "Watson" };

var collection = GetDBCollection<dynamic>("Persion");
await collection.InsertOneAsync(person);
```

#### UTC 시간 보정해서 데이터 넣기

```
var newData = new DBTimeData()
{
	CurTime = DateTime.Now.AddHours(9)
};

var collection = GetDBCollection<DBTimeData>("TimeData");
await collection.InsertOneAsync(newData);
```


### Update
#### 기본 업데이트

```
// UpdateOneAsync 는 하나만, 복수는 UpdateManyAsync
var collection = GetDBCollection<DBUserSkill>("Skill");

var userID = "jacking";

var result = await collection.UpdateOneAsync(x => x._id == userID,
								Builders<DBUserSkill>.Update.Set(x => x.Value, 14));
// result.MatchedCount 가 1 보다 작으면 업데이트 한 것이 없음
```


#### 도큐먼트의 내부 도큐먼트를 변경할 때

```
var collection = GetDBCollection<DBUserSkill>("Skill");

var userID = "jacking";
int skillItemID = 1;
var newInfo = new SkillItemInfo() { Value = 101 };


var filter = Builders<DBUserSkill>.Filter.And(Builders<DBUserSkill>.Filter.Eq(x => x._id, userID),
  											Builders<DBUserSkill>.Filter.ElemMatch(x => x.SkillItems, x => x.ID == skillItemID));

var update = Builders<DBUserSkill>.Update.Set("SkillItems.$.Info", newInfo);
//or
//var update = Builders<DBUserSkill>.Update.Set(x => x.SkillItems.ElementAt(-1).Info, newInfo);

collection.UpdateOneAsync(filter, update);
```


### Replace

```
var collection = GetDBCollection<DBBasic>("Basic");

var userID = "jacking";
var documents = await collection.Find(x=> x._id == userID).SingleAsync();

if( documents != null)
{
documents.Level = documents.Level + 3;
var result = await collection.ReplaceOneAsync(x => x._id == documents._id, documents);
```

```
var tom = await collection.Find(x => x.Id == ObjectId.Parse("550c4aa98e59471bddf68eef"))
	.SingleAsync();

tom.Name = "Thomas";
tom.Age = 43;
tom.Profession = "Hacker";

var result = await collection.ReplaceOneAsync(x => x.Id == tom.Id, tom);
```



### Delete
#### 삭제. 한번에 복수개를 지울 때는 DeleteManyAsync
```
var collection = GetDBCollection<DBBasic>("Basic");

var userID = "jacking4";
var result = await collection.DeleteOneAsync(x=> x._id == userID);
```
