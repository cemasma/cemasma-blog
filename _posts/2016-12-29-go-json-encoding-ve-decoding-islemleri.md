---
layout: post
title:  "Go - JSON Encoding ve Decoding İşlemleri"
date:   2016-12-29 00:27:16
categories: go
---


Go dilinde bir JSON'ın encoding ve decoding işlemleri için [json](https://golang.org/pkg/encoding/json/) paketi ve bu işlemler için paket dahilinde Marshal ve Unmarshal fonksiyonları vardır. Marshal fonksiyonu gönderdiğimiz veriyi JSON'a çevirip bunu byte dizisi şeklinde döner. Buradan dönen JSON girintisizdir. Bunun için girintilerini de ayarlayabileceğimiz MarshalIndent fonksiyonu vardır. Unmarshal fonksiyonuna ise byte dizisi şeklindeki JSON'ı ve bu JSON'ın giydirileceği değişkeni adresiyle birlikte göndeririz.


#### Marshal Fonksiyonu
```go
func Marshal(v interface{}) ([]byte, error)
```

#### MarshalIndent Fonksiyonu
```go
func MarshalIndent(v interface{}, prefix, indent string) ([]byte, error)
```

#### Unmarshal Fonksiyonu
```go
func Unmarshal(data []byte, v interface{}) error
```

<br><br>

# Encode

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

#### Encode Çıktısı

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

<br><br>

# Decode

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
// etiketler aracılığı ile `json:"keyismi"` şeklinde belirtiyorum.
type Person struct {
	Name string 		`json:"isim"`
	Surname string		`json:"soyisim"`
	Age int			`json:"yas"`
}

func main() {
	bytedJson := []byte(`[
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
	]`)
	
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
```

#### Decode Çıktısı

```go
{Ulsambre Tortomish 27}
{Vincent Norton 30}
{Uvuveveve Enyetveveve Umubabem Osas 19}
```
