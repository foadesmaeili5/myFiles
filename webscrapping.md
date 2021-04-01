

<div dir = "rtl">

چگونگی بدست آوردن دیتای از وب سایتها. در نرم افزار R

در این تمرین سعی میکنیم تا داده های جدول اصلی صفحه ی 
[https://www.worldometers.info/coronavirus/](https://www.worldometers.info/coronavirus/)
را در نرم افزار ذخیره و آماده ی تحلیل کنیم.

بسته ی مورد نیاز برای خواندن صفحات وب 
بسته ی rvest
است.
</div>

```{r,comment=NULL,warning=FALSE,message=FALSE}
library(rvest) 
library(dplyr)
```

<div dir = "rtl">
با استفاده از تابع read_html
صفحه ی وب را درون نرم افزار ذخیره میکنیم
</div>

```{r,comment=NULL,warning=FALSE,message=FALSE}
web <- read_html("https://www.worldometers.info/coronavirus/")
web
```

<div  dir = "rtl">
برای بدست آوردن جدول باید از تابع 
html_nodes
استفاده کنیم.
این تابع دو آرگومان مهم دارد.
آرگومان اول یک آبجکت با کلاس 
html_document
که در حقیقت یک صفحه ی وب هست
و دومین آرگومان نیز 
تگ مربوط به بخشی هست که نیاز داریم.
به عنوان مثال:
در بخش بالا همانطور که نشان داده شد 
دو تگ کلی داریم در این سایت
`<head>و<body>`
برای دسترسی به تگ دوم میتوانیم به صورت زیر اقدام کنیم.
</div>

```{r,comment=NULL,warning=FALSE,message=FALSE}
html_nodes(web,"body")
```

<div dir = "rtl">

برای بدست آوردن جدول 
کافی هست که تگ مربوط به جدول را فراخوانی کنیم.

</div>
```{r,comment=NULL,warning=FALSE,message=FALSE}

tbl <- html_nodes(web,"table")
tbl
```
<div dir = "rtl">
از آنجایی که فقط جدول مربوط به امروز را میخواهیم فقط اولی را نگه میداریم
</div>

```{r,comment=NULL,warning=FALSE,message=FALSE}
tbl <- tbl[1]
```

<div dir = "rtl">
برای آنکه بتوانیم تگ های فرزند را ببینیم کافی هست که از تابع 
html_children()
استفاده کنیم.
</div>


```{r,comment=NULL,warning=FALSE,message=FALSE}
html_children(tbl)
```

<div dir = "rtl">
نام سر سطون ها در مولفه ی اول
و مقادیر هر یک از سطون ها نیز در دومین مولفه هست
اینکه هر کدام را چگونه متوجه شدیم را میتوان از کروم استفاده کرد و دید کد های html 
را 
و یا آنکه به صورت زیر هر آنچه که درون آن وجود دارد را ببینیم.
</div>

```{r,comment=NULL,warning=FALSE,message=FALSE}
html_nodes(tbl,"thead") %>%
  html_text()
```

<div dir = "rtl">
همانطور که مشاهده میشود نام سر سطون ها با استفاده از 
یک `\n`
جدا شده است.

بنا بر این میتوانیم داشته باشیم.
</div>


```{r,comment=NULL,warning=FALSE,message=FALSE}

colNAMES <- tbl %>%
     html_nodes("thead") %>%
     html_text() %>%
  strsplit("\n") %>%
  unlist()

```

<div dir = "rtl">

</div>


```{r,comment=NULL,warning=FALSE,message=FALSE}

tbl2 <-
  tbl %>%
  html_nodes("tbody") %>%
  .[1] %>%
  html_nodes("tr") %>%
  as.list()
head(tbl2)
```

<div dir = "rtl">

با تابع
html_text()
به نوشته های داخل متن دسترسی میتوان پیدا کرد.
همانطور که در پایین دیده میشود
داده های اصلی با `\n`
از هم جدا شده اند پس میتوان با این کد داده ها را از هم تفکیک کرد

</div>

```{r,comment=NULL,warning=FALSE,message=FALSE}
tbl2[[1]] %>%
  html_text()
```

```{r,comment=NULL,warning=FALSE,message=FALSE}
res <- list()
length(res) <- length(tbl2)-7
for(i in 8:length(tbl2)) {
  res[[i-7]] <- tbl2[[i]] %>%
  html_text() %>%
    strsplit("\n") %>%
  unlist()
}

res[[1]]
```

```{r,comment=NULL,warning=FALSE,message=FALSE}

x <- sapply(res, length)
x
```

```{r,comment=NULL,warning=FALSE,message=FALSE}
x <- which(x !=18)
res[[x[1]]] <- res[[x[1]]][-c(19:length(res[[x[1]]]))]
res[[x[2]]] <- res[[x[2]]][-19]
res[[x[3]]] <- res[[x[3]]][-12]
# also we know that column 17 is an empty column


res_data <- do.call(rbind,res)
res_data <- res_data[,seq_along(colNAMES)]
colnames(res_data) <- colNAMES
res_data2 <- as.data.frame(res_data,stringsAsFactors = FALSE)
res_data2 <- res_data2[-length(res_data2)]

f <- function(x) {
  if(any(grepl("[+]",x))) {
    x <- gsub("[+]","",x)
  }
  x <- gsub(',', '', x)
  x <- gsub(" ","",x)
}

head(res_data2 )
head(res_data2[-c(1,2,length(res_data2))]) ## just columns with numeric values.
res_data2[-c(1,2,length(res_data2))] <- lapply(res_data2[-c(1,2,length(res_data2))],f)

head(res_data2)
## transform it to numeric values
res_data2[-c(1,2,length(res_data2))] <- lapply(res_data2[-c(1,2,length(res_data2))],as.numeric)

head(res_data2)
```
