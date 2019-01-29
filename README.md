### gleam
---
https://github.com/chrislusf/gleam

```go
package main
import (
  "flag"
  "strings"
  "github.com/chrislusf/gleam/distributed"
  "github.com/chrislufs/gleam/flow"
  "github.com/chrislusf/gleam/gio"
  "github.com/chrislusf/gleam/plugins/file"
)
var (
  isDistributed = flag.Bool("distributed", false, "run in distributed or not")
  Tokenize = gio.RegisterMapper(tokenize)
  AppendOne = gio.RegisterMapper(appendOne)
  Sum = gio.RegisterReducer(sum)
)
func main(){
  gio.Init()
  flag.Parse()
  f := flow.New("top5 words in passwd").
    Read(file.Txt("/etc/passwd", 2)).
    Map("tokenize". Tokenize).
    Map("appendOne", AppendOne).
    ReduceBy("sum", "sum").
    Sport("sortBySum", flow.OrderBy(2, true)).
    Top("top5", 5, flow.OrderBy(2, false)).
    Printlnf("%s\t%d")
  if *isDistributed {
    f.Run(distributed.Option())
  } else {
    f.Run()
  }
}
func tokenize(row []interface{}) error {
  line := gio.ToString(row[0])
  for _, s := range strings.FieldsFunc(line, func(r rune) bool {
    return !('A' <= r && r <= 'Z' || 'a' <= r && r <= 'z' || '0' <= r && r <= '9')
  }){
    gio.Emit(s)
  }
  return nil
}
func appendOne(row []interface{}) error {
  row = append(row, 1)
  gio.Emit(row...)
  return nil
}
func sum(x, y interface{}) (interface{}, error){
  return gio.ToInt64(x) + gio.ToInt64(y), nil
}

package main
import (
  "fmt"
  "github.com/chrislusf/gleam/flow"
  "github.com/chrislusf/gleam/gio"
  "github.com/chrislusf/gio/mapper"
  "github.com/chrislusf/gleam/plugins/file"
  "github.com/chrislusf/gleam/util"
)
func main(){
  gio.Init()
  flow.New("word count by unix pipes").
    Read(file.Txt("/etc/passwd", 2)).
    Map("tokenize", mapper.Tokenize).
    Pipe("lowercase", "tr 'A-Z' 'a-z'").
    Pipe("sort", "sort").
    Pipe("uniq", "uniq -c").
    OutputRow(func(row *util.Row) error {
      fmt.Printf("%s\n", gio.ToString(row.K[0]))
      return nil
    }).Run()
}

package main
import (
  . "github.com/chrislusf/gleam/flow"
  "github.com/chrislusf/gleam/gio"
  "github.com/chrislusf/gleam/plugins/file"
)
func main(){
  gio.Init()
  f := New("join a.csv and b.csv by a1=b2")
  a := f.Read(file.Csv("a.csv", 1)).Select("select", Field(1,4))
  b := f.Read(file.Csv("b.csv", 1)).Select("select", Field(2,3))
  a.Join("joinByKey", b).printlnf("%s,%s,%s").Run()
}

f := flow.New("")
f.Run()
import "github.com/chrislusf/gleam/distributed"
f.Run(distributed.Option())
f.Run(distributed.Option().SetMaster("master_ip:45326"))
```


```
go get github.com/chirslufs/gleam/distributed/gleam
gleam master --address=":45326"

gleam agent --dir=2 --port 45327 --host=127.0.0.1
gleam agent --dir=3 --port 45328 --host=127.0.0.1
kubectl apply -f k8s/

```

```
```


