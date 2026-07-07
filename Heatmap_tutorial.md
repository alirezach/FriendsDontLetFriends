<div dir="rtl" align="right">

# مقدمه

این گردش‌کار درباره‌ی این است که چگونه به‌صورت برنامه‌نویسی‌شده سطرها و ستون‌های یک نقشه‌ی حرارتی را بازچینش کنیم.
نقشه‌ی حرارتی روشی همه‌کاره برای مصورسازی بسیاری از انواع داده است.
با این حال، برای این‌که نقشه‌های حرارتی کارآمد باشند، باید بازچینش سطرها و ستون‌ها را در نظر بگیریم.
برای نمونه، [این شکل](https://github.com/alirezach/FriendsDontLetFriends#s5) را ببینید.

# بسته‌های موردنیاز

```{r}
library(tidyverse) 
library(patchwork) # not actually required, loaded for this tutorial only 

library(RColorBrewer)
```

# داده

ما از این داده‌ی نمونه که از پژوهش خودم است استفاده خواهیم کرد.
این یک همکاری میان [آزمایشگاه Buell](https://buell-lab.github.io/) در دانشگاه جورجیا و [آزمایشگاه O'Connor](https://www.ice.mpg.de/) در انستیتوی ماکس پلانک است.

نگران این نباشید که این جدول درباره‌ی چیست، اما این گردش‌کار باید با هر نوع داده‌ای کار کند.
این یک گردش‌کار tidyverse است، پس داده‌ی ورودی باید در قالب مرتب (tidy) باشد.
برای اطلاعات بیشتر درباره‌ی داده‌ی مرتب، [این آموزش](https://r4ds.had.co.nz/tidy-data.html) را ببینید.

```{r}
my_data <- read_csv("../Data/heatmap_example.csv", col_types = cols())

head(my_data)
```

`row` و `col` سطرها و ستون‌های نقشه‌ی حرارتی خواهند بود.
ستون `value` برای رنگ‌آمیزی نقشه‌ی حرارتی به کار خواهد رفت.

# بدون بازچینش

بیایید ببینیم اگر سطرها و ستون‌ها را بازچینش نکنیم چه اتفاقی می‌افتد.

```{r}
no_reorder <- my_data %>% 
  ggplot(aes(x = col, y = row)) +
  geom_tile(aes(fill = value)) +
  scale_fill_gradientn(colors = brewer.pal(9, "YlGnBu")) +
  theme_classic() +
  theme(axis.text.y.left = element_blank(),
        axis.ticks.y.left = element_blank())

no_reorder

ggsave("../Results/Heatmap_no_reorder.svg", height = 3.5, width = 4.2, bg = "white")
ggsave("../Results/Heatmap_no_reorder.png", height = 3.5, width = 4.2, bg = "white")
```
![heatmap_no_reorder](https://github.com/alirezach/FriendsDontLetFriends/blob/main/Results/Heatmap_no_reorder.png)

همان‌طور که می‌بینید، هیچ الگویی برای تشخیص در اینجا وجود ندارد.

# ابتدا یک بُعد را بازچینش کنید

برای این‌که تشخیص اطلاعات پنهان در نقشه‌ی حرارتی آسان‌تر شود، باید سطرها و ستون‌ها را بازچینش کنیم.
رویکرد من این است که ابتدا یکی از ابعاد (سطرها یا ستون‌ها) را بازچینش کنم.
اغلب می‌توانیم یک بُعد را بر پایه‌ی معنایی عملی بازچینش کنیم.
برای مثال، آیا یکی از ابعاد یک سری زمانی است؟ مراحل رشد؟ شرایط/تیمارهای آزمایشی مختلف؟

## یک داده‌ی نمونه که در آن ستون‌ها معنای عملی دارند.

فرض کنیم آزمایشی با ۱۰ ژن و ۶ نمونه داریم.

```{r}
my_data2 <- expand.grid(
  genes = letters[1:10],
  samples = 1:6)
```

و فرض کنیم نمونه‌های ۱، ۳، ۵ شاهد و نمونه‌های ۲، ۴، ۶ تیمار هستند.
و فرض کنیم می‌خواهیم آن را طوری بازچینش کنیم که شاهدها ابتدا (سمت چپ) در نقشه‌ی حرارتی ظاهر شوند.

```{r}
my_data2 <- my_data2 %>% 
  mutate(samples = factor(samples, 
                          levels = c(1, 3, 5, 2, 4, 6))) 

head(my_data2)
```

برای (باز)چینش یک ستون از data frame، می‌توانید از `factor()` استفاده کنید.
درون تابع `factor()`، ترتیب دلخواه را در `levels = c(...)` تایپ می‌کنید.

حالا، وقتی بروید و آن را رسم کنید، چه خواهیم دید؟

```{r}
my_data2 %>% 
  ggplot(aes(x = samples, y = genes)) +
  theme_classic()

ggsave("../Results/Reorder_1_dim.svg", height = 3, width = 3, bg = "white")
ggsave("../Results/Reorder_1_dim.png", height = 3, width = 3, bg = "white")
```
![Reorder_one_dimension](https://github.com/alirezach/FriendsDontLetFriends/blob/main/Results/Reorder_1_dim.png)

حالا می‌توانید ببینید که محور x به این ترتیب `1, 3, 5, 2, 4, 6` است، که همان ترتیبی است که مشخص کردیم.

# بازچینش برنامه‌نویسی‌شده‌ی سطرها و ستون‌ها

اگر نخواهیم ترتیب را به‌صورت دستی وارد کنیم چه؟
می‌توانیم آن‌ها را با استفاده از همان آماره‌های پایه بازچینش کنیم: __مقدار قله__ و __تعداد مقادیر قله__.

* مقدار قله: یک سطر مشخص در کدام ستون به بیشینه‌ی مقدار خود می‌رسد؟
* تعداد مقادیر قله: برای هر ستون مشخص، چند سطر به بیشینه‌ی مقدار خود می‌رسند؟

راهی بسیار ساده اما مؤثر برای چینش سطرها و ستون‌ها این است که ستون‌ها را بر پایه‌ی تعداد مقادیر قله بازچینش کنیم.
پس از آن، سطرها را بر پایه‌ی مقادیر قله بازچینش کنیم.

1. ستون‌هایی که بیشترین سطرهای رسیده‌به‌قله را دارند ابتدا (سمت چپ نقشه‌ی حرارتی) ظاهر می‌شوند.
2. ستون‌هایی که کمترین سطرهای رسیده‌به‌قله را دارند در پایان (سمت راست نقشه‌ی حرارتی) ظاهر می‌شوند.
3. سطرهایی که برای ستون نخست به قله می‌رسند در بالا ظاهر می‌شوند.
4. سطرهایی که برای ستون آخر به قله می‌رسند در پایین ظاهر می‌شوند.

شاید درک نحوه‌ی نمود این موضوع با یک نقشه‌ی حرارتی واقعی آسان‌تر باشد.

## یافتن مقادیر قله و تعداد مقادیر قله

بیایید به نمونه‌ی `my_data` خود بازگردیم.
برای یافتن مقدار قله‌ی هر سطر، از `group_by(row)` و سپس `slice_max()` استفاده کنید.

```{r}
my_data_peak_values <- my_data %>% 
  group_by(row) %>% 
  slice_max(order_by = value, n = 1, with_ties = F) %>% 
  rename(peaked_at = col) %>% 
  select(-value)

head(my_data_peak_values)
```

حالا یک data frame داریم که هر سطر آن، سطری است که در نقشه‌ی حرارتی ظاهر خواهد شد.
برای هر سطر، اطلاعاتی درباره‌ی این‌که در کدام ستونِ نقشه‌ی حرارتی به قله می‌رسد وجود دارد.

برای یافتن این‌که چند سطر در هر ستون به قله می‌رسند، از `group_by(column)` و سپس `count()` استفاده کنید.

```{r}
number_of_peaks <- my_data_peak_values %>% 
  group_by(peaked_at) %>% 
  count() %>% 
  arrange(-n)

head(number_of_peaks) 
```
حالا یک data frame داریم که هر سطر آن، ستونی است که در نقشه‌ی حرارتی ظاهر خواهد شد.
و اطلاعاتی داریم درباره‌ی این‌که چند سطر در آن ستون به قله رسیده‌اند.

حالا سطرها را بر پایه‌ی ستون‌ها بازچینش خواهیم کرد.

```{r}
my_data_peak_values_reordered <- my_data_peak_values %>% 
  inner_join(number_of_peaks, by = "peaked_at") %>% 
  arrange(-n)  

my_data_peak_values_reordered
```

پس از یافتن این آماره‌های پایه، می‌توانیم این آماره‌ها را به data frame اصلی خود پیوند بزنیم.

```{r}
my_data_reordered <- my_data %>% 
  inner_join(number_of_peaks, by = c("col" = "peaked_at")) %>% 
  mutate(col = reorder(col, -n)) %>%  # this reorders the columns
  select(-n) %>% 
  inner_join(my_data_peak_values_reordered, by = "row") %>% 
  mutate(peaked_at = reorder(peaked_at, n)) %>% 
  mutate(order_rows = as.numeric(peaked_at)) %>% 
  mutate(row = reorder(row, order_rows)) # this reorders the rows by the "peaked_at" column. 
  

head(my_data_reordered)
```

# حالا می‌توانیم آن را رسم کنیم

```{r}
reordered_heatmap <- my_data_reordered %>% 
  ggplot(aes(x = col, y = row)) +
  geom_tile(aes(fill = value)) +
  scale_fill_gradientn(colors = brewer.pal(9, "YlGnBu")) +
  theme_classic() +
  theme(axis.text.y.left = element_blank(),
        axis.ticks.y.left = element_blank())

reordered_heatmap
```

... و یک مقایسه‌ی پهلوبه‌پهلو انجام دهیم.

```{r}
wrap_plots(no_reorder+ 
             labs(title = "Not reordered"), 
           reordered_heatmap +
             labs(title = "Rows/columns reordered"),
           guides = "collect") 

ggsave("../Results/Heatmap_reorder.svg", height = 3.5, width = 8, bg = "white")
ggsave("../Results/Heatmap_reorder.png", height = 3.5, width = 8, bg = "white")
```
![Side_by_side](https://github.com/alirezach/FriendsDontLetFriends/blob/main/Results/Heatmap_reorder.png)

حالا می‌توانیم ببینیم که یک الگوی روشن پدیدار شد.
هر ستون گروهی از سطرها را دارد که به تراز قله می‌رسند.
در مجموع، نقشه‌ی حرارتی نیز درک آن بسیار آسان‌تر و پرمعناتر است.

</div>
