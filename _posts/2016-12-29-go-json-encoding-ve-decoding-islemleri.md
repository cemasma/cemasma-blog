---
layout: post
title:  "Go - JSON Encoding ve Decoding İşlemleri"
---


Go dilinde bir JSON'ın encoding ve decoding işlemleri için [json](https://golang.org/pkg/encoding/json/) paketi ve bu işlemler için paket dahilinde Marshal ve Unmarshal fonksiyonları vardır. Marshal fonksiyonu gönderdiğimiz veriyi JSON'a çevirip bunu byte dizisi şeklinde döner. Unmarshal fonksiyonuna ise byte dizisi şeklindeki JSON'ı ve bu JSON'ın giydirileceği değişkeni adresiyle birlikte göndeririz, o da JSON'ı değişkene giydirir. Bu işlemleri yaparken değişkenlerimizin JSON veri yapısına uymasına dikkat etmeliyiz.

#### Kullanacağımız Fonksiyonlar
```go
func Marshal(v interface{}) ([]byte, error)
func MarshalIndent(v interface{}, prefix, indent string) ([]byte, error)
func Unmarshal(data []byte, v interface{}) error
func Indent(dst *bytes.Buffer, src []byte, prefix, indent string) error
```

<br><br>

# Encode
Encode örneğinde veri yapısında JSON etiketleri (etiketler derken bahsettiğim -> `json:"name"`) kullanmadım fakat eğer Decode örneğindeki gibi struct'ta JSON etiketleri kullanılırsa oluşturulan JSON'ın keyleri de belirlenen etiketler olur. Eğer belirlenmezse bu keyler değişken isimleri olur. Bu etiketler go literatüründe "struct tags" olarak geçiyor.

Örnekte önce bir veri yapısı ve bu veri yapısını barındıran bir slice değişkeni oluşturdum. Değişkenimi hazırladıktan sonra Marshal fonksiyonu ile JSON'a dönüştürdüm. Burada Marshal fonksiyonu JSON'ı girintisiz olarak döner. Ancak MarshalIndent ile prefix ve indent parametrelerini kullanarak girintilerini de ayarlayarak JSON'a dönüştürme işlemini yapabiliyoruz. Ayriyeten hali hazırda elimizde girintisiz olarak bulunan bir JSON'ı Indent fonksiyonu kullanarak girintilerini ayarlayabiliriz.

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
	
	byt, _ := json.Marshal(personSlice)
	fmt.Println(string(byt), "\n\n")
	
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
Decode ederken de eğer değişken isimleri ile JSON keyleri tutuşuyor ise ek olarak bu değişkenlerin sağında etiket belirlememize gerek yok. Eğer değişken isimleri farklı olacak ise o zaman hangi keyin hangi değişkenle ilişkileneceğini belirlemek için etiketler kullanılır.

Elimizdeki JSON'a baktığımızda bir array ve içerisinde isim, soyisim ve yaş özellikleri olan nesneler barındırdığını görüyoruz. Bunları dikkate alarak bir değişken oluşturmalıyız ki JSON'ı sıkıntı çıkmadan değişkene enjekte edebilelim. Bunun için JSON'daki array'in içindeki nesnelerin veri yapısını ``` struct ``` ile oluşturdum. Burada JSON'ın keyleri ile veri yapımdaki değişken isimleri tutuşmadığı için de etiketler kullandım. Bknz: ```Name string `json:"isim"` ```

Sonrasında da JSON'da nesneleri barındıran array'e göre ben de değişkenimi, oluşturduğum veri yapısını barındıracak bir slice olarak ayarlıyorum. Bknz: ``` var personSlice []Person ```

Son olarak değişkeni adresiyle birlikte Unmarshal fonksiyonuna gönderiyorum ve Unmarshal fonksiyonu JSON'ı değişkenime enjekte ediyor.

```go
package main

import (
	"fmt"
	"encoding/json"
)

type Person struct {
	Name string     `json:"isim"`
	Surname string  `json:"soyisim"`
	Age int         `json:"yas"`
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

	var personSlice []Person

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
