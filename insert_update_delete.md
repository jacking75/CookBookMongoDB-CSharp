
### 기본 추가
- 클래스 맵핑으로 새 도큐먼트 추가

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


### 한번에 복수의 document 추가
- 한번의 요청으로 복수의 document를 추가한다.

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
// 복수의 조건으로 정렬
var sort = Builders<BsonDocument>.Sort.Ascending("borough").Ascending("address.zipcode");
var result = await collection.Find(filter).Sort(sort).ToListAsync();
```


### FindOneAndReplaceAsync
- 한번에 검색&교체

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

```
var collection = GetDBCollection<BsonDocument>("Basic");

var filter = new BsonDocument("_id", "jacking3");

// _id와 Level 필드만 도큐먼트에 남게된다.
var replacement = BsonDocument.Parse("{Level: 12}");
//var projection = BsonDocument.Parse("{x: 1}");
//var sort = BsonDocument.Parse("{a: -1}");
var options = new FindOneAndReplaceOptions<BsonDocument, BsonDocument>()
{
	 IsUpsert = false, // 이것을 true로 하면 도큐먼트가 없으면 추가한다.
  	//Projection = projection,
  	//ReturnDocument = returnDocument,
  	//Sort = sort,
  	MaxTime = TimeSpan.FromSeconds(2)
};

var documents = await collection.FindOneAndReplaceAsync<BsonDocument>(filter, replacement, options, CancellationToken.None);
```


### FindOneAndUpdateAsync
- 한번에 검색&수정
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

- BsonDocume 사용
```
var findUserName = textBox5.Text;
var newNickNameList = new List<string>() { textBox1.Text, textBox6.Text };

var collection = MongoDBLib.Common.GetDBCollection<BsonDocument>("GameUser2");

var filter = new BsonDocument("_id", findUserName);
var update = Builders<BsonDocument>.Update.Set("NickNameList", new BsonArray(newNickNameList));
var options = new FindOneAndUpdateOptions<BsonDocument, BsonDocument>
{
    ReturnDocument = ReturnDocument.After
};

var document = await collection.FindOneAndUpdateAsync<BsonDocument>(filter, update, options);

if (document == null)
{
    DevLog.Write(string.Format("GameUser2:{0} 닉네임 변경 실패", findUserName));
}
else
{
    var nickList = document["NickNameList"].AsBsonArray.Select(p => p.AsString).ToList();

    DevLog.Write(string.Format("GameUser2:{0} 닉네임 변경 {1}, {2}", findUserName, nickList[0], nickList[1]));
}
```
 
- 클래스 맵핑 사용
```
var findUserName = textBox5.Text;
var newNickNameList = new List<string>() { textBox1.Text, textBox6.Text };

var collection = MongoDBLib.Common.GetDBCollection<GameUser2>("GameUser2");

var result = await collection.FindOneAndUpdateAsync<GameUser2>(
                u => u._id == findUserName, 
                Builders<GameUser2>.Update.Set("NickNameList", new BsonArray(newNickNameList)), 
    new FindOneAndUpdateOptions<GameUser2, GameUser2>
    {
        ReturnDocument = ReturnDocument.After
    });

if (result == null)
{
    DevLog.Write(string.Format("GameUser2:{0} 닉네임 변경 실패", findUserName)); 
}
else
{
    DevLog.Write(string.Format("GameUser2:{0} 닉네임 변경 {1}, {2}", findUserName, result.NickNameList[0], result.NickNameList[1]));
}
```


### 동적 기능 사용하기
- C#의 다이나믹 타입 사용

```
dynamic person = new System.Dynamic.ExpandoObject();
person.FirstName = "Jane";
person.Age = 12;
person.PetNames = new List<dynamic> { "Sherlock", "Watson" };

var collection = GetDBCollection<dynamic>("Persion");
await collection.InsertOneAsync(person);
```


### UTC 시간 보정해서 데이터 넣기
- MongoDB에 time 타입 데이터는 utc 기준으로 들어간다. 그래서 mongodb에서 보이는데 시간을 한국 시간과 같게 하려면 도큐먼트를 넣을 때 시간을 +9 한다.

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
