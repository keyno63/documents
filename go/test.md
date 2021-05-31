# Test

テストについて.  
主にユニットテストについて記載.

- testing

基本的に testing を使う.  
testing を import する、テスト関数は (t *testing.T) を引数にする.  
for 式で順番にテストしていく  
test の結果が期待値と違うとき、 `Errorf` させれば失敗する

```go
package main

import "testing"

func Test1(t *testing.T) {
	tests := []struct {
		name string
		value string
		excepted string
    }{
        {
            "title",
            "value",
            "excepted",
        },
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func (t *testing.T) {
            res, err := function1(tt.value)
            if err != nil {
                t.Errorf("exist err, err=[%#v]", err)
            }
            if res != tt.excepted {
                t.Errorf("unmatched actual and excepted. actual=[%#v], excepted=[%#v]", res, tt.excepted)
            }
        })
    }
}
```

- assertion

`stretchr/testify` を取り込み、`assert.Equal` などでチェックできる.  
Java の AssertJ みたいにできそうですね

```go
package main

import "testing"

func Test1(t *testing.T) {
  tests := []struct {
    name string
    value string
    excepted string
  }{
    {
      "title",
      "value",
      "excepted",
    },
  }

  for _, tt := range tests {
    t.Run(tt.name, func (t *testing.T) {
      res, err := function1(tt.value)
      assert.Nil(t, err)
      assert.Equal(t, tt.excepted, res)
    })
  }	  
}
```


