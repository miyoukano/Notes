# foreach遍历Dictionary

一般的遍历Dictionary:

``` python
Dictionary<string,int> myDictionary=new Dictionary<string,int>();
// xxx

foreach(KeyValuePair<string,int> item in myDictionary){
    // xxx
}
```

这种遍历方式仅限于读取Dictionary中的值，无法在遍历的同时修改，因为在`KeyValuePair`底层只有`get`方法，所以无法修改它的值

如果既要用foreach遍历又要能修改其中的值,可以采用以下的办法

## 创建一个Key的List，遍历这个List

``` python
var visitKeys=myDictionary.Keys.ToList();
foreach(var key in visitKeys){
    myDictionary[key]= // xxx
}
```
