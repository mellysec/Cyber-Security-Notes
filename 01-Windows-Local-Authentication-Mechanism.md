# Windows Kimlik Doğrulama Mexanizmləri, LSASS və NTLM Protokolu

Windows-da iki cür kimlik doğrulama mexanizmi var: **Lokal** və **Domain**. Lokal kimlik doğrulaması yalnız öz komputerimizdə olan bir doğrulama mexanizmidir. Domain-də ise domaine qoşulmuş bütün cihazları əhatə edir. Lolbas adlı bir site var bunu araşdıracam. Hakerlərin və ya apt qrupların ən çox üz tutduqları Windows legitim prosesləri var!
Məsələn:
- `explorer.exe`
- `svchost.exe`
- `rundll.exe`
- `lsass.exe`
- `lsm.exe`
- `smss.exe`
- `csrss.exe`
- `winlogon.exe`

`explorer.exe` prosesindən başqa digər bütün proseslər system32 qovluğunun altında yerləşir. `explorer.exe` ise `C:\Windows`-un altında yerləşir, bu prosesləri başqa bir qovluqda görürsənsə bilki fakedir. Məsələn `C:\Programs Files`, `C:\Users\Public` və s. və s. Eyni zamanda bu proseslərin adlarına da diqqət etmək lazımdır. Hackerlar adlarda cüzi bir şeyləri dəyişərək legitim prosesə özlərini oxşada bilərlər. Məsələn `lsass.exe`, `lsasss.exe` və s. 

Windows-da kimlik doğrulama məlumatları SAM faylında saxlanılmır. Açılışı Security Account Manager, yerləşdiyi ünvan: `C:\Windows\System32\config\SAM`

Bu faylı açmaq, dəyişdirmək, silmək, kopyalamaq olmur. SAM-a sadece iki yolla access etmək olar: birincisi computer sönülü olmalıdır, onun SSD-si çıxarılıb başqa kompa taxılmalı və SAM elə oxunmalıdır. İkincisi ile illegal tool-lar vasitəsi ilə oxuna bilər. Məsələn, mimikatz toolu vasitəsi ilə. 

Windows-da bəs necə olur ki bizim loginimiz doğrular? 
Parollarımız SAM ilə doğrulanır. Amma sistem direk SAM ilə əlaqəyə keçmir. İlk önce yazdığımız parolu `winlogon.exe` prosesi qəbul edir. Daha sonra parolu `lsass.exe`-yə göndərir, `lsass` parolu alır və SAM-a göndərir. SAM-da parollar qarşılaşdırılır, eyni olarsa login uğurlu olur, əks halda xəta verir. 

Burada diqqət etməli olduğumuz hissə parolların SAM-a göndərilib yoxlanılmasından əvvəl `lsass`-a göndərilməsidir. Bu o deməkdır ki, parolların kopiyası `lsass`-da saxlanılır. Bu da attack-ların hədəfinə çevirir. Kritik və həssas prosesdir. `lsass`-dakı məlumatlar silinsə nə olacaq? Comp restart olunacaq və bizi login ekranına atacaq. Bir növ session-umuz sonlanacaq. Bu durumda məlumatlarımız müvəqqəti olaraq RAM-da qalır. Attacker-lər digər SAM-a hücum edə bilmədiyi üçün əsas hədəfi `lsass.exe` olur. Çünki bayaq da qeyd etdiyimiz kimi bəzi məlumatlar burada saxlanılır. Məsələn, cari user-in parol məlumatları. SAM-da isə bütün user-lərin kimlik doğrulama məlumatları saxlanılır. Təbiiki ister SAM-da olsun, istersede `lsass`-da parollar açıq şəkildə saxlanılmır. NTLM hash variantından istifadə olunur. Bu alqoritm əvvəllər NT hash və LM hash formasında ayrı-ayrı alqoritmlər şəklində olub. Bu hash-in iki versiyası var. NTLMv1 çox zəif olduğu üçün istifadə olunmaması məsləhətdir, amma bəzən default olaraq gəldiyi üçün istifadə olunur hələ də. Bunu dəyişmək üçün GPO yazmaq və NTLMv2 məcburi işlənməsini məcburi hala gətirmək lazımdır. NTLM arxa planda MD4 alqoritmindən istifadə edir. Attacker-lər `lsass.exe`-yə hücum edərək `lsass`-ın tərkibində olan NTLM hash-ləri oğurlamağa çalışırlar. NTLM hash oğurlandıqdan sonra qırılır. Brute Force üsulu ilə qırmalısan. Bunun üçün rainbow table attack-dan istifadə olunur. Amma bu attack parol uzun olduğu halda uzun çəkə bilər, ona görə çox zaman hacker-lər Pass the Hash attack-ından istifadə edirlər. Bu hücumun məntiqi hash ilə uğurlu login ola bilməkdir. Tam araşdırmamışam, araşdırarıq bu hissəni. 

Digər bir mövzumuz NTLM protokoludur. Bu, hash-ləmədən tamamilə ayrı bir qavramdır. NTLM protokolu şəkildə kimlik doğrulama protokoludur. ŞƏBƏKƏDƏ! Yeni mütləq iki cihaz olmalıdır. Məsələn, Vəli öz komputerində SMB ilə bir fayl paylaşıb, mən öz kompüterim bu faylı görmək üçün kimliyimi doğrulamalıyam, bunu təmin edən də NTLM protokoludur. Bu protokol kompüterin özündə istifadə olunmur! Sırf şəbəkədə. İleridə bundan daha detallı danışacağıq və Wireshark üzərindən praktiki göstərəcəyəm. 

Təkrardan qayıdaq SAM-a. Okey, əgər bizim parolumuz düzgündürsə və SAM-dakı hash dəyəri ilə uyğunlaşırsa, `lsass.exe` bizə bir access token verir. Əgər bu token olmasaydı, bizim hər hərəkətimizdə, məsələn brauzerə girmək üçün sistem bizdən doğrulama istəyərdi. Əslində bunu yuxarıda qeyd etmişəm, sadəcə kurs materialı da bir az qarışıq olduğu üçün belə not almışam, düzəldərik. 

Mimikatz tool-u. Bu tool vasitəsi ilə biz `lsass.exe`-nin tərkibini oxuya bilərik. Təbii ki, bu illegal bir tool-dur, ona görə ən sadə antivirus tərəfindən belə aşkarlanır, kursda əlavə olaraq zone id-dən danışılıb, bunu araşdırmaq lazımdır detallı, çünki tam başa düşmədim. Cmd emrləri falan göstərildi dir /r sonra more filename.zip məsələn. 

Mimikatz-a geri dönək. Mimikatz cmd komandaları ilə işləmir. Onun xüsusi komandaları var, bu komandaları internetdən və ya AI-dan öyrənmək olar. Və onu da qeyd edim ki, Mimikatz mütləq admin yetkisində açılmalıdır. Biz bilirik ki, RAM-da hər bir proses ayrı olaraq saxlanılır və bir proses digər prosesin fəaliyyətinə qarışa bilməz. Mimikatz isə `lsass.exe`-nin tərkib hissəsini oxumaq üçün RAM-da `lsass`-ın beyninə girməlidir, `lsass.exe` access edilməlidir. Bunu necə edəcəyik? Maraqlı hissə burasıdır. Burada işin içinə Windows-da olan özəl yetkilər girir. Hansı ki, bu yetkilər `Se` ifadesi ilə başlayır. Məsələn: `SeDebugPrivilege`, `SeBackupPrivilege` və s. 

Event ID-də bunların hamısını görmək olur, bunu da praktiki olaraq göstərəcəm. Bu yetkiləri almaq üçün də admin səlahiyyəti lazımdır. Event ID-də bu 4672 status kodu ilə qeyd edilir. 4672-Special Logon.

### Event Viewer - Se- ilə Başlayan Yetkilər və 4672 Event ID
Admin hüquqları ilə daxil olunduqda və xüsusi `Se` yetkiləri təyin edildikdə Event Viewer-də qeydə alınan `4672 (Special Logon)` hadisəsi:
<img width="775" height="593" alt="image" src="https://github.com/user-attachments/assets/6deaa82d-43c8-47c9-ba1e-0d804d9436b4" />


Biz burada mimikatz-a `SeDebugPrivilege` yetkisi verməliyik. Bu yetki ilə bir proses RAM-da digər prosesin tərkibini açıb oxuya bilər. Bəs bu yetki niyə var? Bu yetki legal işlər üçün hazırlanıb və debug prosesi üçün hazırlanıb, yeni xətanı detallı araşdırmaq, test etmək üçün. İndi mimikatz-a bu yetkini verək:
1. Admin kimi açırıq.
2. `privilege::debug` (`SeDebugPrivilege` yetkisi veririk)
3. `sekurlsa::logonpasswords` (`lsass.exe` məlumatlarını oxuyuruq)

### Mimikatz İstifadəsi və Lsass Oxunması
Aşağıdakı skrinşotda Mimikatz vasitəsilə `SeDebugPrivilege` yetkisinin verilməsi və `lsass.exe` yaddaşından NTLM hash-lərin oxunması prosesi əks olunmuşdur:

<img width="1318" height="1039" alt="image" src="https://github.com/user-attachments/assets/2e46ab11-813c-4e5c-a3b7-db0866bd213f" />


`lsass.exe` içərisində `dwm-1`, `dwm-2` kimi system manager user-ləri görə bilərik, bunlara fiziki olaraq giriş mümkün deyil, amma yetkilərindən istifadə edə bilərik. Bu user-lər bizim interface-i yaxşı görünməsini təmin edir, sistem işlərini görür.

NTLMv1 --> DES
NTLMv2 --> HMAC-MD5

Bu protokol challenge-response prinsipinə əsasən işləyir:
1. Sorğu göndərir, NTLM negotiate
2. NTLM challenge
3. NTLM response
4. NTLM auth

Bu protokol parolun hash-inin şəbəkə üzərindən açıq-aşkar göndərilməsinin qarşısını alır. Hash-lənmiş parol şifrələnmiş şəkildə hədəfə çatır. 

Qarşı tərəf bizə challenge göndərir. Bu challenge-də nələr qeyd olunur? Challenge qarşı tərəf bizə random həcmdə data göndərir və bu datanı biz alırıq, məqsədi odur ki, random olan datanı öz parolumuzun hashi ilə encryption edək. Yeni parolumuzun hashini açar kimi istifadə edirik.

Negotiate sorğusunda nələr qeyd olunur? Mən sənlə əlaqə qurmaq istəyirəm, dəstəklədiyim şeylər bunlardır və s. Daha detallı yazarsan.

Client cavab verəndə cavabın tərkibinə özünün Challenge məlumatlarını və Timestamp-ini əlavə edərək göndərir. Bunu etməsinin səbəbi, eyni anda iki cihaz eyni parolu daxil edərək öz kimliyini təsdiq etdikdə yaranacaq eyniliyin və bundan qaynaqlanan NTLM Relay hücumunun qarşısını almaqdır.

Nəticənin tamamilə UNİKAL olmasını təmin etmək üçün Client serverin random datasını öz parolunun HASH-i ilə şifrələyib göndərərkən, içərisinə əlavə olaraq öz challenge-ini (yəni random datasını) və time məlumatını da yazır.

### NTLM Protokolu Trafiki (Wireshark)
Wireshark üzərindən NTLM autentifikasiya prosesinin 4 mərhələsi (`Negotiate`, `Challenge`, `Response`, `Auth`) paket təhlili zamanı belə görünür:

<img width="1280" height="430" alt="image" src="https://github.com/user-attachments/assets/767f03fb-da6c-4296-9159-f4bc2fbe1090" />

