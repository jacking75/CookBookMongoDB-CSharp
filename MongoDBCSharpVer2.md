


###MongoDB 공식 C# Driver
- 1.x와 2.x 라이브러리가 많이 다름.
- [2.0 드라이버 소개](https://www.mongodb.com/blog/post/introducing-20-net-driver)
- [레퍼런스](http://mongodb.github.io/mongo-csharp-driver/2.0/reference/)
- [온라인 도움 문서](http://api.mongodb.org/csharp/2.0/html/R_Project_CSharpDriverDocs.htm)
- [Indexes.CreateOneAsync sample](http://qiita.com/SHUAI/items/f9479f2c1a09bbd7e36b)
- [Find & FindAsync sample](http://qiita.com/SHUAI/items/ace6f6d3f08bb8dd79c5)


### collection 객체 얻기

```
public static IMongoCollection<T> GetMongoDBCollectionVer2<T>(string connectString, string dbName, string collectionName)
{
   var mongoClient = new MongoClient(connectString);
   if (mongoClient == null)
   {
       return null;
   }

   var collection = mongoClient.GetDatabase(dbName).GetCollection<T>(collectionName);
   return collection;
}
```


### Find
#### 데이터 하나만 가져온다
```
try
{
	var collection = MyExtensions.GetMongoDBCollectionVer2<DBBasic>("mongodb://172.20.60.221", "TestDB", "Basic");

	// 기본으로는 Find 메소드는 없다. Find는 확장 메소드로 사용하고 싶다면
 	//using MongoDB.Driver.Core.Misc;
 	//using MongoDB.Driver;
 	//을 선언해야 한다.

 	// 첫 번째 값 또는 null
 	var document = await collection.Find(x => x._id == "jacking").FirstOrDefaultAsync();
 	document.Dump();
}
catch (Exception ex)
{
	ex.Message.Dump();
}
```

#### BsonDocument를 사용한 조건 쿼리
```
try
{
	var userID = "jacking";
	int userLevel = 0;

	var collection = MyExtensions.GetMongoDBCollectionVer2<BsonDocument>("mongodb://172.20.60.221", "TestDB", "Basic");
 	var filter = new BsonDocument("_id", userID);
 	var documents = await collection.Find(filter).ToListAsync();

 	if (documents.Count > 0)
 	{
    	userLevel = documents[0]["Level"].AsInt32;
	}

	userLevel.Dump();
}
catch (Exception ex)
{
 	ex.Message.Dump();
}
```

#### BsonDocument를 사용한 조건 쿼리 2
```
try
{
 	var collection = MyExtensions.GetMongoDBCollectionVer2<BsonDocument>("mongodb://172.20.60.221", "TestDB", "Basic");

 	var filter = new BsonDocument("Level", new BsonDocument("$gte", 2));
 	var documents = await collection.Find(filter).ToListAsync();
 	documents.Dump();
}
catch (Exception ex)
{
 	ex.Message.Dump();
}
```

#### BsonDocument를 사용한 조건 쿼리 3
```
try
{
	// Builders를 사용할 때는 Collection은 BsonDocument를 사용해야 한다.

 	var collection = MyExtensions.GetMongoDBCollectionVer2<BsonDocument>("mongodb://172.20.60.221", "TestDB", "Basic");

 	var builder = Builders<BsonDocument>.Filter;
 	var filter = builder.Gte("Level", 2) & builder.Eq("Money", 1000);
 	var documents = await collection.Find(filter).ToListAsync();

 	var IDList = new List<string>();

 	foreach (var document in documents)
 	{
    	IDList.Add(document["_id"].AsString);
 	}

 	IDList.Dump();
}
catch (Exception ex)
{
 	ex.Message.Dump();
}
```

#### 클래스를 사용한 조건 쿼리
```
try
{
 	var collection = MyExtensions.GetMongoDBCollectionVer2<DBBasic>("mongodb://172.20.60.221", "TestDB", "Basic");
 	var documents = await collection.Find(x=> x.Level >= 2).ToListAsync();
 	documents.Dump();
}
catch (Exception ex)
{
 	ex.Message.Dump();
}
```

#### 정렬
```
try
{
 	var collection = MyExtensions.GetMongoDBCollectionVer2<DBBasic>("mongodb://172.20.60.221", "TestDB", "Basic");

	// 내림차순
	//var documents = await collection.Find(x=> x.Level >= 1).SortByDescending(d => d.Level).FirstOrDefaultAsync();
	// or
	//var documents = await collection.Find(x=> x.Level >= 1).SortByDescending(d => d.Level).FirstOrDefaultAsync();
	// or
	//var documents = await collection.Find(x=> x.Level >= 1).SortByDescending(d => d.Level).ToListAsync();
	// or 조건 없이
	var documents = await collection.Find(x => true).SortByDescending(d => d.Level).ToListAsync();
 	documents.Dump();
}
catch (Exception ex)
{
 	ex.Message.Dump();
}
```

#### 정렬 2
```
try
{
 	var collection = MyExtensions.GetMongoDBCollectionVer2<BsonDocument>("mongodb://172.20.60.221", "TestDB", "Basic");

	// 올림 차순
	//var documents = await collection.Find(new BsonDocument()).Sort(new BsonDocument("$Level", 1)).ToListAsync();

	// 내림 차순
	//var documents = await collection.Find(new BsonDocument()).Sort(new BsonDocument("Level", -1)).ToListAsync();
	// or
	var documents = await collection.Find(new BsonDocument()).Sort("{Level: -1}").ToListAsync();
 	documents.Dump();
}
catch (Exception ex)
{
 	ex.Message.Dump();
}
```

#### 정렬 3
```
var sort = Builders<BsonDocument>.Sort.Ascending("borough").Ascending("address.zipcode");
var result = await collection.Find(filter).Sort(sort).ToListAsync();
```

#### 특정 필드의 데이터만 가져오기
```
try
{
 	var collection = MyExtensions.GetMongoDBCollectionVer2<BsonDocument>("mongodb://172.20.60.221", "TestDB", "Basic");

	// Level만
	var documents = await collection.Find(new BsonDocument()).Project(BsonDocument.Parse("{Level:1}")).ToListAsync();
	// Level, Money만
 	//var documents = await collection.Find(new BsonDocument()).Project(BsonDocument.Parse("{Level:1, Money:1}")).ToListAsync();

	documents.Dump();
}
catch (Exception ex)
{
 	ex.Message.Dump();
}
```

#### 특정 필드를 가져오기
```
try
{
 	var collection = MyExtensions.GetMongoDBCollectionVer2<DBBasic>("mongodb://172.20.60.221", "TestDB", "Basic");

	var projection = Builders<DBBasic>.Projection.Include("Exp").Include("Level");
	var documents = await collection.Find(x => true).Project(projection).ToListAsync();
	documents.Dump();
}
catch (Exception ex)
{
 	ex.Message.Dump();
}
```

#### 특정 필드를 제외한 데이터만 가져오기
```
try
{
 	var collection = MyExtensions.GetMongoDBCollectionVer2<BsonDocument>("mongodb://172.20.60.221", "TestDB", "Basic");

	// Level만
	var documents = await collection.Find(new BsonDocument()).Project(BsonDocument.Parse("{Level:0}")).ToListAsync();
	documents.Dump();
}
catch (Exception ex)
{
 	ex.Message.Dump();
}
```

#### Expression. 지정된 필드만, 필드는 다른 이름으로
```
try
{
 	var collection = MyExtensions.GetMongoDBCollectionVer2<DBBasic>("mongodb://172.20.60.221", "TestDB", "Basic");

	var projection = Builders<DBBasic>.Projection.Expression(x => new { X = x.Level, Y = x.Exp });
	var documents = await collection.Find(x => true).Project(projection).ToListAsync();
	documents.Dump();
}
catch (Exception ex)
{
 	ex.Message.Dump();
}

//var projection = Builders<Widget>.Projection.Expression(x => new { X = x.X, Y = x.Y });
//var projection = Builders<Widget>.Projection.Expression(x => new { Sum = x.X + x.Y });
//var projection = Builders<Widget>.Projection.Expression(x => new { Avg = (x.X + x.Y) / 2 });
//var projection = Builders<Widget>.Projection.Expression(x => (x.X + x.Y) / 2);

// sort + projection + skip + limt
try
{
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

    var collection = MyExtensions.GetMongoDBCollectionVer2<DBBasic>("mongodb://172.20.60.221", "TestDB", "Basic");
 	var documents = collection.Find(x => x.Exp >= 0, options).Project(projection)
												.Sort(sort)
												.Limit(30)
												.Skip(0).ToListAsync();

 	documents.Dump();
}
catch (Exception ex)
{
 	ex.Message.Dump();
}
```

#### Builders 사용
```
try
{
	var collection = MyExtensions.GetMongoDBCollectionVer2<DBBasic>("mongodb://172.20.60.221", "TestDB", "Basic");

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
 	document.Dump();
}
catch (Exception ex)
{
	ex.Message.Dump();
}
```

#### 배열 요소 조건 검색
```
try
{
 	var collection = MyExtensions.GetMongoDBCollectionVer2<DBBasic>("mongodb://172.20.60.221", "TestDB", "Basic");

	var filter = Builders<DBBasic>.Filter.AnyGt(x => x.Costume, 0);
	var documents = await collection.Find(filter).ToListAsync();
 	documents.Dump();
}
catch (Exception ex)
{
 	ex.Message.Dump();
}


// 비동기로 검색 후 데이터를 비동기로 바로 사용
var filter = new BsonDocument("x", new BsonDocument("$gte", 100));
await collection.Find(filter).ForEachAsync(async document =>
{
	await ProcessDocumentAsync(document);
});
```

#### FindAll
```
async Task FooAsync()
{
    var list = await collection.Find(_ => true).ToListAsync();
}
```

#### FindOneAndReplaceAsync
```
try
{
 	var collection = MyExtensions.GetMongoDBCollectionVer2<DBBasic>("mongodb://172.20.60.221", "TestDB", "Basic");

	var newData = new DBBasic()
	{
		_id = "jacking3",
		Money = 3333,
		Costume = new List<int>(Enumerable.Repeat(0, 12))
	};

	// 변경 되기 이전 값을 반환한다. 실패하면 null
	var documents = await collection.FindOneAndReplaceAsync(x => x._id == "jacking5", newData);
 	documents.Dump();
}
catch (Exception ex)
{
 	ex.Message.Dump();
}
```

#### FindOneAndReplaceAsync
```
try
{
 	var collection = MyExtensions.GetMongoDBCollectionVer2<BsonDocument>("mongodb://172.20.60.221", "TestDB", "Basic");

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
	documents.Dump();
}
catch (Exception ex)
{
 	ex.Message.Dump();
}
```

#### FindOneAndUpdateAsync
```
try
{
 	var collection = MyExtensions.GetMongoDBCollectionVer2<BsonDocument>("mongodb://172.20.60.221", "TestDB", "Basic");

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
  document.Dump();
}
catch (Exception ex)
{
 	ex.Message.Dump();
}
```


### Delete
#### 삭제. 한번에 복수개를 지울 때는 DeleteManyAsync
```
try
{
 	var collection = MyExtensions.GetMongoDBCollectionVer2<DBBasic>("mongodb://172.20.60.221", "TestDB", "Basic");

	var userID = "jacking4";
 	var result = await collection.DeleteOneAsync(x=> x._id == userID);
 	result.Dump();
}
catch (Exception ex)
{
 	ex.Message.Dump();
}
```


### Insert
#### 기본
```
try
{
	var newData = new DBBasic()
	{
		_id = "jacking3",
		Level = 1,
		Exp = 0,
		Money = 1000,
		Costume = new List<int>(Enumerable.Repeat(0, 12)),
	};

	var collection = MyExtensions.GetMongoDBCollectionVer2<DBBasic>("mongodb://172.20.60.221", "TestDB", "Basic");
	await collection.InsertOneAsync(newData);
}
catch(Exception ex)
{
	ex.Message.Dump();
}
```

#### 복수의 데이터를 넣기
```
try
{
	var newItemList = new List<DBUserItem>();

	var itemIDList = new List<int>() { 1001, 1002, 1003 };
	int i = 0;
 	foreach (var itemID in itemIDList)
 	{
		++i;

    	var newData = new DBUserItem()
     	{
        	_id = DateTime.Now.Ticks + i,
        	UserID = "jacking",
        	ItemID = itemID,
        	AcquireDateTime = DateTime.Now,
     	};

     	newItemList.Add(newData);
 	}

 	var collection = MyExtensions.GetMongoDBCollectionVer2<DBUserItem>("mongodb://172.20.60.221", "TestDB", "Item");
 	await collection.InsertManyAsync(newItemList);
}
catch (Exception ex)
{
	ex.Message.Dump();
}
```

#### 동적 기능 사용하기
```
try
{
	dynamic person = new System.Dynamic.ExpandoObject();
	person.FirstName = "Jane";
	person.Age = 12;
	person.PetNames = new List<dynamic> { "Sherlock", "Watson" };

	var collection = MyExtensions.GetMongoDBCollectionVer2<dynamic>("mongodb://172.20.60.221", "TestDB", "Persion");
	await collection.InsertOneAsync(person);
}
catch (Exception ex)
{
	ex.Message.Dump();
}
```

#### UTC 시간 보정해서 데이터 넣기
```
try
{
	var newData = new DBTimeData()
	{
		CurTime = DateTime.Now.AddHours(9)
	};

	var collection = MyExtensions.GetMongoDBCollectionVer2<DBTimeData>("mongodb://172.20.60.221", "TestDB", "TimeData");
	await collection.InsertOneAsync(newData);
}
catch(Exception ex)
{
	ex.Message.Dump();
}
```


### Update
#### 기본 업데이트
```
// UpdateOneAsync 는 하나만, 복수는 UpdateManyAsync
try
{
	var collection = MyExtensions.GetMongoDBCollectionVer2<DBUserSkill>("mongodb://172.20.60.221", "TestDB", "Skill");

	var userID = "jacking";

	var result = await collection.UpdateOneAsync(x => x._id == userID,
									Builders<DBUserSkill>.Update.Set(x => x.Value, 14));

	result.Dump();
	// result.MatchedCount 가 1 보다 작으면 업데이트 한 것이 없음
}
catch (Exception ex)
{
 	ex.Message.Dump();
}
```

#### 도큐먼트의 내부 도큐먼트를 변경할 때
```
try
{
	var collection = MyExtensions.GetMongoDBCollectionVer2<DBUserSkill>("mongodb://172.20.60.221", "TestDB", "Skill");

	var userID = "jacking";
	int skillItemID = 1;
	var newInfo = new SkillItemInfo() { Value = 101 };


	var filter = Builders<DBUserSkill>.Filter.And(Builders<DBUserSkill>.Filter.Eq(x => x._id, userID),
    											Builders<DBUserSkill>.Filter.ElemMatch(x => x.SkillItems, x => x.ID == skillItemID));

	var update = Builders<DBUserSkill>.Update.Set("SkillItems.$.Info", newInfo);
	//or
	//var update = Builders<DBUserSkill>.Update.Set(x => x.SkillItems.ElementAt(-1).Info, newInfo);

	collection.UpdateOneAsync(filter, update);
}
catch (Exception ex)
{
 	ex.Message.Dump();
}
```

#### Replace
```
try
{
 	var collection = MyExtensions.GetMongoDBCollectionVer2<DBBasic>("mongodb://172.20.60.221", "TestDB", "Basic");

	var userID = "jacking";
 	var documents = await collection.Find(x=> x._id == userID).SingleAsync();

	if( documents != null)
	{
		documents.Level = documents.Level + 3;
		var result = await collection.ReplaceOneAsync(x => x._id == documents._id, documents);
		result.Dump();
	}
	else
	{
		Console.WriteLine("없음");
	}
}
catch (Exception ex)
{
 	ex.Message.Dump();
}
```
```
var tom = await collection.Find(x => x.Id == ObjectId.Parse("550c4aa98e59471bddf68eef"))
	.SingleAsync();

tom.Name = "Thomas";
tom.Age = 43;
tom.Profession = "Hacker";

var result = await collection.ReplaceOneAsync(x => x.Id == tom.Id, tom);
```




***
아래 내용 수정해야한다.
###접속 시 인증
```
// on the connection string
var connectionString = "mongodb://user:pwd@localhost/?safe=true";
var server = MongoServer.Create(connectionString);

// in code
var cs = "mongodb:localhost";
MongoClient cli = new MongoClient(connectString);

var settings = new MongoServerSettings();
var adminCredential = new MongoCredential("databaseName", "user", "pwd");
settings.Credentials.Add(adminCredential);
```



###옵션 설정하기
- connection pool 크기 설정

```
// on the connection string
ConnectString="mongodb://172.20.60.221:27018/?maxPoolSize=256"

// in code
MongoClientSettings.MaxConnectionPoolSize
MongoClientSettings.MinConnectionPoolSize Property
```



###직렬화

```
class Person
{
	[BsonElement("fn")]
	public string FirstName { get; set; }

	[BsonElement("fn")]
	public string LastName { get; set; }
}

public class GroceryList : MongoEntity<ObjectId>
{
    public FacebookList Owner { get; set; }
    [BsonIgnore]
    public bool IsOwner { get; set; }
}
```


###시간 사용
####MongoDB에서 사용할 수 있는 현재 시간 변환.
MongoDB는 유니버셜 시간대를 사용하고 있으므로 시간을 그냥 사용하면 현재 시간 -9 시간으로 변환되어 버린다.
```
public static DateTime GetLocalTime()
{
	string strCurDateTime = DateTime.Now.ToString("yyyyMMddHHmmss");
	IFormatProvider culture = new System.Globalization.CultureInfo("", true);

	DateTime LocalDateTime =  DateTime.ParseExact(strCurDateTime, "yyyyMMddHHmmss", culture, System.Globalization.DateTimeStyles.AssumeUniversal);

	return LocalDateTime;
}
```

####입력
- 201308011121 라는 문자열로 DateTime 객체를 만든다.
- 중요: MongoDB는 UTC date 타임대를 사용하므로

```
System.Globalization.DateTimeStyles.AssumeUniversal 로 조정한다
IFormatProvider culture = new System.Globalization.CultureInfo("", true);
DateTime date = DateTime.ParseExact(date_time, "yyyyMMddHHmm", culture, System.Globalization.DateTimeStyles.AssumeUniversal);


var collection = BasicLogDB.GetCollection<BsonDocument>("Login");

BsonDocument data = new BsonDocument()
{
   {"ID", id},
   {"UniqNum", UniqueNumber},
   {"date", date}
};

await collection.InsertOneAsync(data);
```


###예제 Insert
####비동기로 하나만 추가
```
using MongoDB.Bson;
using MongoDB.Driver;
using MongoDB.Driver.Builders;

MongoClient cli = new MongoClient(textBoxAddress.Text);
MongoDatabase testdb = cli.GetServer().GetDatabase("test");

MongoCollection<BsonDocument> Customers = testdb.GetCollection<BsonDocument>("Account");

try
{
  for (int i = 0; i < Count; ++i)
  {
      BsonDocument data = new BsonDocument()
      {
          {"ID", string.Format("{0}{1}",textBoxPreFix.Text, (StartIndex+i).ToString())},
          {"PW", "1111"}
      };

      var result = Customers.Insert(data);

      if(result.Ok == false)
         return false;
  }
}
catch (Exception err)
{
   MessageBox.Show(err.ToString());
}
```

####제너릭 데이터를 사용한 추가
```
...

userInfo = new UserInfo
{
    UserIndex = 1,
    ID          = id,
    PW          = pw,
};

var result = Coll.Insert(userInfo);

if (result.Ok == false)
{
    return null;
}
```

####다수의 값을 한번에 추가할 때 InsertBatck
- 복사할 곳(dest)에 복사대상(source)와 같은 index를 가진 것이 있으면 안 된다.

```
MongoCollection<BsonDocument> books;
BsonDocument[] batch = {
    new BsonDocument {
        { "author", "Kurt Vonnegut" },
        { "title", "Cat's Cradle" }
    },
    new BsonDocument {
        { "author", "Kurt Vonnegut" },
        { "title", "Slaughterhouse-Five" }
    }
};
books.InsertBatch(batch);
var squadList = new List<CharData>();
squadList.Add(new CharData
                {
                    UserIndex = userIndex,
                    Slot = 0,
                });
squadList.Add(new CharData
                {
                    UserIndex = userIndex,
                    Slot = 1,
                });

var result = Coll.InsertBatch(squadList);

if (result.Count() != 1)
{
    return false;
}
```


###문자열을 ObjectId로 변환하기
```
var query_id = Query.EQ("_id", ObjectId.Parse("50ed4e7d5baffd13a44d0153"));
// 몽고디비 툴에서 검색할 때
db.Users.find( {_id:ObjectId("546173b07c935c15942aa233")})
```



###예제 Select
####하나만 반환
- 검색된 것이 없을 때는 null 반환. 그러나 하나 이상 있을 때는 예외 발생

```
public static bool Auth(string id, string pw)
{
   try
   {
      MongoClient cli = new MongoClient("mongodb://172.20.60.221");
      MongoDatabase database = cli.GetServer().GetDatabase("test");
      var Accounts = database.GetCollection<BsonDocument>("Account");

      IMongoQuery query = Query.EQ("ID", id);
      var result = Accounts.Find(query).SingleOrDefault();

      if (result == null)
      {
         return false;
      }

      if (result["PW"] != pw)
      {
         return false;
      }
   }
   catch(Exception ex)
   {
      return false;
   }

  return true;
}
```

####여러개 반환
```
IMongoQuery query = Query.EQ("version", version);
var results = ItemInfo.Find(query);


List<ItemInfo> itemlist = new List<ItemInfo>();

foreach (var iteminfo in results)
{
   var item = new ItemInfo
   {
      version = version,
      ID = iteminfo["ID"].AsInt32,
      name = iteminfo["name"].AsString,
      Lv = iteminfo["Lv"].AsInt32
   };

   itemlist.Add(item);
}
//
public class ItemInfo
{
   public int version;
   public int ID;
   public string name;
   public int Lv;
}
```

####Where절 사용
```
Query<Employee>.Where(d => d.EmployeeNumber == 1234)
Query<Employee>.Where(d => d.Name == "John Doe")
Query<Employee>.Where(d => d.EmployeeStatus != Status.Active)
Query<Employee>.Where(d => d.Salary > 100000)
```

####복합 쿼리
```
var query = Query.GTE("x", 1).LTE(3);
// becomes
var query = Query.And(Query.GTE("x", 1), Query.LTE("x", 3));

var query = Query.Not("x").GT(1);
// becomes
var query = Query.Not(Query.GT("x", 1));

var query = Query.Exists("x", true);
// becomes
var query = Query.Exists("x");

var query = Query.Exists("x", false);
// becomes
var query = Query.NotExists("x");
```

####해당 collection 모든 데이터 가져오기
```
List<AccountData> lstAccount = new List<AccountData>();

MongoClient cli = new MongoClient("mongodb://172.20.60.221");
MongoDatabase database = cli.GetServer().GetDatabase("WebTest");
var Accounts = database.GetCollection<BsonDocument>("Account");

// query문 없이 FindAll 사용, 해당 collection에 모든 데이터를 가져올 수 있다.
var result = Accounts.FindAll();

foreach (var AccountInfo in result)
{
    var Account = new AccountData
    {
        AccountID = AccountInfo["ID"].AsString,
        PassWord = AccountInfo["PW"].AsDouble
    };
}
return lstAccount;
```

###First, Last
- First는 제일 첫 번째, Last는 제일 마지막 도큐먼트 반환한다. 그러나 컬렉션에 도큐먼트가 하나도 없는 경우 예외가 발생한다.
- 그래서 안전하게 사용하고 싶다면 FirstOrDefault, LastOrDefult를 사용하는 것이 좋다.
    - Default로 만들기 위해 델리게이트 지정을 안하면 null이 반환된다


###예제 Update, Delete
```
// http://www.csharpstudy.com/Tips/Tips-mongodb.aspx
// Mongo DB를 위한 Connection String
string connString = "mongodb://localhost";

// MongoClient 클라이언트 객체 생성
MongoClient cli = new MongoClient(connString);

// testdb 라는 데이타베이스 가져오기
// 만약 해당 DB 없으면 Collection 생성시 새로 생성함
MongoDatabase testdb = cli.GetServer().GetDatabase("testdb");

// testdb 안에 Customers 라는 컬렉션(일종의 테이블)
// 가져오기. 만약 없으면 새로 생성.
var customers = testdb.GetCollection<Customer>("Customers");

// INSERT - 컬렉션 객체의 Insert() 메서드 호출
// Insert시 _id 라는 자동으로 ObjectID 생성
// 이 _id는 해당 다큐먼트는 나타는 키.
Customer cust1 = new Customer { Name = "Kim", Age = 30 };
customers.Insert(cust1);
ObjectId id = cust1.Id;

// SELECT - id로 검색
IMongoQuery query = Query.EQ("_id", id);
var result = customers.Find(query).SingleOrDefault();
if (result != null)
{
   Console.WriteLine(result.ToString());
}

// UPDATE
//    Save() = 전체 다큐먼트 갱신.
//    Update() = 부분 수정
cust1.Age = 31;
customers.Save(cust1);

// DELETE
var qry = Query.EQ("_id", id);
customers.Remove(qry);
```

데이터가 없으면 추가하고, 있다면 업데이트 하기
```
static void SaveServerHeartBeatInfo_Callback(Object state)
{
	try
	{
		...
		var collection = database.GetCollection<SAVE_DATA>("컬렉션");
		var findData = collection.FindOne(MongoDB.Driver.Builders.Query.EQ("name", id));

		if (findData == null)
		{
			var saveData = new SAVE_DATA
			{
				Name = id,
				value = 1000
			};

			collection.Save(saveData);
		}
		else
		{
			findData.value = 1001;
			collection.Save(findData);
		}
	}
	catch (Exception ex)
	{
	}
}
```


###예제 FindAndModify
```
var modifyArgs = new MongoDB.Driver.FindAndModifyArgs()
{
	Query = Query.EQ("_id", objID),
	Update = Update.Set("MaxBuild", maxBuild),
	Fields = Fields.Include("MaxBuild"), // 이걸 지정하지 않으면 모든 필드의 값을 가져온다.
	SortBy = SortBy.Null,
	VersionReturned = MongoDB.Driver.FindAndModifyDocumentVersion.Modified, // 변경된 값을 가져온다.
	Upsert = false, // 없으면 추가하지 않는다
};

var newResult = collection.FindAndModify(modifyArgs);

// 검색 실패이면 null
if (newResult.ModifiedDocument == null)
{
	return false;
}

return true;
```



###증분으로 업데이트 하기
```
var modifyArgs = new MongoDB.Driver.FindAndModifyArgs()
{
	Query = Query.EQ("_id", objID),
	Update = Update.Inc("Money", change),
	Fields = Fields.Include("Money"),
	SortBy = SortBy.Null,
	VersionReturned = MongoDB.Driver.FindAndModifyDocumentVersion.Modified,
	Upsert = false,
};
```



###복수 데이터 일괄 Update
```
var ub = new UpdateBuilder();
ub.Set("Money", inputData.Money);
ub.Set("Exp", inputData.Exp);
ub.Set("Level", inputData.Level);
collection.Update(Query.EQ("UserID", inputData.UserID), ub);
```



###특정 필드 제외, 특정 필드만 쿼리하기
특정 필드 제외
```
MongoClient cli = new MongoClient(DB_ConnectString);
MongoDatabase db = cli.GetServer().GetDatabase(BasicLogDatabaseName);
var UniqueNumberCol = db.GetCollection<BsonDocument>("UniqueNumber");

// _id 필드만 제외하고 모든 필드 데이터를 가져온다
var newResult = UniqueNumberCol.FindAll().SetFields(Fields.Exclude("_id"));
```

특정 필드만 가져오기
```
MongoClient cli = new MongoClient(DB_ConnectString);
MongoDatabase db = cli.GetServer().GetDatabase(BasicLogDatabaseName);
var UniqueNumberCol = db.GetCollection<BsonDocument>("UniqueNumber");

// money, level 필드 데이터만 가져온다
var newResult = UniqueNumberCol.FindAll().SetFields(Fields.include("money", "level"));
```



###날짜 쿼리
- 특정날짜의 데이터 개수 알아내기
- 시간은 mongoDB에서 자동으로 ISODate로 변경한다. 단 mongoDB에 있는 시간을 애플리케이션에서 사용할 때는 LocalDateTime으로 변경해야 OS의 시간과 일치한다.
    - DB에 저장된 시간 데이터를 로딩할 때 DateTime.MinValue로 저장된 값은 ToLocalTime()으로 변환하지 않는다. 변환하면 MinValue와 다른 값이 된다

	```
	// 어제 날짜
	dateTime = DateTime.Now.AddDays(-1);
	// 0시로 만들기 위해서
	DateTime dateTime2 = new DateTime(dateTime.Year, dateTime.Month, dateTime.Day);
	var count = DBWork.DBDataMining.DailyNewUserStatistics(db, dateTime2);

	static public long DailyNewUserStatistics(MongoDBLib dbLib, DateTime dateTime)
	{
		var collection = dbLib.BasicLogDB.GetCollection<BsonDocument>("NewUser");
		var query = Query.And(Query.GTE("date", dateTime), Query.LTE("date", dateTime.AddDays(1));
		var count = collection.Find(query).Count();
		return count;
	}
	```


###중복 제외
```
//{
//    "_id" : ObjectId("5209d0507dbe2af068a42aec"),
//    "ID" : "jacking75",
//    "Uniq" : 5,
//    "date" : ISODate("2013-08-12T06:21:04Z")
//}

DateTime datetime = MongoDBLib.StringToDateTime(yyyyMMddHHmm);

// 로그인한 총 수를 얻는다
var collection = dbLib.BasicLogDB.GetCollection<BsonDocument>("LoginUser");
var query = Query.And(Query.GTE("date", datetime), Query.LTE("date", datetime.AddDays(1)));

// ID 필드 값 중 중복된 것을 제외하고 개수를 구한다
var UniqueLoginUserCount = collection.Distinct("ID", query).Count();
```



###배열 추가
데이터가 배열인 경우
```
var mongoClient = MongoDBLib.GetGameClient(MongoDBLib.GAME_DATABASE, userIndex);

var database = mongoClient.GetServer().GetDatabase(MongoDBLib.GAME_DATABASE);
var collection = database.GetCollection<BsonDocument>(MongoDBLib.CLEARED_STAGE_LIST_COLLECTION);

var result = collection.Update(Query.EQ("UserIndex", userIndex), Update.PushWrapped("StageIDList", stageID));
isResult = result.UpdatedExisting;
```

List 자료구조를 추가하기
```
int[] defaultValue = {0, 0, 0, 0, 0, 0, 0, 0};
List<int> aaa = new List<int>(defaultValue);


var mongoClient = MongoDBLib.GetGameClient();
MongoDatabase db = mongoClient.GetServer().GetDatabase("GAME_DATABASE");
var Coll = db.GetCollection<BsonDocument>("collection");

BsonDocument data = new BsonDocument()
{
	{"UserIndex", userIndex},
	{"Squard1", new BsonArray(aaa)},
	{"Squard2", new BsonArray(aaa)},
	{"Squard3", new BsonArray(aaa)},
};
int[] 기본값 = { 0, 0, 0, 0, 0, 0, 0, 0 };

var Coll = db.GetCollection<CharInfo>(MongoDBLib.CHARACTER_SQUARD_COLLECTION);

var squadList = new List<CharInfo>();
squadList.Add(new CharInfo
				{
					UserIndex = userIndex,
					ItemIDs = 기본값.ToList(),
				});
squadList.Add(new CharInfo
				{
					UserIndex = userIndex,
					ItemIDs = 기본값.ToList(),
				});
squadList.Add(new CharInfo
				{
					UserIndex = userIndex,
					ItemIDs = 기본값.ToList(),
				});

var result = Coll.InsertBatch(squadList);

if (result.Count() != 1)
{
	return false;
}
```



###배열 전체 업데이트
```
List<bool> usePosList;
Update = Update.Set("UseBPosList", new BsonArray(usePosList))
```



###배열의 특정 요소만 업데이트
```
var aValue = new AA();

// AList의 1번 인덱스(두번째 위치)의 값을 바꾼다.
var query = Query.And(Query.EQ("UserID", userID), Query.EQ("Type", type));
var update = Update.Set("AList." + 1, new BsonDocumentWrapper(aValue));

collection.Update(query, update);
```



###BSonDocument에서 배열 데이터 읽기
```
// 도큐먼트
//{
//    "Cash" : 100,
//    "UseList" : [ false, false, false ],
//}

var dataList = collection.Find(query);

foreach (var data in dataList)
{
	returnValue.Cash = data["Cash"].AsInt32;
	returnValue.UseList = data["UseBPosList"].AsBsonArray.Select(p => p.AsBoolean).ToList();
}
```



###skip & limit
```
Console.WriteLine(collection.Find(query).SetSkip(0).SetLimit(1).Size());
```



###제일 마지막 레코드 가져오기
```
...
MongoCollection<BsonDocument> table = db.GetCollection<BsonDocument>("Collection");
var tableCount = (int)table.Count();

var skipCount = tableCount - 1;
var result = table.FindAll().SetSkip(skipCount);

// result.Size() == 1 이면 제일 마지막 것만 가져온 것 이다.
```



###SP 실행
SP(저장프로시저) 코드를 직접 사용
```
var mongoClient = MongoDBLib.GetAccountClient();
var database = mongoClient.GetServer().GetDatabase(MongoDBLib.ACCOUNT_DATABASE);

BsonValue bv1 = database.Eval("function(x, y) { return x + y; }", x, y);
return (int)bv1.AsDouble;
```

몽고DB에 저장된 SP를 가져와서 실행
```
// 몽고DB 데이터베이스의 Functions에 TestSumSP라는 이름의 SP("function(x, y) { return x + y; }")를 만들어 놓음
var mongoClient = MongoDBLib.GetAccountClient();
var database = mongoClient.GetServer().GetDatabase(MongoDBLib.ACCOUNT_DATABASE);

BsonValue bv = database.Eval("TestSumSP", x, y);
BsonValue bv1 = database.Eval(bv.AsBsonJavaScript.Code, x, y);
return (int)bv1.AsDouble;
```



###SP 호출로 데이터 얻기
```
static string SP_GET_USER = @"function(userID, companyName)
                         {
                             var cursor = db.User.find( {$or:[{Name: companyName}, {ID: userID}]} )
                             var hasData = cursor.hasNext() ? cursor.next() : null;
                             return hasData;
                         }";
public static bool GetUser(string userID, string name)
{
	bool result = false;

	try
	{
		var mongoClient = MongoDBLib.GetAccountClient();
		MongoDatabase db = mongoClient.GetServer().GetDatabase("AccountDB");
		BsonValue bv = db.Eval(SP_GET_USER, userID, name);
		...

                result = true;
	}
	catch (Exception ex)
	{
		return false;
	}

	return result;
}
```



###중복 금지
- 인덱스 사용. 컬렉션에서 중복되면 안되는 필드에 인덱스(유니크)를 걸어서 중복으로 추가되지 않도록 한다.
C#에서 위와 같은 이유로(데이터 중복에 의해) 추가가 실패하면 예외가 발생한다.
- SP 사용

```
static string SP_ADD_USER = @"function(userID, companyName)
            {
                var cursor = db.User.find( {$or:[{Name: companyName}, {ID: userID}]} )
                var hasData = cursor.hasNext() ? cursor.next() : null;
                if(hasData)
                {
                    return false;
                }

                db.User.insert({ ID:userID, Name:companyName })
                return true;
             }";
public static bool AddUser(UserInfo userInfo)
{
    bool result = false;

    try
    {
        var mongoClient = MongoDBLib.GetAccountClient();
        MongoDatabase db = mongoClient.GetServer().GetDatabase("AccountDB");
        BsonValue bv = db.Eval(SP_ADD_USER, userInfo.ID, userInfo.Name);
        result = bv.AsBoolean;
    }
    catch (Exception ex)
    {
        return false;
    }

    return result;
}
```

###정렬
- 데이터 개수가 많고 정렬한 데이터를 각각 업데이트 해야 한다면 MongoDB가 아닌 프로그램의 메모리 상에서 직접 정렬하는 것이 좋다.
    - MongoDB가 느린 이유는 정렬자체 보다는 정렬 후 업데이트를 위해서 문서 하나하나를 업데이트 해야 하는 비용이 비싸기 때문이다.
    - 업데이트라서 InsertBatch를 사용할 수 없어서 업데이트에 많은 비용이 소비된다.
    - 그래서 프로그램에서 DB의 데이터를 가져온 후 직접 정렬한 후 기존 컬렉션의 데이터를 지우고 InserBatch로 새로 데이터를 넣는 것이 훨씬 더 좋다
- 정렬에 제한이 있다. 정렬할 데이터가 메모리 상에 32MB를 넘으면 안 된다. 그래서 정렬에 사용되는 필드는 모두 인덱스에 걸어야 한다.

```
// 정렬 조건 TotalScore가 큰 수, Level이 큰 수, UID가 작은 수
// 복합 인덱스를 건다 db.WeekTotalNetProfit.ensureIndex({ TotalScore: -1, Level: -1, UID: 1 })
public class DBUserWeekTotalNetProfit
{
	public string _id;
	public int Level;
	public Int64 UID;
	public Int64 TotalScore;
	public int TotalTurn;
	public int PrevOrder;   // 전날 랭킹 순위
	public int Order;       // 순위
}

try
{
	var sortBy = SortBy<DBUserWeekTotalNetProfit>.Descending(m => m.TotalScore).Descending(m => m.Level).Ascending(m => m.UID);
	var userDataSources = MongoDBLib.GetUserWeekTotalNetProfitCollection<DBUserWeekTotalNetProfit>();

	int ord = 0;
	foreach (var userDataSource in userDataSources.FindAll().SetSortOrder(sortBy))
	{
		ord++;
		userDataSource.PrevOrder = userDataSource.Order;
		userDataSource.Order = ord;
		userDataSources.Save(userDataSource);
	}
}
catch (Exception ex)
{
}  
```

```
// BSonDocument 사용 시
var dbCollection = Collection<BsonDocument>();
var dataList = dbCollection.FindAll().SetSortOrder(SortBy.Descending("TotalScore"));
```


###컬렉션 복사
```
var sourceCollection = MongoDBLib.GetUserWeekTotalNetProfitCollection<DBUserWeekTotalNetProfit>();
var resultCollection = MongoDBLib.GetUserWeekTotalNetProfitResultCollection<DBUserWeekTotalNetProfit>();

resultCollection.RemoveAll();
resultCollection.InsertBatch(sourceCollection.FindAll());
```


###LINQ
- (일어)mongoDB에서도 LINQ를 사용하자
http://cerberus1974.hateblo.jp/entry/2012/12/13/000245

####기본
```
MongoDB.Driver
MongoDB.Driver.Builders
MongoDB.Driver.Linq
MongoDB.Bson
MongoDB.Bson.Serialization.Attributes
[BsonIgnoreExtraElements]
public class AddressEntry
{
  [BsonId]
  public BsonObjectId _id { get; set;}
  public string Name { get; set; }
  public DateTime Birthday { get; set; }
  public List<Email> EmailAddrs { get; set; }

  [BsonIgnore]
   public string Detail { get; set; }  // 맵핑하지 않는다
}

[BsonIgnoreExtraElements]
public class Email
{
  public string DisplayName { get; set; }
  public string MailAddress { get; set; }
}


// MongoDB에 접속
MongoClient cli = new MongoClient();
// Database를 선택
MongoDatabase db = cli.GetServer().GetDatabase("TestDB");
// 컬렉션 취득
MongoCollection<AddressEntry> col = db.GetCollection<AddressEntry>("AddressBook");
public void Order_Where_Member_1()
{
    var orderCollection = GetCollection(); // 컬렉션 취득을 함수화

    var orders = orderCollection.AsQueryable().Where( x => x.MemberId == 1 ).ToList();

    orders.ForEach( x =>
            {
                WriteOrder( x );
                x.OrderDetails.ToList().ForEach( y => WriteOrderDetail( y ) );
            }
        );
}
```


####&&
```
var query = from e in collection.AsQueryable<Employee>()
      where
         e.EmployeeStatus == Status.Active &&
         e.Salary > 100000
      select e;

// translates to:
{ EmployeeStatus : 1, Salary : { $gt : 100000 } }
```


####문자열에서 일부 문자 찾기
```
var query = from e in collection.AsQueryable<Employee>()
      where e.Name.Contains("oh")
      select e;

// translates to:
{ nm: /oh/s }


var query = from e in collection.AsQueryable<Employee>()
      where e.Name.StartsWith("John")
      select e;

// translates to:
{ nm: /^John/s }
```


####string Length, IsNullOrEmpty
```
var query = from e in collection.AsQueryable<Employee>()
      where e.Name.Length == 4
      select e;

// translates to:
{ nm: /^.{4}$/s }


var query = from e in collection.AsQueryable<Employee>()
      where string.IsNullOrEmpty(e.Name)
      select e;

// translates to:
{ $or : [ { nm: { $type : 10 } }, { nm: "" } ] }
```


####string ToLower
```
var query = from e in collection.AsQueryable<Employee>()
      where e.Name.ToLower() == "john macadam"
      select e;

// translates to:
{ nm: /^john macadam$/is }
```


####array element, Length
```
var query = from e in collection.AsQueryable<Employee>()
      where e.Skills[0] == "Java"
      select e;

// translates to:
{ "Skills.0" : "Java" }


var query = from e in collection.AsQueryable<Employee>()
       where e.Skills.Length == 3
       select e;

// translates to:
{ Skills : { $size : 3 } }
```


####dotted names
```
var query = from e in collection.AsQueryable<Employee>()
      where e.Address.City == "Hoboken"
      select e;

// translates to:
{ "Address.City" : "Hoboken" }
```


####$in
```
var states = new [] { "NJ", "NY", "PA" };
var query = from e in collection.AsQueryable<Employee>()
      where states.Contains(e.Address.State)
      select e;

// translates to:
{ "Address.State" : { $in : [ "NJ", "NY", "PA" ] } }


// alternative syntax using C#/.NET driver "In" method
var query = from e in collection.AsQueryable<Employee>()
     where e.Address.State.In(states)
     select e;
```


####$in/$all with arrays
```
var desiredSkills = new [] { "Java", "C#" };
var query = from e in collection.AsQueryable<Employee>()
      where e.Skills.ContainsAny(desiredSkills)
      select e;

// translates to:
{ "Skills" : { $in : [ "Java", "C#" ] } }


var query = from e in collection.AsQueryable<Employee>()
     where e.Skills.ContainsAll(desiredSkills)
     select e;

// translates to:
{ "Skills" : { $all : [ "Java", "C#" ] } }
// note: ContainsAny and ContainsAll are defined by the C#/.NET
driver and are not part of standard LINQ
```


####$elemMatch
```
var query = from e in collection.AsQueryable<Employee>()
   where
      e.Addresses.Any(a =>
         a.City == "Hoboken" &&
         a.State == "NJ")
   select e;

// translates to:
{ "Addresses" : { $elemMatch :
   { City : "Hoboken", State : "NJ" } } }
```


####Mixing LINQ with MongoDB queries
```
var query = from e in collection.AsQueryable()
      where
         e.Salary > 50000 &&
         Query.NotExists("EmployeeStatus").Inject()
      select e;

// translates to:
{
   Salary : { $gt : 50000 },
   EmployeeStatus : { $exists : false }
}
```



###참고 문서
- What's new in the .NET Driver
http://www.slideshare.net/mongodb/webinar-whats-new-in-the-net-driver
- CSharp Driver LINQ Tutorial
http://docs.mongodb.org/ecosystem/tutorial/use-linq-queries-with-csharp-driver/



###Error Code
http://www.mongodb.org/about/contributors/error-codes/
