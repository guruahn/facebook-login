#페이스북 JavaScript SDK : 로그인 로직과 몇 가지 팁

##페이스북 로그인 로직

###1 초기화 코드
먼저 페이스북 초기화 소스를 facebook.js에 붙여넣은 후 facebook.js를 페이지 헤더에 위치시킵니다.  

```js
    //초기화 코드
    window.fbAsyncInit = function() {
        FB.init({
            appId      : '{app-id}',
            xfbml      : true,
            version    : 'v2.2'
        });
        //여기에 기타 초기화 스크립트 작성
    };

    (function(d, s, id) {
        var js, fjs = d.getElementsByTagName(s)[0];
        if (d.getElementById(id)) return;
        js = d.createElement(s); js.id = id;
        js.src = "//connect.facebook.net/ko_KR/sdk.js#xfbml=1&appId=1583286941899718&version=v2.2";
        fjs.parentNode.insertBefore(js, fjs);
    }(document, 'script', 'facebook-jssdk'));
```
###2 로그인 상태 체크

사용자의 상태는 3가지가 있습니다.

-앱의 권한을 부여한 상태(상태값 : connected)
-페이스북에 로그인했지만 앱에 권한을 부여하지 않은 상태(상태값 : not_authorized)
-두 모두 하지 않은 상태

그래서 상태값을 체크하는 함수는 아래와 같이 작성합니다.

```js
    function statusChangeCallback(response, callback) {
        // The response object is returned with a status field that lets the
        // app know the current login status of the person.
        // Full docs on the response object can be found in the documentation
        // for FB.getLoginStatus().
        if (response.status === 'connected') {
            // Logged into your app and Facebook.

        } else if (response.status === 'not_authorized') {
            alert("로그인해야 이용가능한 기능입니다.");
            // The person is logged into Facebook, but not your app.
            //document.getElementById('status').innerHTML = 'Please log ' + 'into this app.';
        } else {
            // The person is not logged into Facebook, so we're not sure if
            // they are logged into this app or not.
            //document.getElementById('status').innerHTML = 'Please log ' + 'into Facebook.';

            alert("로그인해야 이용가능한 기능입니다.");
        }
    }

    FB.getLoginStatus(function(response){
        statusChangeCallback(response);
    });
```

따라서 1 원하는 시점에 위 상태체크 함수를 삽입하면 원하는 시점에 상태값을 체크할 수 있습니다.

##캔버스 사이즈 세팅하기

페이스북 페이지 탭의 경우 탭 페이지 내에 앱에서 설정한 페이지가 삽입되는 영역을 캔버스라고 합니다. 캔버스 사이즈를 세팅하지 않아도되는 경우가 있지만 스크롤바가 생길 수 있기 때문에 설정해주는 것이 좋습니다. 아래와 같이 설정할 수 있습니다.

```js
    FB.Canvas.setSize({ width: 810, height: wrapperHeight});
```

참고로 FB.function 들은 페이스북 초기화 코드(FB.init)가 실행된 시점부터만 가능합니다.

###댓글과 캔버스 사이즈

댓글 플러그인을 달았다면 사용자가 댓글을 작성하면 댓글영역이 커지면서 스크롤이 생기게 됩니다. 이 경우 아래와 같이 작성하십시요.

```js
    FB.Event.subscribe("comment.create", function(response){
        var wrapperHeight = $('#wrapper').height()+ 80;
        FB.Canvas.setSize({ width: 810, height: wrapperHeight});
    });
```

FB.Event.subscribe 라는 메소드는 이벤트가 실행되었을때 실행할 콜백함수를 정의할 수 있도록 도와줍니다. subcribe가능한 이벤트 중 하나가 "comment.create"(댓글 작성)입니다. [FB.Event.subscribe 에 대해서 더 알아보기](https://developers.facebook.com/docs/reference/javascript/FB.Event.subscribe/v2.2?locale=ko_KR)

###초기 캔버스 사이즈 설정
원래는

```js
    FB.Canvas.setSize({ width: 810, height: 1200 });
```

위 코드 만으로 캔버스 사이즈를 설정할 수 있지만 댓글 플러그인을 달았을 경우 페이지가 로딩되더라도 댓글까지 로딩되려면 약간의 시간이 더 필요합니다. 즉 캔버스의 실제 높이가 변한다는 것입니다. 이 때문에 약간의 시간이 흐른 후에 캔버스의 높이값을 설정해주는 것이 좋습니다.

```js
    setTimeout(function(){
        var wrapperHeight = $('#wrapper').height()+ 30;
        FB.Canvas.setSize({ width: 810, height: wrapperHeight});
    },2000);
```

이렇게 하면 약 2초 후에 캔버스 사이즈를 설정하게됩니다. 물론 2초라는 시간은 각자 알아서 환경에 맞게 조정하시면 됩니다.

##로그인 팝업

로그인을 하지 못한 사용자를 위해 로그인 플러그인을 달 수도 있겠지만 직접 만든 디자인으로 로그인 버튼을 달고싶거나 내가 원하는 시점에 로그인 팝업이 나오도록 하고 싶다면 아래와 같이 작성하면 됩니다.

```js
    FB.login(function(){
        FB.getLoginStatus(function(){
            statusChangeCallback(response);
        });
    }, {scope: 'public_profile'});
```

추가로 얻고자 하는 권한은 위 코드처럼 json 형태로 {scope: '{권한이름}'}을 전달하세요. 'public_profile'은 기본권한이라 반드시 명시하지 않아도 되지만 되도록 명시하는 것이 더 좋습니다.

##예시

하나의 시나리오 예시로 샘플을 만들어보겠습니다. 시나리오는 사용자가 도착하거나 댓글을 달았을 경우 캔버스 사이즈를 다시 세팅하고 공유버튼을 누른 후 공유후 공유 여부를 체크하는 과정입니다.
```js
    //index.html
    <html>
        <head>
            ...
            <script src="//your-domain.com/js/facebook.js"></script>
        </head>
        <body>
            <button class="sharing">공유하기</button>
        </body>
    </html>

    //facebook.js
    //상태체크 함수 정의
    function statusChangeCallback(response, callback) {
        // response 객체는 상태 값을 리턴합니다.
        if (response.status === 'connected') {
            // 권한을 얻었을 경우
            if(typeof callback == 'function') callback();
        } else if (response.status === 'not_authorized') {
            // 권한이 없는 경우
            alert("권한 승인을 해야 이용가능한 기능입니다.");
        } else {
            // 페이스북 로그인을 하지 않은 경우
            alert("로그인해야 이용가능한 기능입니다.");
        }
    }

    //초기화 코드
    window.fbAsyncInit = function() {
        FB.init({
            appId      : '{app-id}',
            xfbml      : true,
            version    : 'v2.2'
        });
        //여기에 기타 초기화 스크립트 작성
        FB.Event.subscribe("comment.create", function(response){
            //댓글 달면 캔버스 높이 다시 세팅
            var wrapperHeight = $('#wrapper').height()+ 80;
            FB.Canvas.setSize({ width: 810, height: wrapperHeight});
        });
        //로딩 후 Canvas 높이값 재설정
        setTimeout(function(){
            if(!isMOBILE) $('.fb_iframe_widget').css({'padding':'0 15px'}).find('iframe').css('width', '780px')
            var wrapperHeight = $('#wrapper').height()+ 30;
            FB.Canvas.setSize({ width: 810, height: wrapperHeight});
        },2000);
    };

    (function(d, s, id) {
        var js, fjs = d.getElementsByTagName(s)[0];
        if (d.getElementById(id)) return;
        js = d.createElement(s); js.id = id;
        js.src = "//connect.facebook.net/ko_KR/sdk.js#xfbml=1&appId=1583286941899718&version=v2.2";
        fjs.parentNode.insertBefore(js, fjs);
    }(document, 'script', 'facebook-jssdk'));

    $(document).ready(initPage);
    function initPage(){
        $('.share_event').click(function(){
            var href = "https://your-domain.com/index.html";
            FB.login(function(response) {
                //set user info hear
                FB.getLoginStatus(function(response) {
                    statusChangeCallback(response, sharePopup(href));
                });
            }, {scope: 'public_profile'});
            return false;
        });
    }

    function sharePopup(href){
        FB.ui({
            method: 'share',
            href: href
        },function(response){
            if(response.error_code){
                //공유 실패시
                console.log(response);
            }else{
                //공유 성공시
                update_user_status();//함수 상세 생략
            }
            //console.log(response);
        });
    }
```
이 예제를 이해하면 페이스북 JavaScript SDK 에 대한 절반의 이해는 된 것입니다. [JavaScript SDK 공식문서](https://developers.facebook.com/docs/javascript?locale=ko_KR)를 보면 더 많은 것을 알 수 있을 것입니다.^^
