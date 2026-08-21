grep — Kapsamlı Parametre ve Kullanım Rehberi

> [!summary] Amaç
> Bu not, `grep` komutunu yalnızca ezberlemek yerine parametrelerin ne yaptığını, metin arama mantığının nasıl işlediğini, günlük kullanımda ve log/kod analizi gibi çalışmalarda nerede işe yaradığını öğrenmek için hazırlanmıştır.

## 1. grep Nedir?

`grep`, metin içinde belirli bir kalıba (pattern) uyan satırları bulmak için kullanılan komut satırı aracıdır.

İsmi **g**lobal **r**egular **e**xpression **p**rint ifadesinden gelir.

Temel yapı:

```
grep [parametreler] KALIP [dosya...]
```

Örneğin:

```
grep "hata" log.txt
```

Burada:

```
grep
│
├── "hata"     → aranan kalıp
└── log.txt    → içinde aranacak dosya
```

`grep`, `log.txt` dosyasını satır satır okur ve içinde "hata" geçen satırları terminale yazdırır.

## 2. grep'in Mantığını Anlamak

`grep` aslında bir filtredir. Girdi olarak bir metin akışı alır, her satırı kalıpla karşılaştırır ve eşleşen satırları çıktı olarak verir.

```
Girdi (dosya / stdin)
        |
        v
   ┌─────────┐
   │  grep   │  ← her satırı kalıpla karşılaştırır
   └─────────┘
        |
        v
  Eşleşen satırlar
```

Örneğin bir dosya:

```
Sistem başlatıldı
Bağlantı hatası oluştu
İşlem tamamlandı
Disk hatası: yetersiz alan
```

Şunu çalıştırırsan:

```
grep "hata" dosya.txt
```

Çıktı:

```
Bağlantı hatası oluştu
Disk hatası: yetersiz alan
```

Bu nedenle `grep`, log dosyalarını, kod tabanlarını ve yapılandırma dosyalarını hızlıca taramak için çok değerlidir.

## 3. En Temel Parametreler

### -i, --ignore-case

**Kısa özet**

Büyük/küçük harf duyarlılığını kapatır.

**Ne yapar?**

Normalde:

```
grep "hata" log.txt
```

yalnızca küçük harfli "hata" ifadesini bulur.

`-i` eklediğinde "Hata", "HATA", "hAta" gibi tüm varyasyonlar da eşleşir.

```
grep -i "hata" log.txt
```

**Neden önemlidir?**

Loglar ve kaynak kodlar tutarlı bir büyük/küçük harf kullanımına sahip olmayabilir.

**Günlük hayatta kullanım**

Bir log dosyasında:

- "Error", "error", "ERROR" gibi farklı yazımları
- kullanıcı isimlerini büyük/küçük harf farkı gözetmeden

aramak için kullanılır.

**Örnek**

```
grep -i "error" application.log
```

### -v, --invert-match

**Kısa özet**

Kalıpla **eşleşmeyen** satırları gösterir.

**Örnek**

```
grep -v "DEBUG" log.txt
```

**Ne olur?**

"DEBUG" içeren satırlar hariç, dosyadaki diğer tüm satırlar gösterilir.

**Günlük kullanım**

Gürültülü log satırlarını (örneğin debug/bilgi mesajlarını) elemek için kullanılır.

```
grep -v "DEBUG" app.log | grep -v "INFO"
```

Bu şekilde yalnızca WARNING, ERROR gibi önemli satırlar kalır.

### -c, --count

**Kısa özet**

Eşleşen satır sayısını verir, satırların kendisini göstermez.

**Örnek**

```
grep -c "hata" log.txt
```

Çıktı örneğin:

```
7
```

**Dikkat**

`-c` satır sayısını verir, **eşleşme sayısını değil**. Bir satırda kalıp birden fazla geçse bile o satır 1 kez sayılır.

### -n, --line-number

**Kısa özet**

Eşleşen satırın numarasını da gösterir.

**Örnek**

```
grep -n "hata" log.txt
```

Çıktı:

```
3:Bağlantı hatası oluştu
8:Disk hatası: yetersiz alan
```

**Neden önemli?**

Bir kod dosyasında veya log dosyasında ilgili satıra doğrudan gitmek istediğinde çok işe yarar.

```
grep -n "TODO" main.py
```

### -l, --files-with-matches

**Kısa özet**

Kalıbın **hangi dosyalarda** geçtiğini listeler, satırları göstermez.

**Örnek**

```
grep -l "TODO" *.py
```

Çıktı:

```
utils.py
main.py
```

**Kullanım alanı**

Birçok dosya arasında hangilerinin ilgili kalıbı içerdiğini hızlıca bulmak için kullanılır.

### -L, --files-without-match

**Kısa özet**

`-l`'nin tersidir: kalıbın **geçmediği** dosyaları listeler.

**Örnek**

```
grep -L "# TODO" *.py
```

Bu, henüz TODO içermeyen dosyaları gösterir.

## 4. Eşleşme Genişliği ile İlgili Parametreler

### -w, --word-regexp

**Kısa özet**

Kalıbı yalnızca **tam kelime** olarak eşleştirir.

**Örnek**

```
grep -w "log" not.txt
```

**Fark**

`-w` olmadan:

```
grep "log" not.txt
```

"log", "login", "logout", "catalog" gibi kelimelerin içinde geçen her yeri bulur.

`-w` ile:

```
grep -w "log" not.txt
```

yalnızca bağımsız "log" kelimesini bulur; "login" veya "catalog" eşleşmez.

### -x, --line-regexp

**Kısa özet**

Kalıbın **satırın tamamıyla** birebir eşleşmesini ister.

**Örnek**

```
grep -x "OK" durum.txt
```

Yalnızca içeriği tam olarak "OK" olan satırlar eşleşir; "Durum: OK" gibi satırlar eşleşmez.

### -o, --only-matching

**Kısa özet**

Satırın tamamı yerine yalnızca **eşleşen kısmı** yazdırır.

**Örnek**

```
grep -o "[0-9]\+" veri.txt
```

Bir satırda:

```
Kullanıcı ID: 4521, Yaş: 30
```

`-o` ile çıktı:

```
4521
30
```

**Neden önemli?**

Loglardan yalnızca IP adresi, e-posta, hata kodu gibi belirli parçaları çekmek için çok kullanışlıdır.

```
grep -oE "[0-9]{1,3}(\.[0-9]{1,3}){3}" erisim.log
```

Bu, log dosyasındaki IP adreslerini çeker.

## 5. Birden Fazla Kalıp ve Dosyadan Kalıp

### -e, --regexp

**Kısa özet**

Birden fazla kalıp belirtmeyi sağlar.

**Örnek**

```
grep -e "hata" -e "uyarı" log.txt
```

Ya "hata" ya da "uyarı" geçen satırları gösterir.

**Alternatif**

```
grep -E "hata|uyarı" log.txt
```

ile de aynı sonuç elde edilebilir (bkz. `-E`).

### -f, --file

**Kısa özet**

Kalıpları komut satırından değil, bir dosyadan okur.

**Örnek**

Bir `kaliplar.txt` dosyası:

```
hata
uyarı
kritik
```

Kullanım:

```
grep -f kaliplar.txt log.txt
```

**Kullanım alanı**

Çok sayıda anahtar kelimeyi tararken (örneğin yasaklı kelime listesi, hata kodu listesi) çok kullanışlıdır.

## 6. Regex Modları

### -E, --extended-regexp

**Kısa özet**

Genişletilmiş düzenli ifadeleri (ERE) etkinleştirir.

**Ne değişir?**

Normal `grep`'te (`-G`, temel regex) bazı karakterlerin özel anlamı için ters slash gerekir:

```
grep "hata\|uyarı" log.txt
```

`-E` ile bu karakterler doğrudan özel anlam taşır:

```
grep -E "hata|uyarı" log.txt
```

**Sık kullanılan ERE karakterleri**

```
|   → veya
+   → bir veya daha fazla
?   → sıfır veya bir
()  → gruplama
{}  → tekrar sayısı
```

**Örnek**

```
grep -E "^[0-9]{3}-[0-9]{4}$" telefon.txt
```

Bu, "123-4567" formatındaki satırları bulur.

### -F, --fixed-strings

**Kısa özet**

Kalıbı regex olarak değil, **düz metin** olarak arar.

**Örnek**

```
grep -F "192.168.1.1" log.txt
```

**Neden önemli?**

Regex modunda `.` herhangi bir karakter anlamına gelir. IP adresi gibi noktalı bir metni ararken `-F` kullanmak yanlış eşleşmeleri önler.

```
grep "192.168.1.1" log.txt      → "192a168a1a1" gibi satırları da yanlışlıkla eşleştirebilir
grep -F "192.168.1.1" log.txt   → yalnızca birebir bu metni arar
```

### -P, --perl-regexp

**Kısa özet**

Perl uyumlu düzenli ifadeleri (PCRE) etkinleştirir.

**Örnek**

```
grep -P "\d{3}-\d{4}" telefon.txt
```

**Neden kullanışlı?**

`\d`, `\w`, `\s` gibi kısaltmalar ve lookahead/lookbehind gibi gelişmiş özellikler sunar.

```
grep -P "(?<=Kullanıcı: )\w+" kullanicilar.txt
```

Bu, "Kullanıcı: " ifadesinden sonra gelen kelimeyi yakalar.

**Dikkat**

`-P` her sistemde (özellikle bazı minimal Unix ortamlarında) mevcut olmayabilir.

## 7. Bağlam Gösterme Parametreleri

### -A, --after-context

**Kısa özet**

Eşleşen satırdan sonraki N satırı da gösterir.

**Örnek**

```
grep -A 2 "hata" log.txt
```

Eşleşen satır + sonraki 2 satır gösterilir.

```
Bağlantı hatası oluştu
  Yeniden deneniyor...
  Bağlantı kapatıldı
```

### -B, --before-context

**Kısa özet**

Eşleşen satırdan önceki N satırı gösterir.

**Örnek**

```
grep -B 2 "hata" log.txt
```

### -C, --context

**Kısa özet**

Eşleşen satırın hem öncesini hem sonrasını gösterir.

**Örnek**

```
grep -C 2 "hata" log.txt
```

**Görsel özet**

```
        satır (n-2)
        satır (n-1)
  -->   satır (n)   ← eşleşen satır
        satır (n+1)
        satır (n+2)
```

**Neden önemli?**

Bir hatanın tek başına bir satırdan anlaşılması genelde zordur. Öncesi ve sonrası bağlamı verir.

## 8. Dosya ve Dizin ile İlgili Parametreler

### -r, --recursive

**Kısa özet**

Bir dizin ve alt dizinlerindeki tüm dosyalarda arama yapar.

**Örnek**

```
grep -r "TODO" ./proje
```

Sembolik bağlantıları takip etmez.

### -R

**Kısa özet**

`-r` ile aynıdır, ancak sembolik bağlantıları da takip eder.

```
grep -R "TODO" ./proje
```

### --include

**Kısa özet**

Yalnızca belirli uzantıdaki dosyaları dahil eder.

**Örnek**

```
grep -r --include="*.py" "TODO" ./proje
```

Yalnızca `.py` dosyalarında arama yapar.

### --exclude

**Kısa özet**

Belirli dosyaları aramadan hariç tutar.

**Örnek**

```
grep -r --exclude="*.log" "hata" ./proje
```

### --exclude-dir

**Kısa özet**

Belirli dizinleri aramadan hariç tutar.

**Örnek**

```
grep -r --exclude-dir="node_modules" "TODO" ./proje
```

**Neden önemli?**

Büyük projelerde `node_modules`, `.git`, `dist` gibi klasörleri taramak hem gereksizdir hem de yavaşlatır.

## 9. Çıktı Kontrolü

### -q, --quiet / --silent

**Kısa özet**

Hiçbir çıktı vermez; yalnızca çıkış kodu (exit code) döner.

**Örnek**

```
grep -q "hata" log.txt
```

**Kullanım alanı**

Script'lerde koşul kontrolü için kullanılır:

```
if grep -q "hata" log.txt; then
    echo "Hata bulundu"
else
    echo "Hata yok"
fi
```

### -s, --no-messages

**Kısa özet**

Dosya bulunamadı veya okunamadı gibi hata mesajlarını gizler.

**Örnek**

```
grep -s "hata" olmayan_dosya.txt
```

**Kullanım alanı**

Script'lerde beklenmedik dosya hatalarının çıktıyı kirletmesini önler.

### --color

**Kısa özet**

Eşleşen kısmı terminalde renkli gösterir.

**Örnek**

```
grep --color "hata" log.txt
```

Çoğu dağıtımda `grep`, `alias grep='grep --color=auto'` şeklinde zaten önceden tanımlıdır.

### -m, --max-count

**Kısa özet**

Belirtilen sayıda eşleşme bulunduktan sonra durur.

**Örnek**

```
grep -m 3 "hata" log.txt
```

İlk 3 eşleşmeyi bulduktan sonra aramayı durdurur.

**Kullanım alanı**

Çok büyük dosyalarda yalnızca birkaç örnek görmek yeterliyse zaman kazandırır.

### -z, --null-data

**Kısa özet**

Satırları yeni satır (`\n`) yerine null karakteriyle (`\0`) ayırır.

**Kullanım alanı**

`find -print0` gibi null ile ayrılmış çıktılarla birlikte kullanılır:

```
find . -print0 | xargs -0 grep -l "TODO"
```

## 10. Standart Çıktı, Standart Hata ve `2> /dev/null`

Bu bölüm bir `grep` parametresi değil, bir **shell (kabuk) tekniğidir**. Ancak `grep` ile o kadar sık birlikte kullanılır ki ayrı bir başlığı hak eder.

**Kısa özet**

Bir komutun ürettiği çıktı iki ayrı kanaldan akar:

```
STDOUT (1)  → normal / eşleşen çıktı
STDERR (2)  → hata mesajları
```

`2> /dev/null` ifadesi, bir komutun STDERR (hata) kanalını `/dev/null`'a yönlendirerek yok eder; STDOUT hâlâ terminalde görünmeye devam eder.

**Ne zaman işe yarar?**

`grep`, erişemediği bir dosya veya dizinle karşılaştığında STDERR'e bir hata mesajı yazar:

```
grep "hata" olmayan_dosya.txt
```

Çıktı:

```
grep: olmayan_dosya.txt: No such file or directory
```

Bu mesajı gizlemek için:

```
grep "hata" olmayan_dosya.txt 2> /dev/null
```

Artık dosya yoksa hiçbir şey yazdırılmaz; terminal temiz kalır.

**Görsel özet**

```
grep KALIP DOSYA
        |
        ├── eşleşen satırlar ────────► STDOUT (1) ──► terminalde görünür
        |
        └── "No such file..." vb. ───► STDERR (2) ──► 2>/dev/null ile yok olur
```

**`-s` ile `2> /dev/null` arasındaki fark**

Bu ikisi benzer görünür ama aynı şey değildir:

```
-s          → grep'in KENDİ ürettiği mesajları bastırır (grep'e özgü bir seçenektir)
2>/dev/null → komutun STDERR kanalının TAMAMINI yok eder (grep'e özgü değildir, her komutta çalışır)
```

Örneğin:

```
grep -s "hata" olmayan_dosya.txt
```

ile

```
grep "hata" olmayan_dosya.txt 2> /dev/null
```

çoğu durumda aynı görsel sonucu verir, ancak `2> /dev/null` daha geneldir ve `grep`'in dışında başka herhangi bir komutla da (curl, cat, ls, vb.) kullanılabilir.

**`-r` ile birlikte kullanım (çok yaygın bir kalıp)**

Bir dizinde özyinelemeli arama yaparken izin hatalarıyla sık karşılaşılır:

```
grep -r "kalıp" / 2> /dev/null
```

Bu komut olmadan:

```
grep: /proc/1/task: Permission denied
grep: /root: Permission denied
```

gibi onlarca satır hata mesajı çıktıyı kirletir. `2> /dev/null` eklendiğinde yalnızca gerçek eşleşmeler görünür.

**Script içinde kullanım**

```
if grep -q "hata" log.txt 2> /dev/null; then
    echo "Hata bulundu"
else
    echo "Hata yok veya dosya erişilemez"
fi
```

**Dikkat**

`2> /dev/null` hataları **görünmez** kılar, **ortadan kaldırmaz**. Bir script hatalı çalışıyorsa ve nedenini araştırıyorsan, hata ayıklama (debug) sırasında bu yönlendirmeyi geçici olarak kaldırmak faydalıdır.

**İlgili yönlendirme kalıpları**

```
komut > dosya          → yalnızca STDOUT'u dosyaya yaz
komut 2> dosya         → yalnızca STDERR'i dosyaya yaz
komut > dosya 2>&1     → hem STDOUT hem STDERR'i aynı dosyaya yaz
komut 2>/dev/null      → yalnızca STDERR'i yok say
komut &> /dev/null     → hem STDOUT hem STDERR'i yok say (bash'e özgü kısayol)
```

## 11. Temel Regex Kalıpları

`grep` öğrenirken düzenli ifadelerin (regex) temel mantığını bilmek şarttır.

```
^     → satırın başı
$     → satırın sonu
.     → herhangi bir tek karakter
*     → önceki karakterden sıfır veya daha fazla
+     → önceki karakterden bir veya daha fazla (ERE/-P)
?     → önceki karakter opsiyonel (ERE/-P)
[]    → karakter sınıfı, örn. [0-9], [a-z]
[^]   → sınıfın tersi, örn. [^0-9]
()    → gruplama (ERE/-P)
|     → veya (ERE/-P)
\d    → rakam (yalnızca -P)
\w    → kelime karakteri (yalnızca -P)
\s    → boşluk karakteri (yalnızca -P)
```

**Örnek: satır başı**

```
grep "^Hata" log.txt
```

Yalnızca "Hata" ile **başlayan** satırları bulur.

**Örnek: satır sonu**

```
grep "tamamlandı$" log.txt
```

Yalnızca "tamamlandı" ile **biten** satırları bulur.

**Örnek: karakter sınıfı**

```
grep "[Hh]ata" log.txt
```

Hem "Hata" hem "hata" ifadesini bulur (bu aynı zamanda `-i` ile de yapılabilir).

**Örnek: rakam arama**

```
grep -E "[0-9]+" veri.txt
```

Bir veya daha fazla ardışık rakam içeren satırları bulur.

## 12. Gerçek Kullanım Örnekleri

### Log dosyasında hata arama

```
grep -in "error" uygulama.log
```

`-i` → büyük/küçük harf duyarsız
`-n` → satır numarası göster

### Belirli bir IP adresinin geçtiği satırları bulma

```
grep -F "192.168.1.10" erisim.log
```

### Kod içinde TODO/FIXME arama

```
grep -rn --include="*.js" -e "TODO" -e "FIXME" ./src
```

### Bir portu dinleyen servisleri bulma (örneğin netstat çıktısında)

```
netstat -tulpn | grep ":80"
```

### Bir kullanıcının log dosyasındaki tüm işlemlerini görme

```
grep "kullanici_ali" islemler.log
```

### Sadece dosya isimlerini listeleme

```
grep -rl "gizli_anahtar" ./proje
```

### E-posta adreslerini bir dosyadan çekme

```
grep -oE "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" kisiler.txt
```

### Boş olmayan satırları listeleme

```
grep -v "^$" dosya.txt
```

### Yorum satırları hariç kod satırlarını görme

```
grep -v "^#" ayar.conf
```

## 13. grep'i Diğer Komutlarla Birleştirme (Pipe)

`grep` genellikle tek başına değil, diğer komutlarla **boru (pipe)** üzerinden birlikte kullanılır.

```
komut1 | grep "kalıp"
```

**Örnek: çalışan süreçleri filtreleme**

```
ps aux | grep "python"
```

**Örnek: geçmiş komutlarda arama**

```
history | grep "docker"
```

**Örnek: birden fazla grep zinciri**

```
cat log.txt | grep "2026-08" | grep -v "DEBUG" | grep -c "hata"
```

Mantık:

```
cat log.txt          → dosyayı oku
  |
grep "2026-08"       → yalnızca bu tarihe ait satırlar
  |
grep -v "DEBUG"      → DEBUG satırlarını çıkar
  |
grep -c "hata"       → kalan satırlardan "hata" içerenleri say
```

## 14. -i, -v, -n Birlikte Kullanım Örneği

```
grep -inv "debug" log.txt
```

Açıklaması:

```
-i  → büyük/küçük harf duyarsız
-n  → satır numarası göster
-v  → "debug" GEÇMEYEN satırları göster
```

Yani: "debug" kelimesi (büyük/küçük harf fark etmeksizin) geçmeyen tüm satırları, satır numarasıyla birlikte gösterir.

## 15. En Önemli 15 Parametre

Öncelik sırasıyla:

1. `-i` — büyük/küçük harf duyarsız arama
2. `-v` — eşleşmeyenleri göster
3. `-n` — satır numarası göster
4. `-c` — eşleşme sayısını göster
5. `-r` / `-R` — dizinlerde özyinelemeli arama
6. `-l` — eşleşen dosyaları listele
7. `-w` — tam kelime eşleştirme
8. `-o` — yalnızca eşleşen kısmı göster
9. `-E` — genişletilmiş regex
10. `-F` — düz metin arama
11. `-A` / `-B` / `-C` — bağlam satırları göster
12. `-e` — birden fazla kalıp
13. `--include` / `--exclude` — dosya filtreleme
14. `-q` — script'ler için sessiz kontrol
15. `--color` — eşleşmeyi renklendir

## 16. Öğrenme Sırası

`grep`'i öğrenirken tüm parametreleri aynı anda ezberlemeye çalışma.

Önce:

```
grep "kalıp" dosya.txt
```

Sonra:

```
-i
-v
-n
-c
```

Sonra:

```
-r
-l
-L
2> /dev/null
```

Sonra:

```
-w
-x
-o
```

Sonra:

```
-E
-F
-P
```

Sonra:

```
-A
-B
-C
```

Son olarak:

```
-e
-f
-m
-q
--include
--exclude
```

öğrenmek daha mantıklıdır.

## 17. Günlük Kullanım İçin Mini Cheat Sheet

```
# Basit arama
grep "kalıp" dosya.txt

# Büyük/küçük harf duyarsız
grep -i "kalıp" dosya.txt

# Eşleşmeyenleri göster
grep -v "kalıp" dosya.txt

# Satır numarasıyla
grep -n "kalıp" dosya.txt

# Eşleşme sayısı
grep -c "kalıp" dosya.txt

# Dizinde özyinelemeli arama
grep -r "kalıp" ./dizin

# Sadece dosya isimlerini göster
grep -rl "kalıp" ./dizin

# Tam kelime
grep -w "kalıp" dosya.txt

# Yalnızca eşleşen kısmı göster
grep -o "kalıp" dosya.txt

# Genişletilmiş regex ile birden fazla kalıp
grep -E "kalip1|kalip2" dosya.txt

# Düz metin (regex değil)
grep -F "192.168.1.1" dosya.txt

# Bağlamla birlikte göster
grep -C 2 "kalıp" dosya.txt

# Belirli uzantıyla sınırlı arama
grep -r --include="*.py" "kalıp" ./proje

# Belirli klasörü hariç tut
grep -r --exclude-dir="node_modules" "kalıp" ./proje

# Script içinde sessiz kontrol
grep -q "kalıp" dosya.txt

# İlk N eşleşmeyle sınırla
grep -m 5 "kalıp" dosya.txt

# Hata mesajlarını gizleyerek ara (dizin/izin hataları dahil)
grep -r "kalıp" ./dizin 2> /dev/null

# Script içinde sessiz + hata mesajı gizli kontrol
if grep -q "kalıp" dosya.txt 2> /dev/null; then
    echo "Bulundu"
fi
```

## 18. grep'in Sistemindeki Kılavuzunu Görmek

`grep` sürümleri arasında küçük farklar olabilir (özellikle GNU grep ile BSD grep).

Sürümü görmek için:

```
grep --version
```

Detaylı manuel:

```
man grep
```

Belirli bir konuyu aramak:

```
man grep | grep -A 3 "context"
```

## 19. grep Öğrenirken En Önemli Mantık

`grep` parametrelerini ezberlemek yerine bir aramayı parçalara ayır.

```
ARAMA
│
├── Kalıp (Pattern)
│   └── düz metin / regex
│
├── Kaynak
│   └── dosya / dizin / stdin (pipe)
│
├── Eşleşme Kuralı
│   ├── büyük/küçük harf duyarlı mı? (-i)
│   ├── tam kelime mi? (-w)
│   └── tam satır mı? (-x)
│
├── Çıktı Şekli
│   ├── satırın tamamı mı, yalnızca eşleşen kısım mı? (-o)
│   ├── satır numarası var mı? (-n)
│   ├── sadece sayı mı? (-c)
│   └── sadece dosya adı mı? (-l / -L)
│
└── Kapsam
    ├── tek dosya
    ├── birden fazla dosya
    └── dizin (özyinelemeli, -r)
```

Bu mantığı kavradığında `grep`'in onlarca parametresini ezberlemek zorunda kalmazsın.

## 20. Hızlı Referans Tablosu

| Parametre | Görevi |
|---|---|
| `-i` | Büyük/küçük harf duyarsız |
| `-v` | Eşleşmeyenleri göster |
| `-c` | Eşleşme sayısını say |
| `-n` | Satır numarası göster |
| `-l` | Eşleşen dosyaları listele |
| `-L` | Eşleşmeyen dosyaları listele |
| `-w` | Tam kelime eşleştirme |
| `-x` | Tam satır eşleştirme |
| `-o` | Yalnızca eşleşen kısmı göster |
| `-e` | Birden fazla kalıp belirt |
| `-f` | Kalıpları dosyadan oku |
| `-E` | Genişletilmiş regex (ERE) |
| `-F` | Düz metin (regex değil) |
| `-P` | Perl uyumlu regex (PCRE) |
| `-A` | Sonraki N satırı göster |
| `-B` | Önceki N satırı göster |
| `-C` | Öncesi + sonrasını göster |
| `-r` / `-R` | Dizinde özyinelemeli arama |
| `--include` | Belirli uzantıyı dahil et |
| `--exclude` | Belirli dosyayı hariç tut |
| `--exclude-dir` | Belirli dizini hariç tut |
| `-q` | Sessiz mod, yalnızca exit code |
| `-s` | Hata mesajlarını gizle |
| `--color` | Eşleşmeyi renklendir |
| `-m` | Maksimum eşleşme sayısı |
| `-z` | Satırları null ile ayır |
| `2> /dev/null` | (shell) STDERR'i yok say — grep'e özgü değil |

## 21. Sonuç

`grep` öğrenirken en önemli şey:

```
grep -rniE "pattern" ./dizin
```

gibi bir komutu ezberlemek değildir.

Asıl önemli olan bunun mantıkta:

```
Dizinde
Büyük/küçük harf duyarsız
Satır numarasıyla
Genişletilmiş regex kullanarak
arama yap
```

anlamına geldiğini bilmektir.

Aynı şekilde:

```
-i  → Harf duyarlılığını kapat
-v  → Tersini göster
-n  → Satır numarası ekle
-c  → Say
-r  → Dizinde ara
-w  → Tam kelime
-o  → Sadece eşleşen kısmı al
-E  → Regex'i genişlet
-A/-B/-C → Bağlamı göster
```

mantığını kavramaktır.

Bu mantık oturduğunda `grep` yalnızca bir Linux komutu olmaktan çıkar ve log analizi, kod incelemesi ve metin işleme süreçlerinde kullandığın temel araçlardan biri haline gelir.
