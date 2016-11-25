### MongoDB의 _id에서 시간을 추출하기
MongoDB의 _id 값은(데이터를 넣을 때 MongoDB가 넣어주는 값의 상위 8 문자는 만들어질 때의 unixtime의 16진수 표현이다.
그래서 _id 값의 상위 8 문자를 추출해서 datetime으로 변환하면 시간을 알 수 있다.

```
string mongo_id = "57b3ce6269d7d0098ef3b603";

string hexString = mongo_id.Substring(0, 8);
int num = Int32.Parse(hexString, System.Globalization.NumberStyles.HexNumber);

Console.WriteLine(num);
Console.WriteLine(DateTimeOffset.FromUnixTimeSeconds(num).ToLocalTime());
```
