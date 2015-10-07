# CookBookMongoDB-CSharp
MongoDB를 C#으로 프로그래밍 하는 방법을 설명한다


## MongoDB 공식 C# Driver
- 1.x와 2.x 라이브러리가 많이 다름.
- [2.0 드라이버 소개](https://www.mongodb.com/blog/post/introducing-20-net-driver)
- [레퍼런스](http://mongodb.github.io/mongo-csharp-driver/2.0/reference/)
- [온라인 도움 문서](http://api.mongodb.org/csharp/2.0/html/R_Project_CSharpDriverDocs.htm)
- [Indexes.CreateOneAsync sample](http://qiita.com/SHUAI/items/f9479f2c1a09bbd7e36b)
- [Find & FindAsync sample](http://qiita.com/SHUAI/items/ace6f6d3f08bb8dd79c5)



## 목차
- MongoDB를 Windows에 설치
- C# 드라이버 설치


## MongoDB C# 드라이버 설치
- https://www.nuget.org/packages/MongoDB.Driver
- NuGet으로 설치한다.



## MongoDB collection 객체 생성

```
string DBName; // 데이터베이스 이름

public static IMongoCollection<T> GetDBCollection<T>(string collectionName)
{
    var mongoClient = GetDBClient(DBConnectString);
    if (mongoClient == null)
    {
        return null;
    }

    var collection = mongoClient.GetDatabase(DBName).GetCollection<T>(collectionName);
    return collection;
}


static IMongoClient GetDBClient(string connectString)
{
    try
    {
        // connectString
        // mongodb://testID:123asd@192.168.0.1:27017/?maxPoolSize=200&safe=true
        // mongodb://192.168.0.1:27017
        MongoClient cli = new MongoClient(connectString);
        return cli;
    }
    catch (Exception ex)
    {
        return null;
    }
}
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
