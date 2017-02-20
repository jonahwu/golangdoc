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
    cfg.MustValue("server", "address", "localhost")
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
