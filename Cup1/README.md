## Report

### Kaggle Ranking: 14/20

### Student ID and Name

- 109062135 陳家輝
- 109062117 吳展維
- 109062335 王懷鴻
- 109062108 陳寬宸

### Featrue Engineering

- Feature Selection

    我們嘗試了很多的feature，例如字數，圖片，正向和負向字詞的統計，或是將feature轉換成有意義的數字，像是author和topic score，或是計算一些feature的比例，像是單獨出現字詞的比例。隨後使用Ridge去找出各個feature的重要程度，並選擇性的刪除一些比較不重要的feature。

- Info Word
    - Title

    - Data Channel

    - Author 
    
    - Related Topics

    - Time (Conver`HH:MM` to `HH * 60 + MM`, then conver to `morning`, `noon`, `afternoon`, `night`)
   
    - Weekday or Weekend
   
    - Word count (`wordshort`, `wordmedium`, `wordlong`)
   
    - Video + Image count (`nomedia`, `fewmedia`, `manymedia`)
   
    - Link + Strong Header Count (`nomedia`, `fewmedia`, `manymedia`)
   
    - Number Count (`fewstats`, `somestats`, `manystats`, `lostats`)
   
    - Positive and Negative Words
- Info Stats

    - Time
    
    - Weekend
    
    - Word count
    
    - Scipt count
    
    - Video + Image count
    
    - Link and Strong count

    - Number count
    
    - Positive and Negative count

    - `?` and `!` count

    - Title length

    - Average word length and Unique word rate

- Content

    在文章內容feature的選擇上，我們將內文中每個段落的第一句和最後一句話選取出來。因為文章的架構一般是起承轉合，一個段落的第一句話起頭，最後一句話總和，恰巧包含了該段落最重要的資訊。

### Model Selection and Traning

- SGDClassifier
    
    直接使用Template裡面的model作為初期的嘗試，發現搭配HashingVectorizer的效果還不錯，後來在Info word model的選用上，表現較其他好或較穩定，所以就被徵召了。

- XGBClassifier

    作為現今主流的classifier的三巨頭之一，甚至可以說為老大的他，在前期的表現卻差強人意，可能是我們選的feature它不喜歡QQ。但後來將眾多feature結合起來餵給它後，在Voting內表現良好。

- LGBMClassifier

    同為現今主流的classifier的三巨頭之一，其表現在Info stats feature上力壓SGD和XGB，並且也比RandomForest穩定。

- RandomForestClassifier

    讓我們又愛又恨的RandomForest，愛是愛他什麼feature train出來score都蠻不錯的，恨就恨在那個score有的時候是假的，雖然已經將tree的數量增加到五百，但有的時候仍然會有偏頗，可最後我們仍然選用其作為Content model。

- GradientBoostingClassifier

    雖然在Info stats feature上單打獨鬥幹不過LGBMClassifier，但是在最終嘗試裡，結合XGBMClassifier在Voting內表現良好。

- VotingClassifier

    可以結合多個model對眾多feature進行訓練，每個內部模型的表現不一，但經過Voting之後可以呈現較好的表現，正所謂人多力量大。

- Voting Ratio

    參照RandomForest的作法，我們將不同feature用不同的model進行訓練，也就是使用Info word, Info num和Content model三者的predict做乘上比例並相加作為最終的預測，透過Ratio分配選擇，選出一個在train和valid表現上都較好的比例作為Final predict的分配。

### Final Approach

    在最後的嘗試裡，我們發現`feature_selection_part2`在單個模型上有比較好的表現，經過多個模型的測試，我們選用VotingClassifier包含XGBClassifier和GradientBoosting Classifier作為最終的模型

### Conclusion
    
    將多個model結合起來最後用比例分配的作法在valid的表現上可以比個別model都要來得好，但在public上卻沒有想像的好，我們猜測可能是因為feature之間並不是獨立的。

    之後的Cup要早點開始，因為每天繳交的次數有限，到後面的時候擔心次數用完，有一些方向不敢就這麼丟上去看是否正確，只能透過不是很準的valid去相信它在test上也可以表現的很好，而且到後面training的時間也會越久，需要更多的時間去嘗試不同的方向。

| train score | valid score | public score | Method                                                                                                                                                                  |
| :---------- | ----------- | :----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0.701       | 0.561       | 0.544        | Feature Hashing + SGDClassifier                                                                                                                                         |
| 0.857       | 0.548       |              | Info word + Feature Hashing + XGBClassifier <br> Info num + GradientBoostingClassifier <br> Content + TFIDF +RandomForestClassifier <br> Proportion: 0, 0.4, 0.6        |
| 0.769       | 0.567       |              | Info word + Feature Hashing + SGDClassifier <br>Info num + GradientBoostingClassifier <br>Content + TFIDF +RandomForestClassifier <br> Proportion: 0.45, 0.1925, 0.3575 |
| 0.888       | 0.576       | 0.550        | Info word + Feature Hashing + RandomForestClassifier<br>Info num + RandomForestClassifier<br>Content + TFIDF +RandomForestClassifier<br>Proportion: 0.4, 0.27, 0.33     |
| 0.857       | 0.548       |              | TFIDF + LogisticRegression                                                                                                                                              |
| 0.627       | 0.553       |              | TFIDF + SGDClassifier                                                                                                                                                   |
| 0.911       | 0.546       |              | TFIDF + XGBClassifier                                                                                                                                                   |
| 0.713       | 0.558       |              | TFIDF + RndomForestClassifier                                                                                                                                           |
| 0.817       | 0.569       | 0.552        | TF-IDF + xgboost + GradientBoostingClassifier                                                                                                                           |
| 0.591       | 0.584       | 0.562        | Feature_selection_part2 + XGBClassifier                                                                                                                                 |
| 0.788       | 0.593       | 0.570        | (Info word + SGDCalssifier) & (Info stats + LGBMClassifier) & (Content + RandomForestClassifier)                                                                        |
| 0.626       | 0.595       | 0.565        | Feature_selection_part2 + VotingClassifier                                                                                                                              |