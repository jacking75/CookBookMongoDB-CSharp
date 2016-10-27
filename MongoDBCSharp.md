
[TOC]



###MongoDB 공식 C# Driver
- CSharp Language Center
http://docs.mongodb.org/ecosystem/drivers/csharp/
- CSharp Driver Tutorial
http://docs.mongodb.org/ecosystem/tutorial/use-csharp-driver/#csharp-driver-tutorial
- 1.8.1 버전 API 문서
http://api.mongodb.org/csharp/1.8.1/



###접속 시 인증
```
// on the connection string
var connectionString = "mongodb://user:pwd@localhost/?safe=true";
var server = MongoServer.Create(connectionString);

// in code
var settings = new MongoServerSettings() {
   DefaultCredentials = new MongoCredentials("user", "pwd"),
   SafeMode = SafeMode.True
};

var server = MongoServer.Create(settings);
```

###서로 인증 정보가 다른 database를 사용할 때
```
var credentialsStore = new MongoCredentialsStore();
credentialsStore.AddCredentials(
      "hr", new MongoCredentials("user1", "pwd1"));
credentialsStore.AddCredentials(
      "inventory", new MongoCredentials("user2", "pwd2"));

var settings = new MongoServerSettings {
      CredentialsStore = credentialsStore,
      SafeMode = SafeMode.True
};

var server = MongoServer.Create(settings);
var credentials = new MongoCredentials("user", "pwd");
var database = server.GetDatabase("hr", credentials);
```


###Admin으로 인증하기
```
var cs = "mongodb://user(admin):pwd@localhost/?safe=true";
var server = MongoServer.Create(cs);

var adminCredentials = new MongoCredentials("user", "pwd", true);
var settings = new MongoServerSettings {
      DefaultCredentials = adminCredentials,
      SafeMode = SafeMode.True
};

var server = MongoServer.Create(settings);
```


###옵션 설정하기
- connection pool 크기 설정
ConnectString="mongodb://172.20.60.221:27018/?maxPoolSize=256"
- 옵션 설정 예제 코드
https://github.com/mongodb/mongo-csharp-driver/blob/master/MongoDB.DriverUnitTests/MongoClientSettingsTests.cs



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


var login = BasicLogDB.GetCollection<BsonDocument>("Login");

BsonDocument data = new BsonDocument()
{
   {"ID", id},
   {"UniqNum", UniqueNumber},
   {"date", date}
};

login.Insert(data);
```


###예제 Insert
####기본
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
특정날짜의 데이터 개수 알아내기
```
// 어제 날짜
dateTime = DateTime.Now.AddDays(-1);
// 0시로 만들기 위해서
DateTime dateTime2 = new DateTime(dateTime.Year, dateTime.Month, dateTime.Day);
var count = DBWork.DBDataMining.DailyNewUserStatistics(db, dateTime2);

static public long DailyNewUserStatistics(MongoDBLib dbLib, DateTime dateTime)
{
	var collection = dbLib.BasicLogDB.GetCollection<BsonDocument>("NewUser");

        //
	var query = Query.And(Query.GTE("date", dateTime.ToUniversalTime()), Query.LTE("date", dateTime.AddDays(1).ToUniversalTime()));
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
