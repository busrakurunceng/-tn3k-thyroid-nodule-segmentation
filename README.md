# TN3K Thyroid Nodule Segmentation

Bu yüksek lisans projesi, TN3K tiroid ultrason görüntülerindeki
nodül bölgelerini piksel düzeyinde segmentlemeyi amaçlar.

## Mevcut durum

Veri denetimi ve keşifsel analiz tamamlandı. Notebook, görüntü–maske
eşleşmelerini, görüntü boyutlarını, maske eşiklerini, nodül alanı
dağılımını ve trainval–test arasındaki birebir görüntü tekrarlarını
inceler. Model eğitimi sonraki aşamada yapılacaktır.

## Veri seti

[TN3K Kaggle dağıtımı](https://www.kaggle.com/datasets/tjahan/tn3k-thyroid-nodule-region-segmentation-dataset/data) ·
[Kaynak makale](https://pubmed.ncbi.nlm.nih.gov/36812810/)

Veriyi indirip `trainval-image`, `trainval-mask`, `test-image` ve
`test-mask` klasörlerini doğrudan `data/tn3k/` altına çıkarın.
Sağlanan test bölümü nihai model değerlendirmesi için ayrılmıştır.

## Windows ve VS Code kurulumu

```powershell
py -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

VS Code'da Python ve Jupyter eklentilerini kurun.
[EDA notebook'unu](notebooks/01_tn3k_eda.ipynb) açın,
`.venv` kernel'ını seçin ve **Run All** ile çalıştırın.

## İlk EDA bulguları

| Kontrol | Sonuç |
|---|---:|
| Trainval görüntü–maske çifti | 2.879 |
| Test görüntü–maske çifti | 614 |
| Eksik eşleşme, okunamayan çift, boyut uyumsuzluğu | Her biri 0 |
| Trainval medyan nodül alanı | %8,57 |
| 64, 128 ve 192 eşiklerinde değişen trainval maskesi | 0 |

## Sonraki aşama

Doğrudan resize ile en-boy oranını koruyan resize + padding
yöntemlerini karşılaştırmak, küçük maskelerde piksel kaybını ölçmek
ve temel U-Net modelini Dice ve IoU ile değerlendirmek.