
# プラスチック暴露データの解析

---

## 📌 セクション 1: データの準備と可視化

### ライブラリの読み込み

```R
library(tidyverse)
library(table1)
```

### データの読み込み

```R
df_plastic <- read_csv("plastic_2025.csv", na = "NA")
```

### データの整形

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

```R
head(data_plastic, n = 30) %>% utils::View(title = "最初の30行")
str(data_plastic)
head(data_plastic, n = 2) %>% utils::View(title = "データ例")
```

### 分布の可視化

#### 年齢の分布

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

#### IL1bの分布

```R
data_plastic %>% 
  ggplot(aes(x = IL1b)) +
  geom_histogram(bins = 30, fill = "#FFDB6D", color = "#C4961A", alpha = 0.6) +
  theme_bw()

data_plastic %>% 
  ggplot(aes(x = IL1b)) +
  geom_boxplot(width = 0.2, fill = "#FFDB6D", color = "#C4961A", alpha = 0.6) +
  theme_bw()

data_plastic %>% 
  ggplot(aes(x = IL1b)) +
  geom_boxplot(width = 0.2, fill = "#FFDB6D", color = "#C4961A", alpha = 0.6) +
  theme_bw() +
  coord_flip()
```

### 記述統計量

```R
table1(~ age + IL1b, data = data_plastic)
```

---

## 📌 セクション 2: 推測統計

### MNPの有無による心血管疾患の比較

```R
table1(~ 心血管疾患 | MNPありなし, data = data_plastic)
```

### IL1bの推測統計

```R
mean(data_plastic$IL1b)
sd_il1b <- sd(data_plastic$IL1b)
sd_il1b/sqrt(257)

curve(dnorm(x, mean(data_plastic$IL1b), sd_il1b/sqrt(257)), from = 0, to = 1500)

data_plastic %>% 
  ggplot(aes(x = IL1b)) +
  geom_histogram(bins = 30, fill = "#FFDB6D", color = "#C4961A", alpha = 0.6) +
  xlim(0, 1500) +
  theme_bw()
```

### MNP有無によるIL1bの平均値差の検定

```R
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

---

このMarkdown文書は、Rのスクリプトを各ステップごとに詳しく解説し、初学者にも理解しやすいよう工夫されています。
