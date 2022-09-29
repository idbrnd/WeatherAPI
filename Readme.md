# Naver Geocoding
​
네이버 클라우드 로그인후,
​
아래 절차를 따라하면 Naver Geocoding을 위한 전반적인 세팅이 완료될 것입니다.
​
![](https://velog.velcdn.com/images/new_ego_doc/post/900c1205-5315-4488-a53e-cae8098ea946/image.png)
​
![](https://velog.velcdn.com/images/new_ego_doc/post/7e1e0588-51d7-4ac9-b660-192f9104a2b7/image.png)
​
![](https://velog.velcdn.com/images/new_ego_doc/post/47192a90-17c7-45e7-a53e-89b8e9dca614/image.png)
​
![](https://velog.velcdn.com/images/new_ego_doc/post/eab03014-2b67-4b4a-bd92-7dff588a9ad4/image.png)
​
![](https://velog.velcdn.com/images/new_ego_doc/post/36a83f94-3146-430e-ad37-fc955dc67ed7/image.png)
​
![](https://velog.velcdn.com/images/new_ego_doc/post/5714ec1a-3304-4fb6-a8a7-2286c8c1b14b/image.png)
해당 부분은 블러처리를 해놓았지만 개개인이 볼수있는 각각의 키가 발행됩니다. 
​
<br/>
<br/>
​
# Naver Geocoding API 사용하기
​
출처: https://api.ncloud-docs.com/docs/ai-naver-mapsgeocoding-geocode
​
### **`Request`**
​
​
```shell
https://naveropenapi.apigw.ntruss.com/map-geocode/v2/geocode?query={주소}
```
​
- `{주소}`부분은 반드시 encode 처리가 되어야 합니다.
​
- `헤더(요청 header)` 꼭 넣어주셔야합니다.
​
|헤더명|필수 여부|설명|
|---------|:----------:|---------|
|**X-NCP-APIGW-API-KEY-ID** |Y|	앱 등록 시 발급받은 <br/>`Client ID X-NCP-APIGW-API-KEY-ID:{Client ID}`|
|**X-NCP-APIGW-API-KEY**    |Y|	앱 등록 시 발급 받은 Client Secret <br/>`X-NCP-APIGW-API-KEY:{Client Secret}`|
|Accept                 |N|- 응답 포맷<br/>JSON(기본값), XML 지원 <br/>MIME 타입으로 원하는 포맷을 설정 <br/>`Accept: application/json`<br/>`Accept: application/xml`|
​
### **`Response`**
```json
{
    "status": "OK",
    "meta": {
        "totalCount": 1,
        "page": 1,
        "count": 1
    },
    "addresses": [
        {
            "roadAddress": "경기도 성남시 분당구 불정로 6 그린팩토리",
            "jibunAddress": "경기도 성남시 분당구 정자동 178-1 그린팩토리",
            "englishAddress": "6, Buljeong-ro, Bundang-gu, Seongnam-si, Gyeonggi-do, Republic of Korea",
            "addressElements": [
                {
                    "types": [
                        "POSTAL_CODE"
                    ],
                    "longName": "13561",
                    "shortName": "",
                    "code": ""
                }
            ],
            "x": "127.10522081658463",
            "y": "37.35951219616309",
            "distance": 20.925857741585514
        }
    ],
    "errorMessage": ""
}
```
​
# 그러나!!!! 실제 javascript에서 문제가 생깁니다
​
## 직면한 문제 : **`CORS`**
___
​
## **해결책 - 다른서버(node-red 사용)를 통해서 CORS 우회**
​
![](https://velog.velcdn.com/images/new_ego_doc/post/53ba22e3-cb76-4ad1-9bbf-98e3f42a463c/image.png)
​
```
[{"id":"226986d.7ec397a","type":"http request","z":"774a325f.f611dc","name":"","method":"GET","ret":"txt","paytoqs":false,"url":"","tls":"","persist":false,"proxy":"","authType":"","x":610,"y":860,"wires":[["b81a7f69.151de"]]},{"id":"b4cf4e71.48831","type":"function","z":"774a325f.f611dc","name":"","func":"msg.method = \"GET\";\n\nquery = encodeURIComponent(msg.payload.query)\n\nmsg.url = \"https://naveropenapi.apigw.ntruss.com/map-geocode/v2/geocode?query=\"+query;\n\nmsg.headers = {\n    \"X-NCP-APIGW-API-KEY-ID\":\"xxxxxxxxx\",\n    \"X-NCP-APIGW-API-KEY\":\"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx\"\n}\nreturn msg;","outputs":1,"noerr":0,"x":330,"y":860,"wires":[["226986d.7ec397a"]]},{"id":"b81a7f69.151de","type":"function","z":"774a325f.f611dc","name":"","func":"msg.payload = JSON.parse(msg.payload)\nreturn msg;","outputs":1,"noerr":0,"x":840,"y":860,"wires":[["48021ed3.39917"]]},{"id":"16065d38.3a7953","type":"http in","z":"774a325f.f611dc","name":"","url":"/idbGeo","method":"get","upload":false,"swaggerDoc":"","x":130,"y":860,"wires":[["b4cf4e71.48831"]]},{"id":"48021ed3.39917","type":"http response","z":"774a325f.f611dc","name":"","statusCode":"","headers":{},"x":1050,"y":860,"wires":[]}]
```
​
**`첫번째 function 노드`**
![](https://velog.velcdn.com/images/new_ego_doc/post/aacc5007-2abb-49c7-80ad-015a288419b8/image.png)
```javascript
msg.method = "GET";
// 우회한 API에서 얻은 query=OOOO 의 값(=msg.payload.query)을 그대로 넘겨받아 저장
query = encodeURIComponent(msg.payload.query)
//url 설정 및 쿼리스트링
msg.url = "https://naveropenapi.apigw.ntruss.com/map-geocode/v2/geocode?query="+query;
​
msg.headers = {//헤더 설정
    "X-NCP-APIGW-API-KEY-ID":"xxxxxx",
    "X-NCP-APIGW-API-KEY":"xxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
}
return msg;
```
<br/><br/>
​
# **`결과적으로 `**
​
```shell
http://localhost:1880/idbGeo?query={주소}
```
> 를 통해서 해당 주소의 lat과 lon의 정보를 얻어 올수 있다.
​

​
# openweather API

https://home.openweathermap.org/

![](https://velog.velcdn.com/images/new_ego_doc/post/e7363e40-8347-4228-b30c-938da0bb1edd/image.png)
​
![](https://velog.velcdn.com/images/new_ego_doc/post/90e67885-6d53-4daf-9e56-e1f07c2b3d3e/image.png)

로그인을 해주고

![](https://velog.velcdn.com/images/new_ego_doc/post/cf6766bd-df83-4059-9c9b-420e5d9669ca/image.png)

My API keys 를 누른 후에

![](https://velog.velcdn.com/images/new_ego_doc/post/568974c3-e099-443e-abb9-c19063d64ad6/image.png)

`API keys`탭에서 default로 만들어져 잇는 `Key`를 사용하면 된다

이렇게 얻은 Key 사용하여 앞서 얻은 위경도 정보와 함께 결과를 호출하면 된다 
​
<br/>
<br/><br/>
<br/><br/>
<br/>
​
# Finally
​
```javascript
query = encodeURIComponent("아현동");
url = "http://localhost:1880/idbGeo?query="+query;
​
fetch(url)
.then(response => response.json())
.then(parsedResponse => {
​
    const lon = parsedResponse.addresses[0].x;
    const lat = parsedResponse.addresses[0].y;
​
    getWeatherInfo(lon,lat);
​
})
.catch((error) => console.log(error));
​
/*openweather API 사용*/
function getWeatherInfo(lon, lat){

    //appid 부분에 openweather application key를 넣어주자!
    
    url = `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=**************`


    
    fetch(url)
    .then(response => response.json())
    .then(parsedResponse => {
​
        console.log(parsedResponse.main)
        //아현동(정확히는 아현동과 가장 가까운 관측소)의 온습도 정보 출력
​
    })
    .catch((error) => console.log(error));
}
```
​
결과 
​
![](https://velog.velcdn.com/images/new_ego_doc/post/33dc40ce-f1ac-479b-af7a-b9ad97d1d86b/image.png)