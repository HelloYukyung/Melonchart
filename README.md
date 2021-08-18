# analysis_of_kpop_lyrics_by_genre

장르별 한국대중음악 가사 분석

</br>

# Data collection and preprocessing

---
## 0. 크롤링하려는 웹 페이지 및 데이터 
크롤링 웹 페이지 :melon 홈페이지 장르별 순위권에 있는 장르별 top 50 데이터 <br> 

크롤링 할 데이터 :가수, 제목, 가사 

## 1. 가수, 제목 크롤링 
참고 : melon_crawling_title&singer.ipynb

1. 사용 tool 소개<br>
BeautifulSoup

2. 크롤링<br>
soup와 정규식을 사용해 크롤링
~~~python 
song_title = soup.find_all('div', attrs={'class': 'ellipsis rank01'})
song_list= []

for i in song_title :
    print(i.text)
    song_list.append(i.text)
~~~
3. 데이터 저장 <br>
데이터 프레임 형태로 저장 
~~~python
df = pd.DataFrame({'곡명':song_list,'가수명':singer_list,"좋아요_수":like_count_list})
df.to_csv('chart crawling.csv', encoding='utf-8')
~~~

## 2. 가사 크롤링 
참고 :melon_crawling_lyrics.ipynb

1. 사용 tool 소개<br>
Selenium

2. 크롤링<br>
xpath를 사용해 노래고유번호(songid)값을 불러온 뒤 , 
for 문을 통해 "https://www.melon.com/song/detail.htm?songId=" songId뒤에 고유번호를 입력하여 가사를 가져옴
~~~python
number=[]

for i in range(1, 51):
    tmp_xpath = "/html/body/div/div[3]/div/div/div[5]/form/div/table/tbody/tr[" + str(i) + "]/td[1]/div"
    tmp = driver.find_element_by_xpath(tmp_xpath)
    print(tmp.find_element_by_class_name('input_check ').get_attribute("value"))
    number.append(tmp.find_element_by_class_name('input_check ').get_attribute("value"))

LYRIC=[]
for i in number:
    driver.get("https://www.melon.com/song/detail.htm?songId=" + i)
    lyric=driver.find_element_by_class_name("lyric")
    LYRIC.append(lyric.text)
    
~~~
3. 데이터 저장 <br>
데이터 프레임 형태로 melon_lyrics.xlsx에 저장 
~~~python
df=pd.DataFrame({"가사":LYRIC})
df.to_excel("melon_lyrics.xlsx",  encoding='utf-8')
~~~

</br>

# Visualization
참고 : lyrics analysis.ipynb
---
colab 환경에서 진행 
1. 사용 tool 소개<br>
konply-mecab, pandas, wordcloud, matplotlib

2. 형태소 분석 <br>
은전한닢(mecab)을 사용한 형태소 분석 <br>
가사를 nouns, pos, morphs별로 출력
~~~python
tag = Mecab()
para = lyrics
print(tag.nouns(para))
print(tag.pos(para))
print(tag.morphs(para))
~~~
가사 데이터(nouns)를 counter하여 명사가 가장 많은 개수를 세어보기
한글자(ex. 너, 나, 등)는 제거 
~~~python
nouns = tag.nouns(para)
nouns = [n for n in nouns if len(n) > 1]

# Counter(nouns)
count = Counter(nouns)

# count_top10
tags = count.most_common(10)
tags
~~~
![image](https://user-images.githubusercontent.com/49505843/129848354-8228582d-325e-4d30-8f13-1008e1996061.png)


이후, nltk를 사용하여 빈도수 plt를 그려보려 했으나 한글이 깨지는 에러가 발생<br>
여러차례 시도해보았지만 실패,, 

~~~python
common = Text(tag.nouns(para), name="common")
common.plot(10)
fig = plt.figure(figsize=(20,20))
plt.rc('font', family='NanumBarunGothic') 
plt.show()
~~~

3. Wordcloud<br>
wordcloud와 matplotlib을 사용해 가사 빈도수 시각화 

~~~python
wordcloud = WordCloud(font_path=my_path, 
                      background_color='white', 
                      width=1200, 
                      height=800).generate_from_frequencies(dict(count))
~~~
~~~python
fig = plt.figure(figsize=(10,10))
plt.imshow(wordcloud, interpolation='bilinear')
plt.axis('off')
plt.show()
~~~
![image](https://user-images.githubusercontent.com/49505843/129848259-82bc7339-6d79-425e-b0fd-4c2b4f40f324.png)

</br>

# Conclusions and Inferences

발라드 장르의 워드 클라우드 결과, 위와 같은 결과가 나왔다. <br>
예상했다시피(?) 발라드에서는 '사랑'단어가 제일 많이 나왔던 것을 볼 수 있었으며 <br>
그다음으로는 '그대', '우리', '시간' 등의 순으로 많이 등장한 것을 확인할 수 있었다.<br>
단어만 보았을 때, 딱 발라드나 사랑 노래가 연상이 되지 않은가💗<br>
나중에 발라드 장르의 가사를 만든다면 위 워드 클라우드 내의 단어를 참고해도 좋을 것 같다. 




---

