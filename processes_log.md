# 完整流程
1. 
    1. 開發環境建置，安裝R & RStudio，利用以下指令安裝ChAMP套件
    ```R
      if(!requireNamespace("BiocManager", quietly=TRUE))
        install.packages("BiocManager")
      BiocManager::install("ChAMP)
    ```
      可用以下指令檢查是否安裝成功
    ```R
      library("ChAMP")
    ```
    2. 在RStudio輸入IDAT檔（分為測試集test和訓練集train），經ChAMP套件之norm正規化後輸出存有甲基化後beta值的csv檔。以及用ChAMP.DMP分析輸出各位點資料之csv檔。
    ```R
      myLoad <- champ.load("PATH/train or test")#, SampleCutoff = 0.2)#, arraytype = "EPIC")
      myNorm <- champ.norm(beta=myLoad$beta,plotBMIQ=FALSE,cores=5)#, arraytype = "EPIC")
      write.csv(myNorm,file="all_beta_normalized.csv",quote=F,row.names=T)
      myDMP <- champ.DMP(beta=myNorm, pheno=myLoad$pd$Sample_Group)#, arraytype = "EPIC")
      write.csv(myDMP[[1]], file="DMP_result_TN.csv", quote=F)
    ```
    3. 打開資料集中的sample_sheet，可以知道訓練集前50筆為Normal，後506筆為Tumor；測試集前50筆為Normal，後500筆為Tumor。計算$IQR$(四分位距中$Q_3 - Q_1$)，以$Q_3 + 1.5 \times IQR$為上限、$Q_1 - 1.5 \times IQR$為下限，分別去除Normal和Tumor的離群值。
    4. 計算去離群值後Normal的平均值，然後將每筆資料減去Normal平均值已取得每一個差值。
    5. 計算$\Delta\beta$，其中
        ${\Delta\beta =}$${\Sigma^m_i (\beta _i - \beta _{normal} {avg})}\over{m}$
        且Normal和Tumor的$\Delta\beta$應分開計算。
    6. 去除$\Delta\beta$的離群值，方法同3.。
    7. 將$\Delta\beta$與DMP進行交集。
    8. 設定$\Delta\beta$門檻值，從相同基因中取出$|\Delta\beta|$最高的位點。
    9. 以8.的位點繪製火山圖，設定P-value門檻，取出Hyper及Hypo
