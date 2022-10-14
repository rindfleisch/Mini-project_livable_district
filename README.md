# Mini-project_livable_district
วัตถุประสงค์คือวิเคราะห์ข้อมูลข้อมูลคะแนนอาหารด้านต่างๆทั้งกลางวันและกลางคืน ข้อมูลคะแนนการเดินทางเข้าถึงสถานที่ต่างๆ ข้อมูลค่ากลางราคาในการเช่าอสังหาริมทรัพย์ และข้อมูลการรายงานปัญหาที่เจอในกรุงเทพและปริมณฑล

# ย้ายเข้ากรุงเทพ 101 เขตน่ารู้ในเการเลือกเช่าที่พัก
คำถามวิจัย

## Import and check data
นำเข้า library และข้อมูลที่จะใช้ ซึ่งได้แก่ 
1. ข้อมูลคะแนนด้านการบริโภค ซึ่งได้แก่ เขตในกรุงเทพ ช่วงกลางวันและกลางคืนของ คะแนนการรวมการบริโภค คะแนนการการบริโภคด้านราคา คะแนนการการบริโภคด้านคุณภาพการบริการ การการบริโภคด้านคุณภาพการบริการ การการบริโภคด้านความหลากหลายของอาหาร
2. คะแนนด้านการเข้าถึงสถานที่ต่างๆในกรุงเทพ ซึ่งได้แก่ เขตในกรุงเทพ คะแนนการเข้าถึงสถานที่ดำเนินชีวิตประจำวันโดยการเดิน คะแนนการเข้าถึงสถานที่ดำเนินชีวิตประจำวันโดยขนส่งสาธารณะ คะแนนการเข้าถึงสถานที่ดำเนินชีวิตประจำวันโดยการขับรถยนต์
3. ราคาประกาศเช่าที่อยู่อาศัยประเภท จัดสรร ราคาแสดงแบบค่ากลาง (median price) ซึ่งได้แก่ ปี(2021) เดือน ชื่อจังหวัด ชื่อเขต รูปแบบอสังหา ราคาเช่าเฉลี่ย/ตรม
4. ข้อมูลรายงานปัญหาที่เจอในกรุงเทพและปริมณฑลจาก traffy ซึ่งได้แก่ ตั๋วข้อมูล เขตในกรุงเทพ มาหาจำนวนที่เกิดปัญหา


```
import pandas as pd
import requests
from io import StringIO
from google.colab import drive
from google.colab import files
import matplotlib.patches as patches
import matplotlib.pyplot as plt
#drive.mount('/content/drive/MyDrive/dads5001_dataset')
drive.mount('/content/drive')
```
```
#Upload data from the url
eating_url = "https://drive.google.com/file/d/10DgxfeivkUSetxbVqqPGuq0g2PiMNCuj/view?usp=sharing"
path_df1 ='https://drive.google.com/uc?id=' + eating_url.split('/')[-2] #problem 1: cannot add path -> have to use .split
df_eating_score = pd.read_csv(path_df1)

path_df2 = "/content/drive/MyDrive/dads5001_dataset/opendata_living_score.csv"#problem 2: cannot read csv file, became<!DOCTYPE html><html><head><title>Google Drive -> Has to mount gg drive or upload
df_living_score = pd.read_csv(path_df2)

path_df3 = "/content/drive/MyDrive/dads5001_dataset/opendata_median_price_rent.csv"
df_median_price_rent = pd.read_csv(path_df3)

path4 = "/content/drive/MyDrive/dads5001_dataset/teamchadchart.csv" 
df_chadchart = pd.read_csv(path4)
```
### Check data
พบว่าข้อมูลส่วนใหญ่ค่อนข้างสะอาดยกเว้นของ traffy ที่มี null 
```
df_eating_score.info() #check data -> all = non-null
RangeIndex: 554 entries, 0 to 553
Data columns (total 19 columns):
 #   Column                    Non-Null Count  Dtype  
---  ------                    --------------  -----  
 0   row_number                554 non-null    int64  
 1   subdistrict_id            554 non-null    int64  
 2   subdistrict_name_en       554 non-null    object 
 3   subdistrict_name_th       554 non-null    object 
 4   district_id               554 non-null    int64  
 5   district_name_en          554 non-null    object 
 6   district_name_th          554 non-null    object 
 7   province_id               554 non-null    int64  
 8   province_name_en          554 non-null    object 
 9   province_name_th          554 non-null    object 
 10  eating_daytime            554 non-null    float64
 11  eating_daytime_price      554 non-null    float64
 12  eating_daytime_quality    554 non-null    float64
 13  eating_daytime_variety    554 non-null    float64
 14  eating_nighttime          554 non-null    float64
 15  eating_nighttime_price    554 non-null    float64
 16  eating_nighttime_quality  554 non-null    float64
 17  eating_nighttime_variety  554 non-null    float64
 18  source                    554 non-null    object 
dtypes: float64(8), int64(4), object(7)

df_living_score.info() #check data -> all = non-null
RangeIndex: 708 entries, 0 to 707
Data columns (total 27 columns):
 #   Column                Non-Null Count  Dtype  
---  ------                --------------  -----  
 0   row_number            708 non-null    int64  
 1   subdistrict_id        708 non-null    int64  
 2   subdistrict_name_en   708 non-null    object 
 3   subdistrict_name_th   708 non-null    object 
 4   district_id           708 non-null    int64  
 5   district_name_en      708 non-null    object 
 6   district_name_th      708 non-null    object 
 7   province_id           708 non-null    int64  
 8   province_name_en      708 non-null    object 
 9   province_name_th      708 non-null    object 
 10  walk_leisure          708 non-null    float64
 11  walk_public_service   708 non-null    float64
 12  walk_school           708 non-null    float64
 13  walk_shopping         708 non-null    float64
 14  walk_work             708 non-null    float64
 15  walk_total            708 non-null    float64
 16  access_boat           708 non-null    float64
 17  access_bus            708 non-null    float64
 18  access_train          708 non-null    float64
 19  access_total          708 non-null    float64
 20  drive_leisure         708 non-null    float64
 21  drive_public_service  708 non-null    float64
 22  drive_school          708 non-null    float64
 23  drive_shopping        708 non-null    float64
 24  drive_work            708 non-null    float64
 25  drive_total           708 non-null    float64
 26  source                708 non-null    object 

df_median_price_rent.info() #check data -> all = non-null
RangeIndex: 1550 entries, 0 to 1549
Data columns (total 11 columns):
 #   Column                        Non-Null Count  Dtype 
---  ------                        --------------  ----- 
 0   row_number                    1550 non-null   int64 
 1   transction_year               1550 non-null   int64 
 2   transction_month              1550 non-null   int64 
 3   listing_province_name_th      1550 non-null   object
 4   listing_province_name_en      1550 non-null   object
 5   listing_district_name_th      1550 non-null   object
 6   listing_district_name_en      1550 non-null   object
 7   project_propertytype_name_th  1550 non-null   object
 8   project_propertytype_name_en  1550 non-null   object
 9   total_listing                 1550 non-null   int64 
 10  median_rent_price_sqm         1550 non-null   int64 
dtypes: int64(5), object(6)

df_chadchart.info() #check data -> has null
RangeIndex: 122163 entries, 0 to 122162
Data columns (total 12 columns):
 #   Column        Non-Null Count   Dtype 
---  ------        --------------   ----- 
 0   ticket_id     122109 non-null  object
 1   type          68412 non-null   object
 2   organization  122109 non-null  object
 3   comment       121957 non-null  object
 4   coords        122163 non-null  object
 5   photo         122163 non-null  object
 6   address       122109 non-null  object
 7   district      121519 non-null  object
 8   subdistrict   121521 non-null  object
 9   province      122101 non-null  object
 10  timestamp     122163 non-null  object
 11  state         122132 non-null  object
dtypes: object(12)
```


## Filter and clean data
### Filter
ทำการเลือกข้อมูลที่ต้องการจะใช้จากข้อมูลด้านบนซึ่งเลือกจากในเขตกรุงเทพเท่านั้น

### ***ใส่รูป***


### Aggreate and merge
คิดค่าเฉลี่ยของของค่ากลางในแต่ละเขต และจำนวนเคสที่ถูกรายงานแยกเป็นรายเขตและรวมข้อมูลไว้ด้วยกัน ซึ่งจากเขตจำนวน 50 เขต เมื่อนำมารวมกับค่าเช่าต่อตารางเมตรปี 2021 จะพบว่ามีเพียง 37 เขตเท่านั้น และนำทุกข้อมูลมาหาค่าเฉลี่ยแบ่งเป็นประเภท เช่น การบริโภค การเดินทาง และนำมาคำนวณรวมกันอีกครั้ง จากนั้นทำการ normalize ข้อมูลแต่ละอัน ได้แก่่ คะแนนรวมของการกินและการเดินทาง และเขตที่จำนวนเคสน้อยที่สุดและถูกที่สุด นำมารวมกันหาค่าเฉลี่ย จะได้ค่าเฉลี่ยคะแนนที่ดีที่สุดจากทั้ง 3 ด้าน 


### ***ใส่รูป***

```

Int64Index: 118 entries, 0 to 117
Data columns (total 16 columns):
 #   Column                                     Non-Null Count  Dtype  
---  ------                                     --------------  -----  
 0   subdistrict_name_en                        118 non-null    object 
 1   district_name_en                           118 non-null    object 
 2   province_name_en                           118 non-null    object 
 3   eating_daytime                             118 non-null    float64
 4   eating_daytime_price                       118 non-null    float64
 5   eating_daytime_quality                     118 non-null    float64
 6   eating_daytime_variety                     118 non-null    float64
 7   eating_nighttime                           118 non-null    float64
 8   eating_nighttime_price                     118 non-null    float64
 9   eating_nighttime_quality                   118 non-null    float64
 10  eating_nighttime_variety                   118 non-null    float64
 11  walk_total                                 118 non-null    float64
 12  access_total                               118 non-null    float64
 13  drive_total                                118 non-null    float64
 14  mean_median_rent_price_sqm_district(2021)  118 non-null    float64
 15  No.case                                    118 non-null    int64  
dtypes: float64(12), int64(1), object(3)
```




## Visualization
จากข้อมูลสามารถตอบคำถามวิจัยได้ดังนี้
1.Q:มีเขตไหนบ้างในกรุงเทพที่น่าอยู่ที่สุดถึงน้อยที่สุดจากคะแนนรวมด้านอาหาร ด้านการเดินทาง ที่มากที่สุด และราคากลาง และจำนวนปัญหาน้อยที่สุด  A: ข้อมูลที่มีอันดับสูงที่สุด 5 อันดับแรกได้แก่ บางกอกใหญ่(83.46%) ธนบุรี(81.98%) บางคอแหลม(79.86%) คลองสาน(78.31%) และวังทองหลาง(77.04%) ส่วนอันดับต่ำที่สุด 5 อันดับหลังได้แก่ จตุจักร(40.66%) บางขุนเทียน(51.26%) ลาดกระบัง(52.76%) บางแค(54.34%) และบางเขน(56.16%) 


### All district ranking
***รูป***


### top and bottom 5 livable districts 
***รูป***




### top and bottom 5 eating districts 
2.Q: มีเขตไหนบ้างในกรุงเทพที่น่าอยู่ที่สุดและน้อยที่สุดจากคะแนนรวมด้านอาหาร A: 5 อันดับแรกได้แก่ พญาไท(77.19%) วัฒนา(75.83%) ปทุมวัน(75.66%) บางรัก(74.9%) และคลองเตย(74.83%) ส่วนอันดับต่ำที่สุด 5 อันดับหลังได้แก่ จตุจักร(52.10%) บางขุนเทียน(53.60%) ลาดกระบัง(58.88%) บางแค(59.72%) และบางเขน(59.79%) 
***รูป***

### top and bottom 5 travel districts 
3.	Q: มีเขตไหนบ้างในกรุงเทพที่น่าอยู่ที่สุดและน้อยที่สุดจากคะแนนรวมด้านการเดินทาง A: 5 อันดับแรกได้แก่ บางรัก(99%) ราชเทวี(98.92%)  ปทุมวัน(98.83%)  คลองสาน(98.58%) และธนบุรี(98.50%) ส่วนอันดับต่ำที่สุด 5 อันดับหลังได้แก่ บางขุนเทียน(62.50%) สายไหม(67.55%) ลาดกระบัง(69.17%) มีนบุรี(69.33%) บางแค(69.50%)
***รูป***


### top and bottom 5 price of the districts 
4.	Q: มีเขตไหนบ้างในกรุงเทพที่ราคาคอนโดต่อตารางเมตรแพงและถูกที่สุด  A: 5 อันดับที่แพงที่สุดได้แก่ ปทุมวัน(737.67) บางรัก(647.42) วัฒนา(635.67) คลองเตย(632.5) และสาธร(586) และ 5 อันดับที่ถูกที่สุด ได้แก่ มีนบุรี(228) ประเวศ(242.67) ลาดกระบัง(244) บึงกุ่ม(250.5) และบางแค(275.67)
***รูป***



