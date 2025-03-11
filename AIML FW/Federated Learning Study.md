# <center>Federated Learning Study</center>

<center></center>

###### tags: `AI/ML`

![](https://i.imgur.com/JORnn3y.png =150x)@NTUST

>[name=Jerry][time=Wed, Jan2 , 2024 3:38 PM][color=#38c16a]

:::info
**Action Item :**

:::
**<center>Table of Content</center>**
[TOC]
## <center>2024/01/09</center>
### 1. What is Federated Learning?

Federated Learning其實就是讓每個用戶用自己的裝置進行訓練，得出「個人小模型」，訓練結果再統整到雲端上，統整出一個「多用戶通用大模型」，藉此用戶可以保有自己資料不用上傳，同時又可以享受到智能化服務。

<font color="#f00">而Federated Learning核心困難也是如何讓溝通的成本變得盡量低。</font>

- 用戶不用上傳資料到雲端，只要把資料存在自己裝置中
- 每個用戶在自己裝置上進行訓練
- 訓練完後把跟資料無關的性質（像是Model weight）上傳至雲端，雲端再統整大模型
- <font color="#f00">我們希望這樣訓練的結果跟把所有用戶資料統整起來一起訓練的結果差不多。</font>

![Screenshot 2024-01-09 at 2.12.52 PM](https://hackmd.io/_uploads/HyYeNvqd6.png)
