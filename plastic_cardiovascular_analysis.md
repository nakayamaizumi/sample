
# プラスチックと心血管疾患データの解析

---

## 📌 セクション 1: データの準備と可視化

### Web R 

<a href="https://webr.r-wasm.org/latest/" target="_blank">Web R website にアクセス</a>

演習用データ(plastic_2025.csv)をWeb Rにuploadする

<img src="images/file upload on web R.png" alt="WebRにデータをupload" width="600px">


### ライブラリの読み込み

以下のコードでは、Rの分析に必要なライブラリ（パッケージ）を読み込みます。

- **tidyverse**: データ操作と可視化を簡単にするライブラリの集合。
- **table1**: データの要約統計を見やすい表形式で表示するライブラリ。
- **DT**: データを表示したり操作したりするためのライブラリ。
- **plotly**: 描画したグラフを見やすく表示するライブラリ。
    
```R
library(tidyverse)
library(table1)
library(DT)
library(plotly)
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
- `head()`: データの先頭数行を表示します。
- `str()`: データの構造(structure)を概観します。
- `datatable()`: データを操作できる形で表示します。
```R
#windows
head(data_plastic, n = 30) %>% utils::View(title = "最初の30行")

#mac
head(data_plastic, n = 30) %>% datatable()

str(data_plastic)
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
- `ggplotly`: ggplot関数で作成したグラフをweb表示します。
  
**年齢の分布**

```R

age_hist <- data_plastic %>% 
  ggplot(aes(x = age)) +
  geom_histogram(bins = 30, fill = "#00AFBB", color = "black", alpha = 0.6) +
  scale_x_continuous(limits = c(40, 100)) +
  theme_bw()

ggplotly(age_hist)

age_box <- data_plastic %>% 
  ggplot(aes(x = age)) +
  geom_boxplot(width = 0.2, fill = "#00AFBB", color = "black", alpha = 0.6) +
  scale_x_continuous(limits = c(40, 100)) +
  theme_bw()

ggplotly(age_box)
```

**IL1bの分布**

```R
il1b_hist <- data_plastic %>% 
  ggplot(aes(x = IL1b)) +
  geom_histogram(bins = 30, fill = "#FFDB6D", color = "#C4961A", alpha = 0.6) +
  theme_bw()

ggplotly(il1b_hist)

il1b_box <- data_plastic %>% 
  ggplot(aes(x = IL1b)) +
  geom_boxplot(width = 0.2, fill = "#FFDB6D", color = "#C4961A", alpha = 0.6) +
  theme_bw()

ggplotly(il1b_box)
```
### 記述統計量
各変数の統計要約を表で表示します。
- `table1()`: 各変数の要約値を算出し、表形式で出力します。

```R
table1(~ age + IL1b, data = data_plastic)
```
---

## 📌 セクション 2: 推測統計
---

### IL1bの推測統計

平均値と標準偏差を計算し、標本平均の標準誤差を算出します。
- `mean()`: 算術平均を算出。
- `sd()`: 標準偏差を算出。
- `sqrt()`: 平方根を算出。

```R
mean(data_plastic$IL1b)
sd_il1b <- sd(data_plastic$IL1b)
sd_il1b/sqrt(257)
```

**IL1bの標本平均の分布の可視化**
- `curve()`: 指定した曲線を描く。
- `dnorm()`: 平均と標準偏差をinputすると、対応する正規分布を描く。

```R
# mac/safari の場合は省略
curve(dnorm(x, mean(data_plastic$IL1b), sd_il1b/sqrt(257)), from = 0, to = 1500)
```
**IL1bの分布の可視化（再掲）**

```R
ggplotly(il1b_hist)
```

### MNP有無によるIL1bの平均値差の検定

- **broom**: 統計結果を整然データに変換するパッケージ。
- `tidy()`: 統計解析の結果を整然（tidy）な形式に変換します。

```R
library(broom)

t.test(IL1b ~ MNPありなし, data = data_plastic) %>% 
  broom::tidy() %>% 
  mutate_if(is.numeric, round, 6) %>% 
  datatable()
```

### MNP有無による心血管疾患の割合差の検定

```R
prop.test(table(data_plastic$MNPありなし, data_plastic$心血管疾患)) %>% 
  broom::tidy() %>% 
  datatable()
```
