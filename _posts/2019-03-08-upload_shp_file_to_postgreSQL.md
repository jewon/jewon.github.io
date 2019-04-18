---
layout: post
title:  "[오픈소스GIS] postgreSQL에 shp파일 올려보기(with. postGIS)"
date:   2018-03-08 18:00:00 +0900
categories: OSGIS
---

[OSGEO](https://www.osgeo.kr/) 교육자료를 우연히 습득해서 이것저것 해보는 중. 오픈소스 GIS서비스를 만들 때에는 DB로 보통 postgreSQL기반의 postGIS를 쓰고, 백단에 geoserver를 올려서 서비스하는 게 일반적인 것 같다. 오늘은 먼저 postgreSQL, postGIS를 설치하고, 일반적으로 공간 데이터로 사용되는 포맷인 shp를 올려 보는 것 까지 해보자.  

[https://postgis.net/workshops/postgis-intro/](https://postgis.net/workshops/postgis-intro/)
오늘 할 일은 여기서 잘 설명해 주고 있다. 링크 안에서 4, 5번이 오늘 할 일에 해당한다. (그리고 이 페이지는 그것을 번역만 한 것과 다를 게 없다) 단, 이 곳의 pgAdmin은 3버전이다.  
먼저 postgreSQL을 설치해준다. postgreSQL 역시 오픈소스이다.

[https://www.postgresql.org/](https://www.postgresql.org/)
살펴 보니 그냥 편하게 binary로 설치해주면 될 것 같다. 아래 링크다.

[https://www.enterprisedb.com/downloads/postgres-postgresql-downloads](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads)  
나는 64비트 윈도우 환경에 11.2 최신 버전을 다운로드했다.  

![1]({{ site.url }}/assets/2019-03-08-upload_shp_file_to_postgreSQL/1.PNG)

pgAdmin은 DB를 관리하고 쿼리를 할 수 있는 GUI환경이다. 같이 설치해 준다.  
Stack Builder는 관련된 확장 툴들을 편리하게 설치해 줄 수 있게 해준다.
체크하면 postgreSQL 설치가 끝난 뒤 바로 postGIS를 편하게 설치할 수 있으니 꼭 체크해 주자.  
포트넘버, su 비밀번호를 설정하고, Locale은 기본값(Default, System)으로 주고 설치 시작.  

![2]({{ site.url }}/assets/2019-03-08-upload_shp_file_to_postgreSQL/2.PNG)

Stack Builder를 체크하고 설치했으면, 설치가 끝난 뒤 Stack Builder를 실행하는 옵션이 있다.
체크하고 마치면 Stack Builder가 바로 실행되고, 설치한 서버를 선택하고 다음을 누르면 추가 설치할 수 있는 목록이 나온다. 이 중, 아예 Spatial Extensions가 카테고리로 분류되어 있는 데, 여기 있는 PostGIS 번들을 체크 후 설치하면 된다. (POST GIS + 공간정보 조작에 필수적인 패키지들)

​![3]({{ site.url }}/assets/2019-03-08-upload_shp_file_to_postgreSQL/3.PNG)

(....) 뭐가 문제인지 모르겠지만 경로를 셋팅하는 과정에서 자꾸 멈춘다.  
진행되면 쭉 설치해 주시고, 안되면 그냥 PostGIS를 따로 찾아서 설치해주자.  

[https://postgis.net/windows_downloads/](https://postgis.net/windows_downloads/)  
나는 64비트 윈도우에 postgreSQL11을 설치했으므로, 맞는 버전을 찾아 설치해준다.  
보아하니, postgreSQL11은 postGIS2.5 이후 버전에서만 지원하는 것 같다.  
2.5.1 번들을 설치했다.  
(링크 안에 릴리즈 버전 - OSGEO 다운로드 링크 - pg11폴더 - exe파일을 받아 설치하면 된다.)

![4]({{ site.url }}/assets/2019-03-08-upload_shp_file_to_postgreSQL/4.PNG)

여기까지 했으면 필요한 설치는 끝났다. 이제 DB를 만들어 보자.  
먼저, postgreSQL을 컨트롤하는 pgAdmin을 실행시킨다.  

![5]({{ site.url }}/assets/2019-03-08-upload_shp_file_to_postgreSQL/5.PNG)

postgreSQL 11을 클릭해 접속한다. 아까 설치할 때 설정했던 슈퍼유저 비밀번호를 입력하면 된다.  
이런저런 속성 정보와 기본 템플릿, 스키마들을 살펴볼 수 있다.

![6]({{ site.url }}/assets/2019-03-08-upload_shp_file_to_postgreSQL/6.PNG)

Database 우클릭 -> Create -> Database... 를 클릭해 새로운 DB를 생성한다.  
이름은 마음대로 주고, Definition 탭에서 위와 같이 설정을 해 주고 Save를 누르면 새 DB 생성 완료.  

이제 postGIS를 새 DB에 장착(?)시킬 시간이다.  
현재 상태는 그냥 일반적인 관계형DB 상태.  
postGIS를 장착하면 다음과 같은 쿼리가 가능하다.

```SQL
SELECT superhero.name
FROM city, superhero
WHERE ST_Contains(city.geom, superhero.geom)
AND city.name = 'Gotham';
/* https://postgis.net 의 샘플 소스 */
```

위의 쿼리는 city.name이 Gotham인 city에 포함되는 superhero의 superhero.name을 뽑는 SELECT문이다.  
즉, 공간적으로 city는 폴리곤일 것이고, superhero는 포인트일 것인 데, 폴리곤 안에 포함되는 포인트를 뽑아내는 것. postGIS는 이렇게 ST_Contains(A, B)와 같은 공간 연산을 가능하게 한다.

새로 만든 DB에 postGIS를 얹으려면, 다음과 같은 쿼리를 입력해 주어야 한다.  

```SQL
CREATE EXTENSION postgis;
```

쿼리 입력은 대상이 되는 DB에 마우스 오른쪽 - Query tool을 클릭하면 쿼리 창이 나온다.
여기에 쿼리 입력 후 실행(pgAdmin4 기준번개 모양, F5)하면 된다.

​![7]({{ site.url }}/assets/2019-03-08-upload_shp_file_to_postgreSQL/7.PNG)


쿼리 입력 후 성공적으로 수행되면, Extensions항목에 postgis가 들어간 것을 볼 수 있다.  
Schemas>public에서 다양한 부분에 지리(공간)자료를 다룰 수 있는 것들이 추가되었다.  
예를 들어 Types를 살펴 보면(기본적으로는 아무것도 없다.) 지오메트리 관련, 래스터 관련 타입들이.... Functions에서는 아까 보았던 st_contains를 비롯한 다양한 공간 연산 및 정의 함수들이 있는 것을 볼 수 있다. (1200여개 함수가 정의되어 있다.) 특히 주목해야 할 것은 Tables에서 spatial_ref_sys 테이블이다. 이는 좌표계가 정의되어 있는 테이블이다.  해당 테이블에 마우스 우클릭>View Data를 하거나, 쿼리에
```SQL
SELECT * FROM public.spatial_ref_sys
```
를 입력하면 그 내용을 모두 볼 수 있는 데, 여기서 srid 컬럼이 우리가 일반적으로 알고 있는 EPSG코드이다.

이제 공간 정보를 갖는 데이터를 올려서 조작할 준비가 다 되었다.

이제 부터는 일반적으로 사용하는 shp파일셋을 우리가 만든 DB에 올려보는 시간이다.

이를 위해서 'PostGIS 2.0 Shapefile and DBF Loader Exporter'라는 툴이 필요하다. 이는  postGIS를 설치할 때 번들로 설치되어 왔을 것이다. 못 찾겠으면 postGIS를 설치했던 경로(일반적으로 C:\Program Files\PostgreSQL\11\bin)의 postgisgui 내에 shp2pgsql-gui.exe를 실행하면 된다.

![8]({{ site.url }}/assets/2019-03-08-upload_shp_file_to_postgreSQL/8.PNG)

실행하면 먼저 DB서버와 연결 셋팅을 해야 한다. View connection details...를 눌러서 위와 같이 셋팅  
(postgres는 기본 계정이고, 5432포트도 기본 포트, Database란에 아까 만든 DB 이름을 넣어준다.)

​
![9]({{ site.url }}/assets/2019-03-08-upload_shp_file_to_postgreSQL/9.PNG)

Add File을 눌러 추가할 shp파일을 올려 준다. 이 때, 좌표계의 EPSG코드를 SRID에 넣어 준다. (더블 클릭으로 편집 가능) 나는 EPSG:5179 좌표계를 가진 일산 주엽역 부근 도로 shp파일을 올려주었다.  
혹시 한글 속성 정보가 포함된 파일에서 한글이 깨질 경우 options에서 인코딩을 CP949나 EUC-KR 등으로 바꿔주면 깨짐 없이 읽어올 수 있다.  

이후 import를 한다. 혹시 import failed가 뜨면 shp파일셋(dbf포함)이 모두 같은 폴더에 있는 지 확인하고, 그래도 안 되면 디렉토리를 다른 곳으로 옮겨서 시도해 본다.  

![10]({{ site.url }}/assets/2019-03-08-upload_shp_file_to_postgreSQL/10.PNG)

pgAdmin에서 잘 import되었나 확인해 보자. 바로 database>(DB이름)>Tables를 보면 추가가 안 되어 있는 데,  
상단 메뉴에서 Object>Refesh를 한 번 해 주면(새로고침) 같은 경로에 불러온 공간데이터가 잘 포함되어 있다.

​![11]({{ site.url }}/assets/2019-03-08-upload_shp_file_to_postgreSQL/11.PNG)

참고로 postGIS 테이블은 QGIS에서도 바로 불러다 쓸 수 있다. ~~(아래 테이블은 무시하자)~~
메뉴에서 postGIS 테이블 추가를 하여 호스트에 local>>host로 하여 나머지 항목들을 DB셋팅할 때 사용한 값으로 입력해 주고 연결을 하면 위와 같이 현재 DB에 있는 테이블을 바로 가져올 수 있다.

![12]({{ site.url }}/assets/2019-03-08-upload_shp_file_to_postgreSQL/12.PNG)

이 때, SRID를 제대로 설정했으면 불러와서 TMSforkorea를 이용해 웹 지도와 겹쳤을 때 오차 없이 지오메트리가 잘 들어맞을 것이다.  
다음 시간에는 postGIS의 기능을 이용한 간단한 쿼리 몇 가지를 해 보고, 이후에는 GeoServer를 통해 만들어진 공간 데이터를 웹에 서비스 하는 기초적인 작업을 해 보자.

​
osgeo한국어 지부에서 오픈소스GIS에 대한 많은 정보를 얻고 교류할 수 있다.
> 참조: [https://www.osgeo.kr/][https://www.osgeo.kr/]
