# TN3K Thyroid Nodule Segmentation

Bu yüksek lisans projesinde TN3K tiroid ultrason görüntülerindeki nodül bölgelerini piksel düzeyinde segmentledim. Veri denetimini, ön işleme analizini, Base U-Net ve U-Net 3+ eğitimlerini ve modellerin ortak test kümesindeki karşılaştırmasını tamamladım.

U-Net 3+, **614 ortak test görüntüsünde %77,77 ortalama Dice** elde etti. Base U-Net’in ortalama Dice’ı **%74,98** oldu. İki modelin tahminlerini görüntü düzeyinde eşleştirerek skor farklarını ve seçili örneklerdeki piksel hatalarını da inceledim.

## Veri seti ve ayrım

| Alt küme | Görüntü–maske çifti |
|---|---:|
| Eğitim | 2.303 |
| Doğrulama | 576 |
| Resmî test | 614 |
| **Toplam** | **3.493** |

Veri setinin sağladığı 614 test çiftini nihai değerlendirme için ayırdım. Kalan 2.879 `trainval` çiftini yaklaşık %80 eğitim ve %20 doğrulama olarak böldüm. Her iki deneyde de aynı üyeliği kullandım ve `verified_split_manifest.csv` dosyasının SHA-256 değerini kontrol ettim.

- 3.493 çiftin tamamı okunabildi. Eksik eşleşme, boyut uyumsuzluğu veya manifestle içerik uyuşmazlığı bulunmadı.
- Alt kümeler arasında piksel içeriği birebir aynı görüntü bulunmadı. Hasta kimlikleri bulunmadığı için aynı hastanın farklı çekimlerinin alt kümeler arasında dağılıp dağılmadığı doğrulanamadı.
- `trainval` bölümünde medyan nodül alanı görüntü alanının %8,57’siydi. 2.879 maskenin tamamında 64, 128 ve 192 eşiklerinde işaretli piksel sayısı değişmedi.

[TN3K Kaggle dağıtımı](https://www.kaggle.com/datasets/tjahan/tn3k-thyroid-nodule-region-segmentation-dataset/data) · [Kaynak makale](https://pubmed.ncbi.nlm.nih.gov/36812810/)

## Ön işleme

1. Görüntüyü aynı veri bölümündeki referans maskesiyle eşledim ve boyutlarını kontrol ettim.
2. Ultrasonu gri tonlu okudum. Maske değerlerini 128 eşiğiyle nodül (`1`) ve arka plan (`0`) olarak etiketledim.
3. Eğitimde görüntü ve maskeyi %50 olasılıkla birlikte yatay çevirdim. Doğrulama ve testte veri artırma kapalıydı.
4. Ultrason yoğunluklarını 255’e bölerek 0–1 aralığına ölçekledim. Yüksekliği ve genişliği ayrı ayrı 32’nin katına tamamlamak için sağa ve alta sıfır dolgu ekledim.
5. Değişken boyutlu görüntüleri `batch_size=1` ile işledim. Dolgu alanını geçerli alan maskesiyle kayıp hesabından çıkardım. Testte 0,5 tahmin eşiğini kullandım ve metrikleri özgün görüntü alanında hesapladım.

Dolgu kaldırıldığında **2.879 eğitim ve doğrulama çiftinin tamamında** ölçeklenmiş görüntü ve ikili maske dolgu öncesiyle piksel düzeyinde eşleşti. Örneğin `test/0000.jpg` için boyut dönüşümü **332 × 365 → 352 × 384 → 332 × 365** oldu (yükseklik × genişlik). Veri hazırlama notebook’u ayrıca EDA’daki 256 × 256 yeniden boyutlandırma denemesini gösterir. Bu inceleme, dolgunun özgün piksel ızgarasını koruduğunu doğrular. Dolgunun Dice üzerindeki etkisini ölçmek için aynı eğitim koşullarında ayrı bir karşılaştırma gerekir.

## Eğitim ve test

- **Base U-Net:** Doğrulama Dice’ına göre seçilen **27. epoch** ağırlığını 614 test görüntüsünde değerlendirdim.
- **U-Net 3+:** Yerel CPU’daki ilk denemenin ardından Tesla T4 üzerinde sıfırdan ayrı bir deney başlattım. Tamamlanan epoch’ları Drive’a kaydederek eğitimi sürdürebildim. Erken durdurma **82. epoch** sonunda devreye girdi. **62. epoch** ağırlığı **%79,04 doğrulama Dice’ı** ile final test için seçildi.

### Resmî test sonuçları

| Metrik | Base U-Net · epoch 27 | U-Net 3+ · epoch 62 | U-Net 3+ farkı |
|---|---:|---:|---:|
| Ortalama Dice ↑ | %74,98 | **%77,77** | +2,79 yüzde puan |
| Ortalama IoU ↑ | %64,13 | **%67,51** | +3,38 yüzde puan |
| Ortalama HD95 ↓ | 93,57 px | **81,04 px** | −12,53 px |

Dice ve IoU, tahmin ile referans maskenin alan örtüşmesini ölçer. HD95, maske sınırları arasındaki mesafeyi özgün görüntü pikseli cinsinden özetler. Her iki modelde de 614 görüntünün tamamında HD95 sonluydu.

Görüntü bazlı Dice karşılaştırmasında U-Net 3+ **388 görüntüde**, Base U-Net **225 görüntüde** öndeydi ve **1 görüntüde** skorlar eşitti. Medyan Dice farkı U-Net 3+ lehine **1,46 yüzde puandı**. `0031.jpg` örneğinde Dice 0,151’den 0,831’e yükselirken `0117.jpg` örneğinde 0,844’ten 0,348’e düştü. Piksel bazlı hata haritaları, bu farkları referans dışında işaretlenen ve kaçırılan alanlar üzerinden gösterir.

Karşılaştırma aynı görüntüler, ön işleme, tahmin eşiği ve metriklerle değerlendirilen iki checkpoint’i kapsar. Eğitim bütçeleri farklı olduğu için ölçülen farkı yalnızca mimariye atfetmek mümkün değildir.

## Notebooks

| Notebook | Description |
|---|---|
| [`TN3K_Thyroid_EDA.ipynb`](notebooks/TN3K_Thyroid_EDA.ipynb) | Audits image–mask pairs, image dimensions, mask thresholds, and nodule area distributions. |
| [`TN3K_Modele_Veri_Hazirlama.ipynb`](notebooks/TN3K_Modele_Veri_Hazirlama.ipynb) | Visualizes preprocessing and checks that removing zero padding restores the original pixel grid. |
| [`TN3K_UNet3Plus_Egitim.ipynb`](notebooks/TN3K_UNet3Plus_Egitim.ipynb) | Trains U-Net 3+ from scratch in Colab with resumable Drive checkpoints, validation Dice, and early stopping. |
| [`TN3K_Base_UNet_Epoch27_Test_Colab.ipynb`](notebooks/TN3K_Base_UNet_Epoch27_Test_Colab.ipynb) | Verifies the shared split and evaluates the epoch 27 Base U-Net checkpoint on the official test set. |
| [`TN3K_BaseUNet_vs_UNet3Plus_Test_Karsilastirmasi.ipynb`](notebooks/TN3K_BaseUNet_vs_UNet3Plus_Test_Karsilastirmasi.ipynb) | Compares the checkpoints on paired test images and visualizes pixel-level errors on selected cases. |

## Çalıştırma

**Windows ve VS Code.** Veriyi indirip `trainval-image`, `trainval-mask`, `test-image` ve `test-mask` klasörlerini doğrudan `data/tn3k/` altına çıkarın. Sanal ortamı oluşturun ve bağımlılıkları kurun.

```powershell
py -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

VS Code’da Python ve Jupyter eklentileriyle [EDA notebook’unu](notebooks/TN3K_Thyroid_EDA.ipynb) açın, `.venv` kernel’ını seçin ve **Run All** ile çalıştırın.

**Google Colab.** Ön işleme analizi CPU’da çalışabilir. U-Net 3+ eğitim notebook’u Tesla T4 kullanır. Colab notebook’larındaki ilk ayar hücresinde Drive, veri kökü, doğrulanmış manifest ve gerektiğinde checkpoint yollarını kendi ortamınıza göre düzenleyin. Test için eğitilmiş model ağırlıkları gerekir. Kaydedilmiş görüntü bazlı CSV’ler, modelleri yeniden çalıştırmadan eşleştirilmiş metrik analizini yapmaya yeter. Görsel hata analizi ayrıca referans ve tahmin maskelerini kullanır.

## Çıktılar

Test çalıştırmaları `summary.json`, `per_image_metrics.csv`, doğrulanmış manifest ve özgün çözünürlükte tahmin maskeleri üretir. Karşılaştırma notebook’u bu kayıtları kullanarak özet metrik tablosu, görüntü bazlı fark dağılımları ve seçili örneklerin piksel hata görsellerini oluşturur.
