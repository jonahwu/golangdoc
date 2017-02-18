#Golang Doc

#GO Programing

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
