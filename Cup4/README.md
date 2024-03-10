## Report

### Kaggle Ranking: 5/20

### StudentID and Name

- 109062135 陳家輝
- 109062117 吳展維
- 109062335 王懷鴻
- 109062108 陳寬宸


### Model

在本次Cup中，我們首先使用了pretrained好的Sentence Embedding對新聞的headline和description做處理，並將兩者的embedding進行加權平均，作為item embedding的初始值。隨後根據user data的history，對user點擊的相關item取出它們的embedding，再做平均作為user embedding的初始值。

模型的選用上，我們首先嘗試了Tutorial中的FunkSVD，在訓練一小段時間後，忽然想到上課老師有提到BiasSVD僅僅加上了item和user的bias就可以取得重大的突破，隨後便稍微修改了一下code將模型變成BiasSVD。

### Data Collection

在對模型進行訓練幾十個epochs後，我們便會跑一次training environment，在與環境互動的過程中做data collection，其中若是有點擊的item，我們就將該item記錄在該user的history裡面，若是都沒有點擊，則將五個item記錄在該user的non_clicks裡面。等到一次training environment互動都結束，就將這些收集到的資訊寫回到json裡面做永久保存。

隨後在做新的訓練時，我們會再初始化一次interaction matrix，將更新過的user_data.json重新讀出，對點擊過的history設為1，沒有點擊的non_clicks設為0，並且我們發現，有的時候user點過的item，在後續又被推薦時可能會出現不點擊的情況，意思就是同一個item可能會同時出現在history和non_clicks，此時我們的做法就是將同時出現的item認定為user會偏好的，只是可能看過了所以不會想再看一次，那麼在設值的時候會將其設為1。

### Training

在訓練上，我們有分為offline和online的learning方式。

- Offline
    
    Offline learning是基於user_data，對interaction matrix進行幾十個epochs的訓練後，會執行一次training environment進行互動，並且在過程中做data collection。這樣跑完1 round後，對更新過的資料重新初始化interaction matrix，再做新的訓練。

- Online

    Online learning是基於在training environment互動時，對user做出的response進行更新。若user有點擊item，則對該user和該item的embeddings進行正向更新，而若user都沒有點擊任何item，則對該五個items對應到該user的embeddings進行負向更新。每互動一次就更新一次，並且僅對特定的user和item進行更新。

- Hybrid

    Offline的好處是訓練較快，較容易取得一定的成果，但缺點是需要收集到很多的資訊才可以支撐的起訓練，否則很容易overfitting。在初期訓練的時候，因為很常發生overfitting的情況，導致training enviroment的互動很快就結束，在data collection上獲得不了太多的新資訊。

    Online的好處則是訓練比較穩定，因為是互動一次就更新一次，對於user的偏好不會太快就擬合，呈現一個比較平滑的訓練趨勢，所以在推薦的過程中往往比Offline可以在環境中待得更久。但是壞處就是收斂的速度很慢。

    所以我們嘗試結合兩者進行Hybrid的訓練。首先基於interaction matrix做Offline訓練一定的epochs，使模型在目前的資訊上具有一定的表現，再進入training environment中進行Online的訓練，幫助模型較平滑的進行更新避免overfitting。在與環境互動結束後，同樣將收集到的資訊寫回json，並在下一次訓練時重新初始化interaction matrix，再重複Offline和Online的訓練。

### Recommendation

在推薦的過程中，我們發現有的時候user點擊過的item在後續重新被推薦時，該user又會不點擊了，我們猜測可能是因為看過的item再次被推薦時user不會想再看一次，所以我們修改了我們的推薦策略，讓其可以基於user點擊過的記錄，不再推薦已經點擊過的item，從而發掘更多user的偏好。我們在每一次訓練時會重新初始化user history為原始僅具有3個點擊記錄的data，而在互動的過程中不斷將點擊item給記錄下來，避免重複推薦。

具體的做法就是在推薦時會預測該user對於所有item的評分，我們將那些點擊過的item的分數手動設成很低，讓其不會再被推薦。此一做法比起一開始只推薦預測評分高的item而不考慮重複性的方法來的好上很多，取得了很重大的突破。