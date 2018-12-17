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
### 3-1. 맞춤법 교정 API - SpellCheckAPI
맞춤법 교정 API (retrofit 사용) 
     
      public interface SpellCheckAPI {
        // query를 넣어주면 해당 문자열을 맞춤법에 맞게 교정하도록 네이버에 요청 
        @GET("p/csearch/ocontent/util/SpellerProxy?color_blindness=0")  Call Speller(@Query("q") String query);
        // retrofit 객체 (네이버 맞춤법 검사 URL에 맞게 생성)
        Retrofit retrofit = new Retrofit.Builder() .baseUrl("https://m.search.naver.com/") .build(); }
        
### 3-2. 맞춤법 검사 구현 - ChatActivity
#### A. 500자 초과 메세지 
onSendPressed 메소드에서 postCorrectSpellMessage (맞춤법 검사) 메서드 호출

    public void onSendPressed(String text) {
        postCorrectSpellMessage(text);
    }
  
retrofit 객체로 네이버 맞춤법검사기에 접근해 text를 올바르게 설정하여 send하는 메소드
500자 초과의 메세지는 그대로 전송 (네이버 맞춤법 검사기가 500자 까지 지원)

    private void postCorrectSpellMessage(String text) {  
    if (text.length() > 500) {
     sendMessage(text, true);
     return; 
     }

#### B. 요청 준비작업
HTTP 요청 전송 준비 작업으로 SpellCheckAPI의 객체 생성

      SpellCheckAPI api = SpellCheckAPI.retrofit.create(SpellCheckAPI.class);
      Call http = api.Speller(text);

#### C. http 요청 성공 - OnResponse
받은 Response Body를 문자열 형태로 변환 
     
      String result;
      try {
      result = response.body().string(); 
      }catch (Exception e) {
      e.printStackTrace();
      onFailure(call, new Throwable(e.getMessage()));
      return;
      }

JSON 데이터 파싱으로 변환된 메시지를 추출
      
      try {
      JSONObject jsonObject = new JSONObject(result);
      jsonObject = jsonObject.getJSONObject("message").getJSONObject("result");
      sendMessage(Html.fromHtml(jsonObject.getString("notag_html")).toString(), true); 
      // 메시지 전송 
      } catch (Exception e) {
      e.printStackTrace();
      onFailure(call, new Throwable(e.getMessage()));
      }
      
#### D. http 요청 실패 - onFailure
맞춤법 검사기 요청이 실패했으면 원래 메시지를 그대로 전송 
      
      public void onFailure(Call call, Throwable t) { sendMessage(text, true); }


### 3-3 주요 기능 스크린 샷 

<div>
<img src="https://user-images.githubusercontent.com/43198950/50071581-99d5b880-0215-11e9-9619-ec79669e67e7.jpg" width="40%"></img>
<img src="https://user-images.githubusercontent.com/43198950/50071586-9cd0a900-0215-11e9-9209-2598dc80fb6d.jpg" width="40%"></img>
</div>

## 4. 사용 오픈소스
+ Chat SDK for Android 오픈소스 https://github.com/chat-sdk/chat-sdk-android
+ 네이버 맞춤법 검사기 https://search.naver.com/search.naver?sm=top_sug.pre&fbm=1&acr=1&acq=%EB%84%A4%EC%9D%B4%EB%B2%84+%EB%A7%9E%EC%B6%94&qdt=0&ie=utf8&query=%EB%84%A4%EC%9D%B4%EB%B2%84+%EB%A7%9E%EC%B6%A4%EB%B2%95+%EA%B2%80%EC%82%AC%EA%B8%B0

## 5. License
GPLv3 License https://github.com/cjh9727/SiruTalk/blob/master/LICENSE

## 6. 개발자 정보
+ 1515035 신채연(acc11311) : HTTP 요청 전송 작업, firebase, 중간 ppt 제작, 기말 발표자
+ 1771025 방수정(tnwjd7732) : 맞춤법 교정 API, UI, 중간 발표자
+ 1771054 최라윤(RayunC) : 맞춤법 교정 API, UI, 중간 발표자, 기말 ppt 제작, 기말 발표자
+ 1771109 최정화(cjh9727) : HTTP 요청 전송 작업, 앱 로고 디자인, UI, 기말 ppt 제작
