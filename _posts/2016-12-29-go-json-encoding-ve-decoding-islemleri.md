---
layout: post
title:  "Go - JSON Encoding ve Decoding İşlemleri"
date:   2016-12-29 00:27:16
categories: go
---

Go dilinde bir JSON'ın encoding ve decoding işlemleri için json paketi kullanılır. Bu paket aşağıdaki gibi import edilir.

```go
import "encoding/json"
```

Bu paketin içeriğine de şuradan ulaşabilirsiniz: https://golang.org/pkg/encoding/json/

Go dilinde JSON encode işlemi için bu paketin Marshal fonksiyonunu kullanırız. Arayüzü aşağıdaki gibidir.

```go
	func Marshal(v interface{}) ([]byte, error)
```

Bu fonksiyonun bir de JSON girintilerini ayarlamak için MarshalIndent versiyonu vardır.

```go
	func MarshalIndent(v interface{}, prefix, indent string) ([]byte, error)
```

## Örnek Encode

```go
package main

import (
	"fmt"
	"encoding/json"
)


type Person struct {
	Name string
	Surname string	
	Age int
}

func main() {
	person1 := Person{Name:"Ulsambre", Surname:"Tortomish", Age:27}
	person2 := Person{Name:"Vincent", Surname:"Norton", Age:30}
	personSlice := []Person{person1, person2}
	
	// Elimizdeki datayı Marshal ile encode ediyoruz
	// Bu şekilde bize girintileri olmayan bir JSON çıktısı verir
	byt, _ := json.Marshal(personSlice)
	fmt.Println(string(byt), "\n\n")
	
	// Ancak MarshalIndent ile prefix ve indent
	// parametrelerini kullanarak girintileri ayarlayabiliriz
	indentedByt, _ := json.MarshalIndent(personSlice, "", " ")
	fmt.Println(string(indentedByt))
}

```

### Çıktı

```javascript
[{"Name":"Ulsambre","Surname":"Tortomish","Age":27},{"Name":"Vincent","Surname":"Norton","Age":30}] 


[
 {
  "Name": "Ulsambre",
  "Surname": "Tortomish",
  "Age": 27
 },
 {
  "Name": "Vincent",
  "Surname": "Norton",
  "Age": 30
 }
```

## Örnek Decode

Decode işlemi için aşağıdaki JSON'ı kullanacağız ve buna göre veri yapımızı belirleyeceğiz.

```javascript
[
    {
      "isim" : "Ulsambre",
      "soyisim" : "Tortomish",
      "yas" : 27
    },
    {
      "isim" : "Vincent",
      "soyisim" : "Norton",
      "yas" : 30
    },
    {
      "isim" : "Uvuveveve Enyetveveve",
      "soyisim" : "Umubabem Osas",
      "yas" : 19
    }
]
```

```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
	"encoding/json"
)

// Burada JSON versine göre bir veri yapısı oluşturdum.
// Ve bu yapıda hangi keydeki verinin hangi niteliğe aktarılacağını da
// etiketler aracılığı ile `json:"isim"` şeklinde belirtiyorum.
type Person struct {
	Name string 		`json:"isim"`
	Surname string		`json:"soyisim"`
	Age int				`json:"yas"`
}

func main() {
	bytedJson := getJSON()
	
	// Aktaracağımı yapacağım değişkeni önceden tanımlamamız gerekiyor.
	// JSON verisi bir array içerisinde dataları içerdiğinden bu şekilde tanımlıyorum.
	var personSlice []Person
	// Oluşturduğum değişkeni adresiyle birlikte gönderiyorum ve
	// değişkenime aktarma işlemi yapılıyor.
	json.Unmarshal(bytedJson, &personSlice)
	
	for _, value := range personSlice {
		fmt.Println(value)
	}
}

// Bu fonksiyon kaynağımdan JSON'ı çekiyor
func getJSON() []byte {
	jsonURL := "https://gist.githubusercontent.com/cemasma/f3e157f048ef54166bdeaf9c1ab5489a/raw/6a142e3540a60ee3e55368a1c613df3f5f4eaade/test0.json"
	resp, _ := http.Get(jsonURL)
	
	defer resp.Body.Close()
	body, _ := ioutil.ReadAll(resp.Body)
	return body
}
```

### Çıktı

```go
{Ulsambre Tortomish 27}
{Vincent Norton 30}
{Uvuveveve Enyetveveve Umubabem Osas 19}
```
