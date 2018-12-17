# Android messenger "Siru-Talk"
----------------------------------------------------------------------------------------------------
## 1. 시루톡
시루톡은 사용자가 보낸 메시지의 잘못된 맞춤법을 자동으로 인식 및 수정하여 전송한다. 이것으로 사용자의 올바른 맞춤법 사용을 돕고 사용자 간의 원활한 의사소통을 가능하게 한다. 맞춤법 틀리는 사람이 싫다는 의미로 (맞춤법 틀리는 사람) 시루시루 시루톡을 구현하게 되었다.

## 2. 앱 설치 방법 및 사용법
+ chat-sdk-android.apk를 안드로이드 기기에 다운로드한다.
+ 알 수 없는 개발자 허용 후 설치 버튼을 누른다.
+ 처음 앱 실행 시 가입을 진행한다. 혹은 구글 아이디로 로그인한다.
<img src="https://user-images.githubusercontent.com/43198950/50071527-6561fc80-0215-11e9-9c2c-97386352dd8a.jpg" width="40%">
</img>

+ 로그인 후, 상단 메뉴의 Chat Rooms를 누른다.

<img src="https://user-images.githubusercontent.com/43198950/50071536-6c890a80-0215-11e9-8812-ed4a567323da.jpg" width="40%"></img>

+ 기존 채팅방에 들어가거나, + 버튼을 통해 채팅방을 개설하여 채팅방에 입장하면 대화가 가능하다.

<img src="https://user-images.githubusercontent.com/43198950/50071545-76127280-0215-11e9-8fcc-6336622b84fd.jpg" width="40%"></img>

+ 상단 메뉴의 Profile을 누르면 프로필 수정이 가능하다.
<div>
<img src="https://user-images.githubusercontent.com/43198950/50071567-8f1b2380-0215-11e9-9beb-2a40cc31612d.jpg" width="30%"></img>
<img src="https://user-images.githubusercontent.com/43198950/50071572-92161400-0215-11e9-8e5e-3e7b557792b3.jpg" width="30%"></img>
<img src="https://user-images.githubusercontent.com/43198950/50071574-94786e00-0215-11e9-80e5-8d4e0d80e62b.jpg" width="30%"></img>
</div>

## 3. 주요 기능 및 관련 코드/API 설명
### 3-1. Retrofit2 - http 통신
+ Retrofit은 HTTP REST API를 자바 인터페이스로 제공하는데, 다른 통신 관련 라이브러리보다 안전하고 간편하여 사용했다.
+ 유사 라이브러리 Volley에 비하여 25 discussions 기준 약  4.2배, AsyncTask에 비하여 14배 빠른 속도를 가짐을 알 수 있다.
+ Retrofit2를 이용하기 위한 2단계 기초작업

          //먼저 Gradle Scripts>build.gradle(Module:chat-sdk-ui)
		implementation 'com.squareup.retrofit2:retrofit:2.5.0'

          //그 다음, 네트워크 사용을 위해 app단위 manifest파일에 아래 인터넷 사용 허가를 추가
		<uses-permission android:name="android.permission.INTERNET"/>

+ 기본으로 제공하는 5가지 요청 어노테이션 (@GET, @POST, @PUT, @DELETE, @HEAD) 중 시루톡은 @GET request를 통해 데이터를 읽어온다.

### 3-2. 맞춤법 교정 API - SpellCheckAPI
+ 제공되는 맞춤법 교정 API가 현재 없는것으로 판단되어, Retrofit을 통해 Naver와 통신하는 SIRU talk만의 맞춤법 교정 API를 직접 구현했다.
+ (retrofit에 대한 자세한 정보은 3-1에서 확인 가능)
+ SpellCheckAPI 인터페이스는 naver 맞춤법 검사 URL에 입력받은 문자열을 더해 교정을 요청한다.
+ base URL은 naver 모바일 버전의 기본 URL에 /를 더한 "https://m.search.naver.com/" 이며, 뒷부분은 전달받는 값에따라 달라진다.

        // query를 넣어주면 해당 문자열을 맞춤법에 맞게 교정하도록 네이버에 요청
        public interface SpellCheckAPI {
	  //5가지 어노테이션 중, 데이터를 받아오는 @GET request 사용
        @GET("p/csearch/ocontent/util/SpellerProxy?color_blindness=0") 
        Call Speller(@Query("q") String query);
      
        // retrofit 객체 (네이버 맞춤법 검사 URL에 맞게 생성)
        Retrofit retrofit = new Retrofit.Builder() .baseUrl("https://m.search.naver.com/") .build(); }
        
### 3-3. 맞춤법 검사 구현 - ChatActivity
#### A. 500자 초과 메세지 
+ 맞춤법을 교정하는 postCorrectSpellMessage는 onSendPressed 메소드에서 호출된다.
+ 매개변수로 사용자가 입력한 text를 넘겨받는다.

    public void onSendPressed(String text) {
        postCorrectSpellMessage(text);
    }
  
+ postCorrectSpellMessage 메소드는 SIRU talk의 맞춤법 교정기능을 구현하는 핵심 코드로, text를 올바르게 교정하여 전송하는 기능을 구현한다.
+ Naver 맞춤법 검사에 입력할 수 있는 글의 길이가 500이므로, SIRU talk에서 500자 이상의 글은 바로 전송한다.

    private void postCorrectSpellMessage(String text) {  
    if (text.length() > 500) {
     sendMessage(text, true);
     return; 
     }

#### B. 요청 준비작업
+ HTTP 요청 전송 준비 작업으로 SpellCheckAPI의 객체를 생성한다.
+ 사용자의 입력인 text를 Speller의 query로 넘겨 맞춤법 검사기 URL에 요청하는 과정이다. 

      SpellCheckAPI api = SpellCheckAPI.retrofit.create(SpellCheckAPI.class);
      Call http = api.Speller(text);

#### C. http 요청 성공 - OnResponse
+ HTTP 요청에 성공하면 onResponse 메소드가 수행된다.

      String result;
      try {
      //받은 Response Body를 문자열 형태로 변환 
      result = response.body().string(); 
      }catch (Exception e) {
      e.printStackTrace();
      onFailure(call, new Throwable(e.getMessage()));
      return;
      }

+ 통신의 결과로 받은 데이터 JSON(JavaScript Object Notation)파일을 사용 가능한 데이터로 파싱하고 변환된 메시지를 추출하는 과정이다.
+ JSON 파일의 데이터가 {}로 묶여있어 JSONObject 타입으로 받아온다.
+ 데이터 파싱을 통해 얻은 result를 sendMessage 메소드를 통해 전송하며 마무리한다.
      
      try {
      //{}로 묶인 object타입의 데이터를 받기 위해 JSONObject 타입으로 받아오기
      JSONObject jsonObject = new JSONObject(result);
      //message를 key로 받아오고 그 중 result 데이터를 받아옴
      jsonObject = jsonObject.getJSONObject("message").getJSONObject("result");
      //받아온 메시지를 최종적으로 전송
      sendMessage(Html.fromHtml(jsonObject.getString("notag_html")).toString(), true); 
      } catch (Exception e) {
      e.printStackTrace();
      onFailure(call, new Throwable(e.getMessage()));
      }
      
#### D. http 요청 실패 - onFailure
+ 맞춤법 검사기 요청이 실패하면 교정 과정을 거치지 않고 사용자 입력 메시지를 전송한다.
      
      public void onFailure(Call call, Throwable t) {
      sendMessage(text, true); }


### 3-4 주요 기능 스크린 샷 

<div>
<img src="https://user-images.githubusercontent.com/43198950/50071581-99d5b880-0215-11e9-9619-ec79669e67e7.jpg" width="40%"></img>
<img src="https://user-images.githubusercontent.com/43198950/50071586-9cd0a900-0215-11e9-9209-2598dc80fb6d.jpg" width="40%"></img>
</div>

## 4. 사용 오픈소스
+ Chat SDK for Android 오픈소스 https://github.com/chat-sdk/chat-sdk-android
+ 네이버 맞춤법 검사기 https://search.naver.com/search.naver?sm=top_sug.pre&fbm=1&acr=1&acq=%EB%84%A4%EC%9D%B4%EB%B2%84+%EB%A7%9E%EC%B6%94&qdt=0&ie=utf8&query=%EB%84%A4%EC%9D%B4%EB%B2%84+%EB%A7%9E%EC%B6%A4%EB%B2%95+%EA%B2%80%EC%82%AC%EA%B8%B0
+ retrofit2 안드로이드 오픈소스 https://github.com/square/retrofit

## 5. License
GPLv3 License https://github.com/cjh9727/SiruTalk/blob/master/LICENSE

## 6. 개발자 정보
+ 1515035 신채연(acc11311) : HTTP 요청 전송 작업, firebase, 중간 ppt 제작, 기말 발표자
+ 1771025 방수정(tnwjd7732) : 맞춤법 교정 API, UI, 중간 발표자, 동영상 편집
+ 1771054 최라윤(RayunC) : 맞춤법 교정 API, UI, 중간 발표자, 기말 ppt 제작, 기말 발표자
+ 1771109 최정화(cjh9727) : HTTP 요청 전송 작업, 앱 로고 디자인, UI, 기말 ppt 제작
