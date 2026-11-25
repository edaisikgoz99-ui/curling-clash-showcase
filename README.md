# Curling Clash — Proje Tanıtımı

> **Bu depo bir tanıtımdır.** Oyunun kaynak kodu, görselleri ve sesleri özel bir
> depoda tutulur; burada yalnızca projenin mimarisi, teknik yaklaşımı ve ekran
> görüntüleri yer alır. Kod incelemesi için erişim talep üzerine verilir.

Curling Clash, Flutter ve Flame ile geliştirilmiş, dikey ekran mobil bir curling
oyunudur. Taş tek bir fizik dünyasında kayar; aynı dünya, hazır bir 3B motor
kullanılmadan yazılmış perspektif kamera sistemiyle atıcının gözünden ve üç
yayın açısından çizilir. Dört ligin her birinin kendi ışıklandırması olan
ayrı bir salonu vardır.

![Ana menü, lig seçimi, oyun modu ve taş seçimi](images/akis.jpg)

## Öne çıkanlar

| | |
|---|---|
| **Kendi 3B çizim hattı** | Salonun tamamı (buz, LED panolar, 14 basamaklı tribünler, çatı makasları, duvarlar) Flutter `Canvas` üzerinde, elle yazılmış bir iğne deliği kamerayla projekte edilir. Hazır 3B motor yoktur. |
| **Tek dünya, tek fizik, dört bakış** | Atıcının gözü, takip kamerası, home arkası kamera ve tepe görünümü aynı dünya koordinatlarını okur. Kamera değişince hiçbir nesne yeniden oluşturulmaz. |
| **Dört lig, dört salon** | Oyun her ligde aynıdır; lig yükseldikçe saha %12 uzar ve salonun ışıkları renklenir: akan LED şeritler, renkli tavan ışıkları, tribünlerin üstünde gezen spotlar, seyircide ışık çubukları. |
| **Yaşayan salon** | Bariyerler LED ekrandır ve atışa göre mesaj verir; yayın kameralarının başlıkları taşı izler; tavandaki flamalar salınır; seyirci heyecana göre ayağa kalkıp zıplar, bayrak sallar. |
| **Renderlanmış süpürücüler** | Süpürücüler Blender'da pozlanıp oyunun kamera açılarından renderlanır, formaları taşın rengine boyanır. Durum makinesi (bekleme, kayma, süpürme, hızlı süpürme, toparlanma) taşın hızına ve süpürme gücüne bağlıdır. |
| **Fizik ile sunumun ayrılması** | Fizik yazar, çizim yalnızca okur. Çizim katmanı taş konumlarını hiçbir koşulda değiştirmez. |
| **Malzeme sistemi ve toplu çizim** | Çelik tribünler, dört ayrıntı seviyesinde önceden süzülmüş bir doku atlasından, dünya koordinatlı doku eşlemesiyle çizilir. Binlerce seyirci ve bütün ışık huzmeleri hazır köşe tamponlarına yazılır: tribün başına tek, ışık için tek çizim çağrısı. |
| **İki süpürücülü ritim kontrolü** | Ekranda düğme yoktur: ekranın iki yarısı iki süpürücüdür. Vuruş ritmi sürtünmeyi, iki taraf arasındaki fark taşın yönünü belirler. |
| **Prosedürel ses** | Oyunun ses efektleri kayıt kullanılmadan, sinüs bileşenleri ve süzülmüş gürültüden kodla üretilir; seyircinin alkış ve tepkileri lisanslı kayıtlardan kesilmiştir. |
| **Tam oyun döngüsü** | Altın ekonomisi, taş yükseltmeleri, dört lig, günlük görevler, bilgisayar rakip, yerel iki oyunculu mod, adım adım öğretici ve Türkçe / İngilizce dil desteği. |

Ölçek: 149 Dart dosyası, yaklaşık 42.000 satır; salonun çizimi tek başına
5.200 satır. Kamera projeksiyonu, süpürme modeli, lig salonları, altın ve
ekonomi kuralları 350'den fazla birim testiyle doğrulanır.

## Oynanış

![Atış ekranı, takip kamerası, süpürme ve home arkası kamera](images/kameralar.jpg)

1. **Atış:** taş tutulur, aşağı çekilerek güç, yana kaydırılarak yön verilir ve bırakılır.
2. **Takip:** kamera taşın arkasından ilerler; salon akar, home uzaktan yaklaşır.
3. **Süpürme:** ekranın sol yarısı sol süpürücüyü, sağ yarısı sağ süpürücüyü çalıştırır. Yoldaki altınlar toplanır.
4. **Home:** taş home'a yaklaşınca yayın home arkası kameraya geçer; taşlar durunca sonuç tepeden gösterilir ve halkaya göre puan verilir. İyi atışta LED panolar mesaj verir; rakibin taşını home'dan çıkaran atış rozetini ve altınını alır.

```mermaid
stateDiagram-v2
    [*] --> TakipKamerasi: yeni taş
    TakipKamerasi --> ArkaKamera: taş home'a yaklaştı
    ArkaKamera --> TepeKamerasi: tüm taşlar durdu
    TepeKamerasi --> TakipKamerasi: sıradaki taş
    TepeKamerasi --> [*]: maç bitti
```

## Ligler ve salon

![Dört ligin salonu: Çaylak, Profesyonel, Elit, Buz Efsanesi](images/ligler.jpg)

Dört lig de oynanabilir; lig puanı yalnızca sıralamayı belirler. Buz, çizgiler
ve fizik her ligde aynıdır. Lig yükseldikçe saha biraz uzar (×1,00 / 1,12 /
1,24 / 1,36) ve salon daha renkli ışıklandırılır. LED panoların rengi buzun
yalnızca kenarlarına yansır; home ve atış hattı her ligde temiz buzdur.

## Mimari

```mermaid
flowchart TD
    subgraph UI["Flutter arayüzü"]
        screens["Ekranlar: menü, mod, taş seçimi, mağaza"]
        overlays["Oyun içi arayüz: atış ekranı, skor tablosu, altın göstergesi"]
        tutorial["Öğretici sistemi"]
    end
    subgraph GAME["Oyun çekirdeği"]
        game["Oyun döngüsü (FlameGame)"]
        turn["Tur ve set yönetimi"]
        physics["Taş fiziği"]
        sweep["Süpürme ritmi"]
        ai["Bilgisayar rakip"]
    end
    subgraph RENDER["Sunum"]
        renderer["Salon çizimi"]
        camera["Perspektif kamera"]
        audio["Ses"]
    end
    subgraph DATA["Kurallar ve veri"]
        config["Saha ve lig ayarları"]
        economy["Altın, yükseltme, puanlama"]
    end
    UI --> game
    game --> turn
    game --> physics
    game --> sweep
    game --> ai
    game --> renderer
    game --> audio
    renderer --> camera
    physics --> config
    renderer --> config
    game --> economy
```

Her karede sıra sabittir: girdi → süpürme ritmi → fizik → çarpışmalar ve altın
toplama → kamera → çizim.

```mermaid
sequenceDiagram
    participant G as Girdi
    participant S as Süpürme ritmi
    participant P as Taş fiziği
    participant C as Oyun döngüsü
    participant R as Çizim
    G->>S: dokunma / tuş
    C->>S: güncelle
    S-->>C: süpürme gücü, net yönlendirme
    C->>P: güç ve yön
    C->>P: güncelle
    P-->>C: yeni hız ve konum
    C->>C: çarpışmalar, altın toplama
    C->>R: kamerayı güncelle ve çiz
```

## Çizim hattı

Salon iki aşamada kurulur. Önce geometri: her yüzey dünya koordinatlarında
tanımlanır ve kameradan projekte edilir. Düz renkli görünüm, yüzeylerin aynı
köşe noktalarında birleştiğini ve salonun kapalı bir hacim olduğunu doğrulamak
için kullanılır. Sonra aynı köşe noktalarının üstüne doku, malzeme, seyirci,
sis ve ışık gelir.

![Geometri ve son görüntü, iki kamera için yan yana](images/render.jpg)

Kamera, optik ekseni yatay olan bir iğne deliği modelidir. Ufuk ekranda sabit
bir satırdır; her nesnenin boyutu yalnızca kameraya uzaklığından gelir:

```dart
Offset _at(double wx, double wz, double d) {
  final s = focal / d;                                   // piksel / dünya birimi
  final sx = screenCenterX - forwardSign * (wx - x) * s; // yön işareti kamerayı çevirir
  final sy = horizonY + (height - wz) * s;
  return Offset(sx, sy);
}
```

Teknik notlar:

- **Dokular dünyaya bağlıdır.** Doku koordinatları ekran konumundan değil dünya
  konumundan üretilir; kamera ilerlerken buz ve çelik deseni yerinde kalır.
- **Perspektif düzeltmesi.** Uzun yüzeyler derinlik boyunca şeritlere bölünür;
  şerit sınırları sabit bir dünya ızgarasına oturur, böylece aydınlatma
  kamerayla birlikte kaymaz.
- **Ayrıntı seviyeleri.** Çelik doku 1, 2, 4 ve 8 texellik ortalamalarla
  önceden süzülür; uzaktaki basamak zaten yumuşatılmış şeridi örnekler ve ince
  kenar parlaması titremek yerine sönümlenir.
- **Çizim sırası.** Derinlik tamponu yoktur; katmanlar uzaktan yakına sabit
  sırayla çizilir.
- **Toplu çizim.** Seyirciler ve ışık huzmeleri tek tek çizilmez; köşeleri
  hazır tamponlara yazılır ve tek çağrıda çizilir. Resmin dışında kalan
  flama, kamera ve huzmeler atlanır.

```mermaid
flowchart LR
    A[Çatı] --> B[Bitiş duvarı] --> C[Tribünler ve seyirciler]
    C --> C2[Flamalar ve yayın kameraları] --> D[Buz]
    D --> E[Çizgiler ve home] --> F[LED panolar] --> G[Sis ve ışık]
    G --> H[Altınlar, taşlar, süpürücüler]
```

## Taş fiziği

Uzunluk birimi oyunun dünya birimidir (yaklaşık 290 birim = 1 metre).

**Sürtünme.** Taş sabit bir ivmeyle yavaşlar; yön korunur. Lig uzadıkça ivme
aynı oranda küçülür, böylece aynı atış gücü sahanın aynı oransal noktasına düşer.

```
a₀ = 280 / ligÇarpanı × (2,1 / 2,8)
v  ← v − a · dt
d  = v² / (2a)
```

**Süpürme.** Süpürme taşa hız eklemez, yalnızca hız kaybını azaltır. Taşın
yükseltme seviyesi (`u`) tam süpürmedeki kaybı daha da düşürür.

```
p = sınırla(0,6 · (p_sol + p_sağ) + 0,1 · min(p_sol, p_sağ), 0, 1)
f = 0,36 − 0,20 · u
a = a₀ · (1,45 + (f − 1,45) · p)
```

| Durum (1000 birim/s hız) | Çarpan | Kalan mesafe (birim) |
|---|---|---|
| Süpürme yok | 1,45 | 1 642 |
| Tam süpürme | 0,36 | 6 614 |
| Tam süpürme, tam yükseltme | 0,16 | 14 881 |

**Yönlendirme.** Süpürücüler arasındaki güç farkı taşı çevirir; hızın büyüklüğü
korunur ve yanal pay sınırlıdır.

```
yön = sınırla(0,22 · (p_sol − p_sağ) + doğrudanGirdi, −1, 1)
v_x ← sınırla(v_x + yön · 560 · dt, −0,42 · |v|, +0,42 · |v|)
```

**Çarpışma.** Taşlar eşit kütleli dairelerdir; itki çarpışma normali boyunca
uygulanır ve toplam momentum korunur.

```
J = (1 + e) · ((v₁ − v₂) · n) / 2        e = 0,92
v₁' = v₁ − J · n        v₂' = v₂ + J · n
```

## Süpürme kontrolü

```mermaid
flowchart TD
    in["Ekranın sol / sağ yarısı"] --> tap{"Basılı tutma süresi"}
    tap -->|kısa| one["Tek vuruş"]
    tap -->|uzun| auto["Otomatik hızlı vuruş"]
    one --> rate["Her taraf için vuruş hızı"]
    auto --> rate
    rate --> lp["Sol güç, sağ güç"]
    lp --> fr["Sürtünme: iki tarafın toplamı"]
    lp --> st["Yön: iki taraf arasındaki fark"]
```

Sol süpürücü taşı sağa, sağ süpürücü sola iter; iki taraf eşit süpürürse taş
düz gider ve en uzağa ulaşır. Süpürücüler bariyere dayandığında taş ortaya
doğru geri itilir ve kamera kısa bir sarsıntı verir.

## Ses

Her olayın kendi sesi vardır: taşı tutma, güç ve yön çentikleri, bırakma,
hıza bağlı kayma uğultusu, fırça vuruşları, tek altın ve çoklu altın, taş
çarpışması, puan, seyirci ve maç sonu. Kısa efektler sabit sayıda düşük
gecikmeli oynatıcıdan çalınır; aynı anda birden çok ses birbirini kesmeden
duyulur ve uzun bir süpürme ses kanallarını tüketmez. Atış sürerken stadyum
uğultusu alkışın altında kalır. Efektleri üreten betik projenin parçasıdır,
bu yüzden ses seti tek komutla yeniden üretilebilir veya ayarlanabilir.

## Ekonomi ve ilerleme

| Olay | Altın |
|---|---|
| Buzdaki altın | +5 |
| Dış halka / beyaz / kırmızı / merkez | +5 / +10 / +15 / +25 |
| Rakip taşı home'dan çıkarmak | +50 (altınlar tek tek keseye uçar) |
| Maçı kazanmak | +100 |
| Günlük görevler | +30 / +50 / +100 |

Kazanılan altınla taşların doğruluk ve süpürme etkisi yükseltilir, yeni taşlar
açılır. Lig yükseldikçe saha uzar ve bilgisayar rakip güçlenir.

## Teknoloji

- Flutter (Dart), Flame oyun döngüsü
- Çizim: `Canvas.drawVertices`, `Vertices.raw`, `ImageShader`, çalışma zamanında üretilen doku atlası
- Görseller: süpürücüler, yayın kamerası ve flamalar Blender (Cycles) ile renderlanır
- Ses: düşük gecikmeli oynatıcılar, Python ile üretilen ses seti
- Hedefler: Android ve web

---

© ZZ Software. Tüm hakları saklıdır. Bu depodaki metin ve görseller yalnızca
tanıtım amacıyla paylaşılmıştır; kopyalanamaz, dağıtılamaz ve ticari amaçla
kullanılamaz.
