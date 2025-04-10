
# プラスチックと心血管疾患データの解析

---

## 📌 セクション 1: データの準備と可視化

### ライブラリの読み込み

以下のコードでは、Rの分析に必要なライブラリ（パッケージ）を読み込みます。

- **tidyverse**: データ操作と可視化を簡単にするライブラリの集合。
- **table1**: データの要約統計を見やすい表形式で表示するライブラリ。

```R
library(tidyverse)
library(table1)
```

### データの読み込み

`read_csv()`関数を使用してCSV形式のデータファイルを読み込みます。

```R
df_plastic <- read_csv("plastic_2025.csv", na = "NA")
```

### データの整形

読み込んだデータを分析しやすい形式に加工します。

- `%>%`: パイプ演算子。左のデータを右の関数へ順次渡します。
- `<-`: 代入演算子。右側の結果を左側の変数に格納します。
- `mutate()`: 新しい変数を作成または既存の変数を変更します。
- `factor()`: 数値や文字列をカテゴリ変数（因子）に変換します。
- `dplyr::select()`: 必要な変数だけを選択します。

```R
data_plastic <- df_plastic %>% 
  mutate(
    MNPありなし = factor(MNP, levels = c(1,0), labels = c("あり","なし")),
    sex = factor(male, levels = c(1,0), labels = c("男性","女性")),
    高血圧 = factor(HTN, levels = c(1,0), labels = c("あり","なし")),
    心血管疾患 = factor(CVD, levels = c(1,0), labels = c("発症","なし"))
  ) %>%
  dplyr::select(rowID, MNPありなし, polyethylene, vinylCL, IL1b, age, sex, 高血圧, 心血管疾患)
```

### データを確認

データの先頭の数行を表示して内容を確認します。

```R
head(data_plastic, n = 30) %>% utils::View(title = "最初の30行")
str(data_plastic)
head(data_plastic, n = 2) %>% utils::View(title = "データ例")
```

### 分布の可視化

`ggplot2`を用いてデータの分布を視覚的に表現します。

- `ggplot()`: グラフの基本設定。
- `aes()`: グラフに使用する変数を指定します。
- `geom_histogram()`: ヒストグラムを描画します。
  - `bins`: ヒストグラムの棒の数を指定。
  - `fill`: 棒の色を指定。
  - `color`: 棒の境界線の色を指定。
  - `alpha`: 透明度を指定（0: 完全透明〜1: 不透明）。
- `geom_boxplot()`: 箱ひげ図を描画します。
  - `width`: 箱の幅を指定。
- `xlim`: 横軸の表示範囲を指定します。

**年齢の分布**

```R
data_plastic %>% 
  ggplot(aes(x = age)) +
  geom_histogram(bins = 30, fill = "#00AFBB", color = "black", alpha = 0.6) +
  scale_x_continuous(limits = c(40, 100)) +
  theme_bw()

data_plastic %>% 
  ggplot(aes(x = age)) +
  geom_boxplot(width = 0.2, fill = "#00AFBB", color = "black", alpha = 0.6) +
  scale_x_continuous(limits = c(40, 100)) +
  theme_bw()
```

---

## 📌 セクション 2: 推測統計

### MNPの有無による心血管疾患の比較

```R
table1(~ 心血管疾患 | MNPありなし, data = data_plastic)
```

### IL1bの推測統計

平均値と標準偏差を計算し、標本平均の標準誤差を算出します。

```R
mean(data_plastic$IL1b)
sd_il1b <- sd(data_plastic$IL1b)
sd_il1b/sqrt(257)
```

### 平均値差の検定

```R
library(broom)

t.test(IL1b ~ MNPありなし, data = data_plastic) %>% 
  broom::tidy() %>% 
  mutate_if(is.numeric, round, 6) %>% 
  utils::View(title = "IL1bの平均値差の検定")
```

### 心血管疾患の割合差の検定

```R
prop.test(table(data_plastic$MNPありなし, data_plastic$心血管疾患)) %>% 
  broom::tidy()
```