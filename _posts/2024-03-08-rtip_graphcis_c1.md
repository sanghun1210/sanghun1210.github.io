---
title: R 그래픽스 Chapter1
author: shlee
date: 2024-03-08 08:32:00 +0800
categories: [Blogging, R_tip]
tags: [Statistics, R_tip]
---

4차 산업 혁명의 기반이 되는 수많은 응용기술(인공지능, IoT, 핀테크, 스마트 테크놀로지 등)은 여전히 데이터를 중심으로 움직이고 있으며, 데이터에 대한 심도 있는 이해 없이는 출발점에 설 자격조차 주어지지 않느다. 단순한 자료 구조나 알고리즘에 대한 논의를 벗어나면, 도메인별로 겪게 되는 다양한 데이터 생성 수집, 정재, 전처리, 후처리, 정규환, 변환, 통합, 학습, 분석, 시각화, 자산화, 제품화 등 모든 프로세스에서 발생할 수많은 문제에 직면하게 된다. 그럴수록 데이터에 대한 튼튼한 기초와 이해가 필요하다. 

R을 이해하면 수학자와 통계학자들의 사고방식을 이해할수 있다. 그리고 또한 R의 장점은 누구보다도 빨리 그리고 아름답게 데이터를 분석하고 시각화해볼 수 있다는 것이다. 

R에서 grapics 패키지는 표준 배포판의 일부로, 다양한 그래픽을 만드는데 필요한 유요한 함수들을 많이 담고 있다. 그리고 tydyverse 패키지의 일부인 ggplot2로 인해 R에서의 그래픽 관련 기능이 크게 확장되었다. 다음은 ggplot2를 사용한 몇가지 사용방법들을 담고 있다. 
데이터 셋과 실제 사용한 코드는 다음 Kaggle 주소를 참고할 수 있다.

![My Kaggle](https://www.kaggle.com/datasets/leesanghun1210/kospi-20240308-csv/code)


```R
library(tidyverse) # metapackage of all tidyverse packages
if(!require(quantmod)) install.packages("quantmod")
library(quantmod)

# Input data files are available in the read-only "../input/" directory
# For example, running this (by clicking run or pressing Shift+Enter) will list all files under the input directory

list.files(path = "../input")
data <- read_csv("/kaggle/input/kospi-20240308-csv/kospi_20240308.csv", show_col_types = FALSE)

summary(data)
```


'kospi-20240308-csv'



          date              open_price     high_price     low_price   
     Min.   :2023-05-15   Min.   :2292   Min.   :2312   Min.   :2274  
     1st Qu.:2023-07-26   1st Qu.:2494   1st Qu.:2510   1st Qu.:2486  
     Median :2023-10-12   Median :2558   Median :2569   Median :2550  
     Mean   :2023-10-09   Mean   :2544   Mean   :2555   Mean   :2530  
     3rd Qu.:2023-12-21   3rd Qu.:2600   3rd Qu.:2612   3rd Qu.:2589  
     Max.   :2024-03-08   Max.   :2681   Max.   :2695   Max.   :2668  
      trade_price       volume         
     Min.   :2278   Min.   :2.894e+08  
     1st Qu.:2497   1st Qu.:4.221e+08  
     Median :2559   Median :4.989e+08  
     Mean   :2542   Mean   :5.199e+08  
     3rd Qu.:2604   3rd Qu.:5.832e+08  
     Max.   :2680   Max.   :1.072e+09  


**데이터 요약 보기**

summary는 데이터의 요약을 보여준다. 요약에 나타난 1st Qu.와 3rd Qu.는 각각 첫 번째와 세번째 사분위수를 말한다. 또 중앙값과 평균 둘 다를 보여주고 있는데, 이러면 비대칭 정도를 빠르게 알수 있어서 좋다. 예를 들어 위에 있는 trade_price(종가)의 중앙값(Median)과 평균(Mean)값을 보면 중앙값이 더 크다. 이를 통해, 로그 정규분포처럼 그래프가 왼쪽으로 기울어 졌다는 것을 알수 있다.  

패키지의 이름이 ggplot2지만 이 패키지 내에서 그래프를 그리는 주요 함수는 ggplot이라고 불린다. ggplot 그래프의 기본적인 요소들을 알아 두어야 한다. 그래프의 특성들을 표현하는 작은 코드 적들을 겹쳐 쌓아서 그래프를 정의하고 생성하는 것을 '그래픽스 문법(grammer of graphics)'이라고 불린다.

**산점도 그리기**

ggplot를 호출한 후, 데이터 프레임을 전달하고 점 도형 함수를 불러와 데이터를 그래프로 그릴수 있다.


```R
df <- data
ggplot(df, aes(x = date, y=trade_price)) + geom_point()
```


    
![png](/assets/img/posts/kospi-with-r-chapter1_files/1.png){:width="80%" height="80%"}


산점도는 새로운 데이터세트를 봤을 때 흔히들 제일 먼저 써 보는 방법이다. x, y가 서로 관계있다는 가정하에, 빠르게 그 관계를 파악하기에도 좋다. 
  ggplot으로 그래프를 그리기 위해서는 ggplot에게 어떤 데이터 프레임을 사용할지, 어떤 종류의 그래프를 그릴 것인지와 어떤 에스테틱(aes)을 쓸 것인지를 알려주어야 한다. 이 경우 aes는 data(데이터프레임)에 있는 어떤 필드가 그래프의 어떤 축에 들어가는지를 정의한다. 그 다음, geom_point라는 명령문이 원하는 그래프가 선이나 다른 종류의 그래프가 아닌, 점 그래프라는 사실을 전달해주는 것이다. 

제목과 레이블 추가하기

ggplot에서 엘리먼트들을 쌓아갈 때는, 각각을 더하기 표시(+)로 잇는다. 다시 말해, 그래픽 요소들을 추가하기 위해서 구문들을 결합시키는 것이다. 다음 코드를 보면 더 확실히 알 수 있다.


```R
ggplot(df, aes(x = date, y=trade_price)) +
    geom_point() + 
    labs(title = "Kospi 2024", x="Date", y="Close")
```


    
![png](/assets/img/posts/kospi-with-r-chapter1_files/2.png){:width="80%" height="80%"}
    


**격자 추가하기**

앞선 레시피에서 보았듯이, ggplot의 배경 격자는 기본 옵션이다. 그러나 theme 함수를 사용해서 배경격자를 변경하거나, 미리 정의된 테마를 그래프에 적용할 수 있다. 
  theme를 사용해서 그래픽의 배경 패널을 수정해 보자. 


```R
ggplot(df) +
    geom_point(aes(x=date, y=trade_price)) + 
    theme(panel.background = element_rect(fill= "white", color = "grey50"))
```


    
![png](/assets/img/posts/kospi-with-r-chapter1_files/3.png){:width="80%" height="80%"}
    


ggplot+ 객체를 생성한 후에, 해당 객체를 호출하고 +를 뒤에 덧붙여 그래픽의 요소들을 추가하거나 변경할 수 있다. ggplot 그래픽의 배경 음영색은 실제로는 세개의 그래프 엘리먼트로 구성되어 있다.

panel.grid.major  주 격자의 기본 옵션은 흰색이고 두껍다.

panel.grid.minor  주 격자의 기본 옵션은 흰색이고 얇다.

panel.background  배경색은 기본 옵션으로 회색이다.


```R
g1 <- ggplot(df) +
    geom_point(aes(x=date, y=trade_price)) + 
    theme(panel.background = element_blank())

g1

g2 <- g1 + theme(panel.grid.major = element_line(color="black", linetype=3)) +
        theme(panel.grid.minor = element_line(color="darkgrey", linetype=4))
g2
```


    
![png](/assets/img/posts/kospi-with-r-chapter1_files/4.png){:width="80%" height="80%"}
    



    
![png](/assets/img/posts/kospi-with-r-chapter1_files/5.png){:width="80%" height="80%"}
    


ggplot 그래프를 g1이라는 변수에 저장한 점을 눈여겨보자. 그 뒤에 g1을 호출해서 그래픽을 출력했다. g1에 그래프를 담아 두게 되면, 그래프를 다시 만들지 않고도 추가적인 그래픽 요소들을 덧붙일 수 있다. 

**ggplot 그래프에 테마 적용하기**

ggplot은 설정 모음인 테마(theme)를 지원한다. 테마 중 하나를 사용하려면 ggplot 객체에 +를 사용해서 원하는 theme 함수를 추가하면 된다. ggplot2 패키지에는 다음과 같은 테마들이 있다.
* theme_bw()
* theme_dark()
* theme_classic()
* theme_gray()
* theme_linedraw()
* theme_light()
* theme_minimal()
* theme_test()



```R
g1 <- ggplot(df, aes(x = date, y=trade_price)) +
    geom_point() + 
    labs(title = "Kospi 2024", x="Date", y="Close")
g1 + theme_bw()
g1 + theme_dark()
g1 + theme_classic()
```


    
![png](/assets/img/posts/kospi-with-r-chapter1_files/6.png){:width="80%" height="80%"}
    



    
![png](/assets/img/posts/kospi-with-r-chapter1_files/7.png){:width="80%" height="80%"}
    


**범례 추가(또는 제거)하기**

그래프 안에 범례, 즉 그래픽의 내용을 해독할 수 있게 해주는 작은 박스가 들어 있었으면 한다.

대부분의 경우 ggplot은 범례를 자동으로 추가해준다. 하지만 aes 함수에서 명시적으로 집단이 매핑되지 않은 경우, ggplot이 자동으로 범례를 보여주지는 않는다. 만약 ggplot이 범례를 보여즈더럭 강제하기를 원한다면 모양 또는 선 종류를 정수로 설정하자. 그러면 ggplot이 하나의 그룹에 대해서도 범례를 보여줄것이다.


```R
ggplot(df, aes(x = date, y=trade_price, shape="Observation")) +
    geom_point() + 
    labs(title = "Kospi 2024", x="Date", y="Close") + 
    guides(shape=guide_legend(title="My Legend Title"))
```


    
![png](/assets/img/posts/kospi-with-r-chapter1_files/8.png){:width="80%" height="80%"}
    


**산점도의 회귀선 그리기**

데이터 쌍들을 그래프에 찍고 있는데, 그들의 선형회귀(linear regression)를 보여주는 선을 그려넣고 싶다.

ggplot에서는 R의 lm함수로 먼저 선형 모형을 계산할 필요가 없다. 그 대신, geom_smooth 함수를 써서 ggplot 호출 내에서 선형회귀를 계산하면 된다.


```R
g1 <- ggplot(df, aes(x = date, y=trade_price, shape="Observation")) +
    geom_point() + 
    labs(title = "Kospi 2024", x="Date", y="Close")
g1 + geom_smooth(method="lm",
                formula= y ~ x,
                se = FALSE)
```


    
![png](/assets/img/posts/kospi-with-r-chapter1_files/9.png){:width="80%" height="80%"}
    


se = FALSE 인자는 ggplot이 회귀선 주위에 표준오차 밴드(error band)를 넣지 않도록 해주는 역할이다.

**모든 변수들간 그래프 그리기**
변수들이 많은 경우는 그들의 상관관계를 찾기 어렵다. 이럴 때 쓸만한 기술 하나는 모든 변수 쌍에 대해 산점도를 들여다보는 것이다. 정석대로 쌍별로 하나씩 생성하려면 무척이나 지겨운 작업이겠지만 GGally 패키지의 ggpairs 함수가 쉽게 모든 산점도를 볼 수 있는 방법을 제공한다.


```R
library(GGally)
ggpairs(df)
```


    
![png](/assets/img/posts/kospi-with-r-chapter1_files/10.png){:width="80%" height="80%"}
    

