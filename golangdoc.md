#Golang Doc

#GO Programing


##規範

```
var aName // private

var BigBro // public

var 123abc // illegal
```
宣告時，第一個字母是大寫表示是Public的，大家都可以存取，如果寫成小寫，就無法存取了。
對於初學者來說，盡量養成習慣寫成大寫，減少錯誤的發生。
有一些比較不好除錯的問題，比如

```
type aa struct {
Name string `json:"name"`
Age string `json:"age"`
}
```
這是一個json的使用方法，如果你誤設成小寫`name and age`，這將導致無法存取json變數，導致無法預期的問題。
FUnction的使用也是如此，所以建議，所有第一位字母皆用大寫。






## Transfer Type

### Floating to String

Here we convert a float to string as dd
```
    dd := fmt.Sprintf("test/%f", 0.01)
    fmt.Println(reflect.TypeOf(dd))
```
Now we can check the type of dd is string by using reflect package.


### String to Float

Here we convert a string to float by using strconv lib

```
a := "3.1415"
c, _ := strconv.ParseFloat(a, 64)
```


### Time.Duration

```
    sec, _ := cfg.Int64("token", "expiretime")
    //sec:=10
    expireTime := client.CreateInOrderOptions{TTL: time.Duration(time.Second * time.Duration(sec))}`
    //expireTime := client.CreateInOrderOptions{TTL: time.Duration(time.Second * 10)}`
```
You can use `time.Second * 10` but you cannot use `time.Second * sec`. 
You must use `time.Second * time.Duration(sec)`. 
This is not as trivial as our think.





## Json, Marshal, and UnMarshal


```
package main

import (
    "encoding/json"
    "fmt"
    "reflect"
)
//Name, Sex declaiEEEust be capital, or it will failed for using json
type aa struct {
    Name string `json:"name"`
    Sex  string `json:"sex"`
}

func main() {
    //define json
    test := aa{}
    test.Name = "jonah"
    test.Sex = "male"
    fmt.Println(reflect.TypeOf(test))
    //convert to character
    chartest, _ := json.Marshal(test)
    fmt.Println(reflect.TypeOf(chartest))
    fmt.Println("print chartest:", chartest)
    //convert to string
    strtest := string(chartest)
    fmt.Println(reflect.TypeOf(strtest))
    fmt.Println("print string:", strtest)
    //convert char to json
    jsontest := aa{}
    _ = json.Unmarshal(chartest, &jsontest)
    fmt.Println(reflect.TypeOf(jsontest))
    fmt.Println(jsontest.Sex)
    //convert string to json
    jsontest1 := aa{}
    // mention that you must trasfer to char by using Unmarshal function
    _ = json.Unmarshal([]byte(strtest), &jsontest1)
    fmt.Println(reflect.TypeOf(jsontest1))
    fmt.Println(jsontest1.Sex)

```
Mention that   
 1. `aa struct` must be capital definition
 2. `Marshal` will transfer to char 
 3. you must be trasfered to string by using string(char)
 4. you can use []char(string) to trasfer to char for UnMarshal input



## Channel

###Channel Buffer

Here is the sample code of channel without and with buffer.
We hope the programe can handle much thoughput than without buffer. 

```
package main

import (
    "fmt"
    //"reflect"
    "time"
)

func run(cb chan int) {
    for {
        for i := 0; i < 15; i++ {
            cb <- i
        }
        time.Sleep(time.Second * 3)
        for i := 15; i < 30; i++ {
            cb <- i
        }

        time.Sleep(time.Second * 10)
    }
}

func send(data int) {
    fmt.Println("send data:", data)
}
func main() {
    cb := make(chan int, 10)
    go run(cb)
    for cbb := range cb { //.where cbb is type of int not chan
        fmt.Println(cbb)
        go send(cbb)
    }
}
```
You can adust `cb := make(chan int, 10)` to set 10 to 1 to be without buffer.

The following is the test of without buffer.
The throughput is small, shoud be 1, but print is 3. I think it's due to `print` delay.

```
0
1
2
send data: 2
send data: 0
3
4
5
send data: 5
send data: 1
send data: 3
6
7
8
send data: 8
send data: 4
send data: 6
9
10
11
send data: 11
send data: 7
send data: 9
12
13
14
send data: 14
send data: 10
send data: 12
send data: 13
```

The following is the result of 10. You can see the throughput is ~10.
The thoughput is much larger.

```
0
1
2
3
4
5
6
7
8
9
10
11
send data: 11
send data: 0
send data: 1
send data: 2
send data: 3
send data: 4
send data: 5
send data: 6
send data: 7
send data: 8
send data: 9
send data: 10
12
13
14
send data: 14
send data: 12
send data: 13
```
The difference is client. When without buffer, client send data, consumer must to read the data.
After that, the client can send data again, since it's blocked after 1 sending.
If buffer is setted to 10. Client send 1 data, the consumer is not read the data. The Client can still 
send 2 data, since the buffer is not filled. It's Nonblocking. If the Consumer not read data, the Client 
will stock on after 10 sending.

You can modify the code to `sending data` in `run` function, and adding sleep before `for` loop,

```
sending data 0
sending data 1
sending data 2
sending data 3
sending data 4
sending data 5
sending data 6
sending data 7
sending data 8
sending data 9
0
sending data 10
1
2
3
4
5
6
7
8
9
10
11
```
You will see the consumer is not active, since it's in sleep, but the client keep sending data.
until the channel filled, and it is then blocked.

However, `go send(cbb)` is must have, or the function will be blocked. 
It does not change the code style, but it really changes the throughput and performance.

## Return Interface

```
func getUserInfo(kAPI client.KeysAPI, username string, usertype string) (string, error) {
    //userinfo := AuthInfo{}
    userinfo := map[string]interface{}{}
    
    if usertype == "UserID" {
        return fmt.Sprintf("%v", userinfo[usertype]), nil
    }
    return "", errors.New("not on the right target of userinfo")

}
```
Do not return `userinfo[usertype]` directly, since our return is string.  
So we have to convert the interface to string by using `fmt.Sprintf` with `%v`, as `fmt.Sprintf("%v", userinfo[usertype])`.

## Access Json

How to access Json.
We might declair a Json struct as followed,

```
type AuthInfo struct {
    UserName string `json:"username"`
    Password string `json:"password"`
    UserID   string `json:"userid"`
    Email    string `json:"email"`
}
```
And how to access it ?

```
Auth.UserName (v)
Auth["UserName"] (X)
```
You can not accees it as an indexing.
It's so bad ......



## How To Call By Name

How to call a struct and its function by a variable.
It's quite important since sometime you wrote a highly abstract code. 
我們常常需要call不同的Method透過外部傳入的參數，透過`reflect`與`MethodByName`如下的方式即可成功。

```
package main

import (
    "fmt"
    "reflect"
)

type aa struct {
    ff int
}

// remember you have use Capital
func (c *aa) Show1() (int, error) {
    fmt.Println("now in show")
    fmt.Println(c.ff)
    a := 2
    return a, nil
}

func main() {
    var t aa
    s := "Show1"
    aa := reflect.ValueOf(&t).MethodByName(s).Call([]reflect.Value{})
    fmt.Println("result", aa[0], aa[1])
   
```

Now you will see the aa[0]=2, and aa[1]=nil.
But the method cannot use in `interface`. 


## Load Config


```
go get github.com/Unknwon/goconfig
```

We use `goconfig` package, since it satisfy my requirement that can get value and setup default value.

```
package main

import (
    "fmt"
    "github.com/Unknwon/goconfig"
)

var cfg *goconfig.ConfigFile

func init() {
    //  cfg, _ := goconfig.LoadConfigFile("./conf.ini")
    cfg, _ = goconfig.LoadConfigFile("./conf.ini")
    _, _ = cfg.MustValueSet("server", "address", "localhost")
}

func main() {

    value, _ := cfg.GetValue(goconfig.DEFAULT_SECTION, "port")
    fmt.Println(value)
    value1, _ := cfg.GetValue("server", "port")
    fmt.Println(value1)
    addr, _ := cfg.GetValue("server", "address")
    fmt.Println(addr)
    test1()

}
```
We declair the cfg as a global variable for multiple go file used.
Any new go file can use the `cfg.GetValue` directly.
Furthermore, we load the config file and setup default value in `init()` function.

conf.ini 

```
port=8000
[server]
port=8081
address= 192.168.33.11
```

In the field without section `[]`, we use `goconfig.DEFAULT_SECTION`.
If you want to read an existed section, just use "server" in first argument.


## Function

Note that, what is the difference between `(a aa)` and `(a *aa)`.

```
type aa struct {
    s float64
}

func (a aa) Reset1() {
    a.s = 0.0
}
func (a *aa) Reset2() {
    a.s = 0.0
}

func main() {
    sa := aa{}
    sa.s = 1
    fmt.Println(sa.s) //1
    sa.Reset1()   
    fmt.Println(sa.s)  //1
    sa.Reset2()  
    fmt.Println(sa.s)  //0

}
```
`Reset1` will not reset, since `a aa` is copy on value definition.  
`Reset2` will reset, since `a *aa` is a pointer.  
It's greate thing? 
But if you really understand and accept it, it will make you less bug and trouble, since you know what 
you were doing.   




## HTTPRouter and Alice

```
import(
    "github.com/gorilla/context"
)


func wrapHandler(h http.Handler) httprouter.Handle {
    return func(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
        context.Set(r, "params", ps)
        h.ServeHTTP(w, r)
    }
}


func GetUserINFOHandler(w http.ResponseWriter, r *http.Request) {
    //token := r.Header.Get("Auth-Token")
    // the following is for general used
    var params httprouter.Params
    if ps := context.Get(r, "params"); ps != nil {
        params = ps.(httprouter.Params) // this is part I hate about gorilla
    }
    fmt.Println("userinfo para", params.ByName("info"))
    // the above is for general used

}


router.GET("/getuserinfo/:info", wrapHandler(commonHandlers.ThenFunc(GetUserINFOHandler)))
```
The url with `info` parameter, and working on `GetUserINFOHandler` to obtain the `info` parameters as `params.ByName("info")`


##Interface

### Send Arbitrary Type and Return Arbatrary Type

```
import (
    "fmt"
    "reflect"
)

func retarb(it interface{}) interface{} {
    fmt.Println(reflect.TypeOf(it))  // related to input type, string or float64
    if fmt.Sprintf("%v", reflect.TypeOf(it)) == "string" {
        return "3"
    }
    if fmt.Sprintf("%v", reflect.TypeOf(it)) == "float64" {
        return 3.14
    }
    return nil
}

func main() {

    arb := retarb("3")
    fmt.Println("result sting", reflect.TypeOf(arb))  //string

    ii := retarb(3.14)
    fmt.Println("result float", reflect.TypeOf(ii))  //float64
}
```
If you are trying to abstract your function. You might send an arbitrary type of data
and return related response.  Try above code.

However You can not use `3+ii`, it will cast the error

```
mismatched types int and interface {}
```
You must write as followed, the ii is not real `float64` type, although `reflect.TypeOf` shows `float64`.

```
3+ii.(float64)
```
That means, it had better to return same type of a interface, such as float64.
said `func retarb(it interface{}) float64 {`

###型別轉換
如果in是interface，並為float64的值。
用如下方式轉換。

```
a:=in.(float64)
```
為什麼要轉換？因為不轉換他是沒辦法做相加的動作，如前一章節說的

```
mismatched types int and interface {}
```



有沒有辦法變成這樣

```
typein:=reflect.TypeOf(in)
a:=in(typea)
```
答案是不行的。
如果要高度抽象你的程式，只能用如下方式

```
    switch t := t.(type) {
    default:
        fmt.Printf("unexpected type %T\n", t) // %T prints whatever type t has
    case bool:
        fmt.Printf("boolean %t\n", t) // t has type bool
    case int:
        fmt.Printf("integer %d\n", t) // t has type int
    case string:
        fmt.Printf("string  %s\n", t) // t has type *bool
    case *int:
        fmt.Printf("pointer to integer %d\n", *t) // t has type *int
    }
```
透過`switch type`的方式完成，你想要的轉換。





#Pattern

## Yeild Patten

We use channel and goroutine to accomplish Yeild patten

```
type gpslocation struct {
    lati float64
    long float64
}

//note channel is blocking
//func callinearloc() (gpslocation, error) {

func callinearloc() chan gpslocation {
    gloc := make(chan gpslocation)
    gg := gpslocation{}
    gg.lati = 23.3333333
    gg.long = 123.333333
    go func() {
        for {
            //fmt.Println("dist loop")
            distx=0.001
            gpsx = gg.lati + distx
            gpsy = gg.lati + disty
            gggloc := gpslocation{gpsx, gpsy}
                        gloc <- gggloc

        }
    }()
    return gloc
}

func gencalllinearloc(ggloc chan gpslocation) gpslocation {
    //here we reuturn value not chan, you can think as type transformation
    return <-ggloc
}

func gpsloc() {
    ggloc := make(chan gpslocation)
    ggloc = callinearloc()
    for {
        fmt.Println("new generated loc", gencalllinearloc(ggloc))
        time.Sleep(time.Second * 1)
    }
}
```

Where `gsloc()`是我們的main function，首先我們先產生一個channel `ggloc`，而`callinearloc()`是主要需要yeild的函數。
在`callinearloc()`中我們先goroutine一段計算函數，並將結果導向channel `gloc`，此`gloc`為此函數自行定義的channel。
我們注意在goroutine後，我們就馬上`return gloc`。並在`gpsloc`函數中用`ggloc channel`接。
這裡很重要的，在這段過程中，channel是block的，換句話說，在`callinearloc()`的`for loop`中他是會被block的。
接收端在哪裡？一個方式是用標準的`<-ggloc`來接，但我們用一個channel to value的轉換器`gencalllinearloc`來接，如此一來
會更符合我們習慣的yeild的方式，記得我們是用`return <-ggloc`，此時return value，而如果是`return ggloc`是return channel。
最後，我們用`gencalllinearloc(ggloc)`去獲得我們要的value。
