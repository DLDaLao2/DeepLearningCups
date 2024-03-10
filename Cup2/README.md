## Report

### Kaggle Ranking: 16/18

### StudentID and Name

- 109062135 陳家輝
- 109062117 吳展維
- 109062335 王懷鴻
- 109062108 陳寬宸

### Data Augmentation

- Mosaic

    在本次Cup當中，我們有嘗試實作了幾種數據增強的方法，例如MixUp，CutMix及Mosaic，前兩者在產生圖片後進行實驗的結果並沒有很符合預期，所以最終決定使用Mosaic做為主要的數據增強的方法。

    Mosaic是YOLOv4所提出的數據增強方法，其概念類似CutMix，也是將不同圖片做裁減過後拼接在一起的方法，但與CutMix不同的點在於，CutMix拼接後的圖片會遮蔽原圖片，在一定程度上可能會造成辨識變的困難，而Mosaic則是界定好拼接的邊界，將四張圖片做一定縮放後以序放入四個位置，而在邊界處理上，也會將被裁減到的object依據剩下的面積大小決定是否保留bounding box，這樣就可以避免可能裁減後只剩下一個耳朵的狗也會有bounding box，反而造成模型訓練的困難。

    在YOLOv4中有提到，因為Mosaic一次結合了四張圖片，可以很好的豐富檢測物體的背景，並且在BatchNormalization的計算中，可以一次就計算四張圖片的數據，對於收斂有一定的幫助。

    同時，因為是offline產生圖片，所以我們也可以對data進行類別平衡，簡單的作法便是選取幾種特定類別的圖片進行Mosaic合成。而因為具有隨機性，隨意產生出來的圖片基本上都不會相同，可以很好的豐富多樣性。
    
- Transform

    我們有實作多種Transform的技巧如下所列，其中比較麻煩的像是flip和rotation，因為圖片的方向改變，所以對應bounding boxes的位置也要跟著改變，這樣才可以確保不會產生出錯的data反而造成反效果

    - random_brightness
    - random_contrast
    - random_hue
    - random_saturation
    - random_jpeg_quality
    - random_flip_left_right
    - random_flip_up_down
    - random_rot_90_270

### Model Selection

在模型選擇上，我們有嘗試過多種不同的模型。一開始是使用template中原本的YOLOv1進行測試實驗，在經過200個epochs過後，score只能來到0.83，後續我們則是嘗試使用pretrained好的CNN模型進行feature extraction，再接上YOLOv1後面的4層Conv及2層FC做為整個model，如下所列。

- YOLOv1
- DenseNet201 + YOLOv1_design
- ResNet50 + YOLOv1_design
- VGG19 + YOLOv1_design
- Xception + YOLOv1_design
- RestNet101v2 + YOLOv1_design

其中，我們最終的模型選用是DenseNet201 + YOLOv1_design，經過一系列的Fine tune後score來到0.51

### Non Max Suppression

在最後輸出的部分，我們有使用non max suppression的方法，來避免同一個物件被預測出多個bounding box，用threshold來過濾掉confidence較低的bounding box，並且將重疊率較高的bounding box去除。

### Fine Tune and Training

訓練過程的Fine tune我們主要是調整loss weight去指引模型往哪些不足的地方去繼續加強，並且因為模型訓練過程loss weight的不同，所以在predict的取向會大不一樣，有些會嘗試predict大的bounding box，而有些則可能會嘗試predict小而多的bounding box，那麼我們在做predict的時候就要去調整iou和confidence的threshold去讓產生出來的predict結果是比較好的。

### Difficulty

因為使用Mosaic進行offline的數據增強，所以訓練一個epochs的時間會變久，那麼整個訓練過程就會變的很漫長，並且訓練到後期的時候，loss的下降非常緩慢，表現的提升也看不太出差別。

而在經過幾小時漫長的訓練過後，模型的表現仍然沒有很符合預期，即使嘗試過實作更多data augment，更換模型，加入focal loss，但仍然無法達到一個好的結果，這使得我們在後面不知道該如何繼續精進。

### Conclution

本次cup的訓練過程比起cup1拉長了很多，而且後續的cup可能還會更長，所以在訓練過程的策略需要進行一定的調整，因為在經過fine tune過後的模型表現會差很多，我們不應該在前中期一直執著於想要直接train出一個表現很好的model，應該早一點開始對模型進行loss weight和threshold的fine tune。
