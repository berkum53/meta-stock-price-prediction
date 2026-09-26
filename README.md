# META hisse fiyatı tahmini

Bu projede PyTorch ile LSTM ve GRU kullanarak META hissesinin bir sonraki işlem günündeki kapanışını tahmin ettim. Başlangıç fikri [Rodolfo Saldanha'nın LSTM–GRU örneği](https://medium.com/swlh/stock-price-prediction-with-pytorch-37f52ae84632). Oradaki Amazon fiyatları yerine Yahoo Finance üzerinden alınan META verisini kullandım. Finansal ekonometri dersindeki getiri yaklaşımını da aynı verilere uygulayarak sonuçları karşılaştırdım.

## Ne yaptım?

- **Fiyat deneyi:** Son 20 kapanış fiyatından bir sonraki kapanışı tahmin eden LSTM ve GRU.
- **Getiri deneyi:** Son 20 log getiriden ertesi günün getirisini tahmin eden LSTM ve GRU. Tahmin edilen getiri, son bilinen fiyatla tekrar dolar cinsinden fiyata çevriliyor.
- **Kıyaslar:** 20 geçmiş getirili doğrusal regresyon (AR(20)) ve ertesi gün için son kapanışı aynen kullanan basit tahmin.

2015-01-02 ile 2026-09-23 arasındaki 2.948 gözlem zaman sırasıyla eğitim (%64), doğrulama (%16) ve değerlendirme (%20) olarak ayrıldı. Ölçekleme yalnızca eğitim dönemiyle öğrenildi. Değerlendirme döneminde 590 gün var. Hedef olarak `Close` kullanılıyor; hesaplanan getiriler temettüleri içeren toplam getiri değil. Her ağın girdisi 20 günlük bir pencere; iki tekrarlayan katman ve 32 gizli birim kullanılıyor. Adam ile MSE en aza indiriliyor, doğrulama hatasına göre ağırlıklar seçiliyor.

## Sonuçlar

| Yöntem | RMSE ($) | R² | Son fiyat kıyasına göre R² |
| --- | ---: | ---: | ---: |
| Son fiyatı kullan | **14,59** | 0,9642 | 0 |
| Fiyat LSTM | 86,50 | -0,2578 | -34,14 |
| Fiyat GRU | 73,33 | 0,0959 | -24,26 |
| Getiri AR(20) | 15,00 | 0,9622 | -0,0567 |
| Getiri LSTM | 14,67 | 0,9638 | -0,0102 |
| Getiri GRU | 14,72 | 0,9636 | -0,0170 |

Getiri modelleri doğrudan fiyat modellerinden daha düşük hata verdi. Ancak basit son fiyat tahminini hiçbiri geçmedi. Fiyat tahmininde tek başına yüksek R² yanıltıcı olabiliyor: basit kıyasın da R² değeri 0,9642. Bu yüzden RMSE ve son fiyat kıyasına göre R²'yi birlikte değerlendirdim. Eğitim dönemindeki ADF testinde log fiyat için birim kök hipotezi reddedilemedi, log getiri için reddedildi. Ljung–Box sonuçları tahmin hatalarının ayrıca incelenmesi için `results/` altında.

Gönderilen AMZN örneğinde GRU RMSE 5,99 $ ve LSTM RMSE 8,56 $ görünüyor; farklı hisse ve tarih aralığı olduğu için bunları META hatalarıyla doğrudan sıralayamıyorum. META'nın eğitim döneminde en yüksek kapanışı 382,18 $, değerlendirme döneminde en düşük kapanışı 453,41 $. Doğrudan fiyat modellerinin ilerideki daha yüksek seviyelere uyum sağlayamaması büyük hata için makul bir açıklama. AMZN örneğinde ölçekleyici eğitim/test ayrımından önce bütün veriye uygulanmış; sonraki dönemin fiyat aralığı hazırlığa karıştığından o sonuçları bizim ayrımla eşdeğer kabul etmiyorum. AMZN örneğinin son fiyat kıyası verilmediği için 5,99 $'lık hatanın basit tahminden iyi olup olmadığını da bilmiyorum.

Getiri modellerini ilk fiyat sonuçlarını gördükten sonra ekledim. Dolayısıyla aynı değerlendirme dönemindeki getiri kıyası keşif amaçlı; bağımsız yeni bir veri dönemiyle teyit edilmedi. Tek hisse, tek ayrım ve tek rastgele başlangıç kullanıldı. İşlem maliyetleri ve yatırım getirisi hesaplanmadı.
