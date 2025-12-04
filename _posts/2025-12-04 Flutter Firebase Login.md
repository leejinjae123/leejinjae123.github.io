---
title: "Flutter Firebase 로그인"
date: 2025-12-03
last_modified_at: 2025-12-03
categories:
  - Flutter
tags:
  - Flutter
  - Firebase
  - Firestore
  - 소셜 로그인
sidebar:
  nav: "sidebar-category"
---

# Flutter Firebase Login 정리

## 0. Firebase Auth 세팅

- 세팅
    
    <aside>
    🛠 flutter pub add firebase_auth 로 dependency 등록
    
    </aside>
    
    - Firebase로 프로젝트 제작
    - Android 등록
        1. 앱 등록
            - 패키지 이름 등록 - 프로젝트 내 android/app/src/main 안의 AndroidManifest.xml 파일의       상단 package 이름을 가져와야 한다.
            - 앱 닉네임 - 자유
        2. google-services.json을 프로젝트 내 android/app/src안에 이동.
            - 디버그 서명 인증서
                - 디버그 서명 인증서란? : 컴퓨터의 고유 키, 컴퓨터마다 고유한 값을 지니고있음.
                - 가져오는 방법
                    
                    ```
                    - Mac / Linux
                    
                    keytool -list -v \
                    -alias <your-key-name>(보통 androiddebugkey) -keystore <path-to-production-keystore>
                    ✨ 초기 비밀번호는 android
                    				
                    - Windows
                    
                    keytool -list -v \
                    -alias androiddebugkey -keystore %USERPROFILE%(컴퓨터계정명)\.android\debug.keystore
                    ✨ 초기 비밀번호는 android	
                    ```
                    
                - Android Studio에서는 Gradle / Tasks / android / signingReport에서 확인 가능.
                
        3. firebase-sdk 설치 - firebase cli(firebase를 편하게 사용할 수 있게 해주는 툴) 사용. ← 필수는 아님.
    - Apple 등록
        1. 앱 등록
            - Apple 번들 ID 등록
                - Mac 이라면 Xcode에서 Runner - General에서 확인 가능, Window라면 package이름
            - 앱 닉네임 - 자유
            - App Store ID - 자유
        2. GoogleService-info.plist를 Runner 안에 등록
            - firebase 사용을 위한 설정
    - firebase 내에서 auth 시작
    - main.dart 파일에 함수 추가.
        
        ```jsx
        @override
          Widget build(BuildContext context) {
            return MultiProvider(
              providers: [
                ChangeNotifierProvider<FirebaseProvider>(
                    builder: (_) => FirebaseProvider())
              ], 
        ```
        
        그러나 이번 프로젝트는 GetIt dependency를 이용하기 때문에, GetIt에 다음 함수를 등록하면 된다.
        
        ```jsx
        Future<void> init() async {
        	await Firebase.initializeApp();
        }
        ```
        

## 1.  이메일/비밀번호 로그인

- 확인
    - 로그인 기능에서 이메일/비밀번호 사용 설정
    - 탬플릿에서 보낼 이메일 형식 작성
    - 회원 가입 코드
    
    ```jsx
    void signUp({
        required String email, // 이메일
        required String pwd, // 비밀번호
        required Function() onSuccess, // 가입 성공시 호출되는 함수
        required Function(String err) onError, // 에러 발생시 호출되는 함수
      }) async {
        // 회원가입
        if (email.isEmpty) {
          onError("이메일을 입력해 주세요.");
          return;
        } else if (pwd.isEmpty) {
          onError("비밀번호를 입력해 주세요.");
          return;
        }
        // firebase auth 회원 가입
        try {
          await FirebaseAuth.instance.createUserWithEmailAndPassword(
            email: email,
            password: pwd,
          );
          // 성공 함수 호출
          onSuccess();
        } on FirebaseAuthException catch (e) {
          // Firebase auth 에러 발생
          onError(e.message!);
        } catch (e) {
          // Firebase auth 이외의 에러 발생
          onError(e.toString());
        }
     }
    ```
    
    - 로그인 함수
    
    ```jsx
    void signIn({
        required String email, // 이메일
        required String pwd, // 비밀번호
        required Function() onSuccess, // 로그인 성공시 호출되는 함수
        required Function(String err) onError, // 에러 발생시 호출되는 함수
      }) async {
        // 로그인
        if (email.isEmpty) {
          onError('이메일을 입력해주세요.');
          return;
        } else if (pwd.isEmpty) {
          onError('비밀번호를 입력해주세요.');
          return;
        }
    
        // 로그인 시도
        try {
          await FirebaseAuth.instance.signInWithEmailAndPassword(
            email: email,
            password: pwd,
          );
    
          onSuccess(); // 성공 함수 호출
          notifyListeners(); // 로그인 상태 변경 알림
        } on FirebaseAuthException catch (e) {
          // firebase auth 에러 발생
          onError(e.message!);
        } catch (e) {
          // Firebase auth 이외의 에러 발생
          onError(e.toString());
        }
      }
    ```
    
    - 로그아웃 함수
    
    ```jsx
    void signOut() async {
        // 로그아웃
        FirebaseAuth.instance.signOut();
      }
    ```
    

## 2.  Google 로그인

- 확인
    - 로그인 기능에서 Google 추가
    - 세팅 시 확인한 SHA-1 인증서 등록
    - Xcode에서 커스텀 URL Schemes에  GoogleService-info.plist 파일의 REVERSED_CLIENT_ID값을 등록해 줘야한다.
    - Runner 프로젝트 → TARGETS RUNNER → Info탭 / URL Types 선택 후 버튼으로 URL 스키마 추가
        
        ```jsx
        🖐️ Xcode가 없다면 Info.plist에서 직접 REVERSED_CLIENT_ID값을 등록 해 줘야한다.
        ```
        
    - 구글 로그인 코드
    
    ```jsx
    Future<UserCredential> googleAuthSignIn() async {
    		//구글 Sign in 플로우 오픈!
        final GoogleSignInAccount? googleUser = await GoogleSignIn().signIn();
    		//구글 인증 정보
        final GoogleSignInAuthentication? googleAuth =
            await googleUser?.authentication;
    		//읽어온 인증정보로 파이어베이스 인증 로그인
        final credential = GoogleAuthProvider.credential(
            accessToken: googleAuth?.accessToken, idToken: googleAuth?.idToken);
    		//파이어 베이스 Signin하고 결과(userCredential) 리턴
        return await FirebaseAuth.instance.signInWithCredential(credential);
      }
    ```
    

## 3. FaceBook 로그인

- 확인
    - 페이스북 개발자 페이지에서 앱 추가
        - Android 추가
            - 패키지 이름 추가 : 프로젝트 패키지 이름
            - 액티비티 이름 추가 : 프로젝트 패키지 이름 + .MainActivity
            - 해시 키 추가 : 디버그 키
            - 프로젝트 안의 android/app/src/main/res/values/strings.xml 에 코드 추가.
                
                만약 없다면, Strings.xml을 만들어서 넣어줘야 한다.
                
            
            ```jsx
            <?xml version="1.0" encoding="utf-8"?>
            <resources>
                <string name="app_name">페이스북 앱 이름</string>
                <string name="facebook_app_id">페이스북 앱 아이디</string>
                <string name="fb_login_protocol_scheme">페이스북 로그인 프로토콜 스키마</string>
                <string name="facebook_client_token">페이스북 앱 시크릿 토큰</string>
            </resources>
            
            - 앱 확인 : 페이스북 앱 설정->기본 설정 창에 정보를 확인할 수 있다.
            ```
            
            - AndroidManifest.xml에 코드 추가.
            
            ```jsx
            <application>
            
                ...
                
                <meta-data
                    android:name="com.facebook.sdk.ApplicationId"
                    android:value="@string/facebook_app_id"/>
            
                <activity
                    android:name="com.facebook.FacebookActivity"
                    android:configChanges=
                            "keyboard|keyboardHidden|screenLayout|screenSize|orientation"
                    android:label="@string/app_name" />
            
                <activity
                    android:name="com.facebook.CustomTabActivity"
                    android:exported="true">
                    <intent-filter>
                        <action android:name="android.intent.action.VIEW" />
                        <category android:name="android.intent.category.DEFAULT" />
                        <category android:name="android.intent.category.BROWSABLE" />
                        <data android:scheme="@string/fb_login_protocol_scheme" />
                    </intent-filter>
                </activity>
            </application>
            ```
            
        - IOS 프로젝트 등록
            - 번들 ID 등록
            - info.plist에 추가
            
            ```jsx
            <key>CFBundleURLTypes</key>
            <array>
                <dict>
                    <key>CFBundleURLSchemes</key>
                    <array>
                        <string>your fbFacebookAppID</string>
                    </array>
                </dict>
            </array>
            <key>FacebookAppID</key>
            <string>your FacebookAppID</string>
            <key>FacebookDisplayName</key>
            <string>your FacebookAppName</string>
            <key>LSApplicationQueriesSchemes</key>
            <array>
                <string>fbapi</string>
                <string>fb-messenger-share-api</string>
                <string>fbauth2</string>
                <string>fbshareextension</string>
            </array>
            
            📕 들여쓰기에 매우 주의해야 하며, CFBundleURLSchemes 이나 중복된 key값은 두번 쓰일수 없다.
            ```
            
    - 페이스북 로그인 함수
    
    ```jsx
    Future<Object> signInWithFacebook() async {
        // 로그인 결과
        final LoginResult loginResult = await FacebookAuth.instance.login();
    
        // accessToken을 받아 credential 받아오기 
        if (loginResult.status == LoginStatus.success) {
          final AccessToken accessToken = loginResult.accessToken!;
          final OAuthCredential credential =
              FacebookAuthProvider.credential(accessToken.token);
    		// firebase 로그인
          return await FirebaseAuth.instance.signInWithCredential(credential);
        } else {
          return false;
        }
      }
    ```
    

## 4. Apple 로그인

- 확인
    
    ```jsx
    📕 Apple Developer 아이디가 필수이며, 1년에 $100이 청구됨.
    ```
    
    - AppleDeveloper - **Certificates, Identifiers & Profiles 페이지로 이동**
    - Account -> resources -> identifiers 로 이동하면 생성한 앱이 표시
        - Sign in with Apple을 클릭해 추가해 주고, Bundle Identifier을 추가.
        1. ID detail에서 Sign in with Apple의 Edit 버튼 클릭
        2. Server to Server Notification Endpoint에서 Firebase apple 로그인 기능을 활성화 할 떄  Firebase Auth Apple Signin method 추가하는 부분에서 찾을 수 있으니 복사해서 붙여넣자.
            
            ![스크린샷 2023-04-24 오후 7.57.06.png](/assets/png/flutter_login1.png)
            
        3. 패키지 설치
        
        ```jsx
        flutter pub add sign_in_with_apple
        ```
        
        1. Apple Sign in 함수
        
        ```jsx
        Future<UserCredential> signInWithApple() async {
          	//애플 크리덴셜 -> 로그인
            final appleCredential = await SignInWithApple.getAppleIDCredential(
              scopes: [AppleIDAuthorizationScopes.email],//이메일 들여다 볼거임!
            );
            //애플 크리덴셜 Oauth 크리덴셜로 변경
            final oauthCredential = OAuthProvider("apple.com").credential(
              idToken: appleCredential.identityToken,
              accessToken: appleCredential.authorizationCode,
            );
            //firebase에 Signin
            return await FirebaseAuth.instance.signInWithCredential(oauthCredential);
          }
        ```
        
        ### 🧨 Android 일 시 apple Sign in 함수
        
        - 위 내용의 코드로 로그인 함수를 구현하고, emulator로 구현 해 보았다. IPhone에서는 잘 구현되는 반면, Android에서는 오류를 뱉는다.
        - Android에서는 따로 Redirect Url API를 구현해 줘야한다는 글을 보고, 코드를 작성
        - [https://pub.dev/packages/sign_in_with_apple](https://pub.dev/packages/sign_in_with_apple) 을 이용하여 API 작성.
        
        ```jsx
        📕 간단하게 Glitch.me를 이용하여 서버리스 API를 제작한 것으로, 보안상의 문제가 심각하므로
        	 실전 코드에서는 따로 만든 서버를 이용하여 API를 구현해야 한다.
        ```
        
        - API의 key 등록을 위해 AppleDeveloper의 **Certificates, Identifiers & Profiles 안의, keys에서 새 키 등록.**
        - keys의 + 버튼을 클릭하여, key의 이름을 등록하고, Sign in with Apple의 Configure 클릭.
        - Description은 자유, Identifire은 Bundle아이디를 역순으로 기입. ex)AppName.example.com
        
        ```jsx
        - Apple 설정
        
        Xcode에서 Runner - Signing & Capabilities에서 + 버튼 클릭
        -> Sign in with Apple 찾아서 더블클릭
        
        - Android 설정
        
        android/app/src/main/Android Manifest.xml 수정
        
        <manifest xmlns:android="http://schemas.android.com/apk/res/android"
            package="{app ID}">
           <application
                android:label="{ID}"
                android:name="${applicationName}"
                android:icon="@mipmap/ic_launcher">
                .
                .
                .
                <!-- 추가 --!>
                <activity
                  android:name="com.aboutyou.dart_packages.sign_in_with_apple.SignInWithAppleCallback"
                  android:exported="true">
                     <intent-filter>
                        <action android:name="android.intent.action.VIEW" />
                        <category android:name="android.intent.category.DEFAULT" />
                        <category android:name="android.intent.category.BROWSABLE" />
        
                        <data android:scheme="signinwithapple" />
                        <data android:path="callback" />
                     </intent-filter>
                </activity>
        				<!-- 추가 --!>
                .
                .
                .
            </application>
        </manifest>
        ```
        
        - 정보에 필요한 TEAM_ID, SERVICE_ID, BUNDLE_ID, KEY_ID, KEY_CONTENTS, ANDROID_PACKAGE_IDENTI 6가지 값을 클래스로 등록한다.
            - Team ID - 아까 생성한 AppID 에서 AppID Prefix 입니다.
            - Bundle ID - ios 프로젝트 bundle ID 값 입니다.
            - Identifiers 에서 우측에 Services IDs 를 눌러주면 나오는 IDENTIFIER 값 입니다.
            - Key 상세에서 나오는 Key ID 값입니다.
            - KEY_CONTENTS는 key를 다운로드 하여 나오는 3줄의 키값을 한줄로 만들어 등록.
            - ANDROID_PACKAGE_IDENTI 는 안드로이드 패키지명을 넣어주면 된다.
        - **Service ID Redirect Url을 설정해줘야한다.**
            - 사이드바에서 Identifiers 를 누르고 우측에 Service IDs 로 설정 후 아까 생성한 Service ID 를 누르고, Configure을 눌러 Apple ID를 선택하고, URL을 등록
            - **Domains and Subdomains에는 도메인, URLs에는 콜백주소를 등록하면 된다.**
        - 콜백 함수
        
        ```jsx
        const express = require("express");
        const AppleAuth = require("apple-auth");
        const jwt = require("jsonwebtoken");
        const bodyParser = require("body-parser");
        
        const app = express();
        
        app.use(bodyParser.urlencoded({ extended: false }));
        
        // make all the files in 'public' available
        // https://expressjs.com/en/starter/static-files.html
        app.use(express.static("public"));
        
        // The callback route used for Android, which will send the callback parameters from Apple into the Android app.
        // This is done using a deeplink, which will cause the Chrome Custom Tab to be dismissed and providing the parameters from Apple back to the app.
        app.post("/callbacks/sign_in_with_apple", (request, response) => {
          const redirect = `intent://callback?${new URLSearchParams(
            request.body
          ).toString()}#Intent;package=${
            process.env.ANDROID_PACKAGE_IDENTIFIER
          };scheme=signinwithapple;end`;
        
          console.log(`Redirecting to ${redirect}`);
        
          response.redirect(307, redirect);
        });
        app.post("/sign_in_with_apple", async (request, response) => {
          const auth = new AppleAuth(
            {
              // use the bundle ID as client ID for native apps, else use the service ID for web-auth flows
              // https://forums.developer.apple.com/thread/118135
              client_id:
                request.query.useBundleId === "true"
                  ? process.env.BUNDLE_ID
                  : process.env.SERVICE_ID,
              team_id: process.env.TEAM_ID,
              redirect_uri:
                "https://flutter-sign-in-with-apple-example.glitch.me/callbacks/sign_in_with_apple", // does not matter here, as this is already the callback that verifies the token after the redirection
              key_id: process.env.KEY_ID
            },
            process.env.KEY_CONTENTS.replace(/\|/g, "\n"),
            "text"
          );
        
          console.log(request.query);
        
          const accessToken = await auth.accessToken(request.query.code);
        
          const idToken = jwt.decode(accessToken.id_token);
        
          const userID = idToken.sub;
        
          console.log(idToken);
        
          // `userEmail` and `userName` will only be provided for the initial authorization with your app
          const userEmail = idToken.email;
          const userName = `${request.query.firstName} ${request.query.lastName}`;
        
          // 👷🏻‍♀️ TODO: Use the values provided create a new session for the user in your system
          const sessionID = `NEW SESSION ID for ${userID} / ${userEmail} / ${userName}`;
        
          console.log(`sessionID = ${sessionID}`);
        
          response.json({ sessionId: sessionID });
        });
        
        // listen for requests :)
        const listener = app.listen(process.env.PORT, () => {
          console.log("Your app is listening on port " + listener.address().port);
        });
        ```
        
        - 새로운 함수
        
        ```jsx
        Future<UserCredential> signInWithApple() async {
            final AuthorizationCredentialAppleID credential =
                await SignInWithApple.getAppleIDCredential(
              scopes: [
                AppleIDAuthorizationScopes.email,
                AppleIDAuthorizationScopes.fullName,
              ],
              webAuthenticationOptions: WebAuthenticationOptions(
                clientId: "irept3.jsoftware.com",
                //보안상의 문제로 glitch.me로 되어있는 uri는 다 교체해야함.
                redirectUri: Uri.parse(
                  "https://scientific-glen-nape.glitch.me/callbacks/sign_in_with_apple",
                ),
              ),
            );
        
            final appleCredential = OAuthProvider('apple.com').credential(
              idToken: credential.identityToken,
              accessToken: credential.authorizationCode,
            );
        
            return await FirebaseAuth.instance.signInWithCredential(appleCredential);
          }
        ```
        

## 5. 카카오 로그인

- 확인
    - 카카오 개발자 페이지에서 어플리케이션 등록
        - 플랫폼에서 Android 등록
            - 패키지 : 패키지명
            - 마켓 URL : 마켓 주소
            - 키 해시 : 디버그 해시 키 등록
        - 플랫폼에서 IOS 등록
            - 번들 ID 등록
    - main.dart에서 초기화 키 등록
    
    ```jsx
    KakaoSdk.init(nativeAppKey: '개발자 NativeApp키');
    ```
    
    - 카카오 로그인은 따로 기능창에 없기 때문에 Firebase admin을 이용하여 구현해야 한다.
    - API 구현을 위해 Firebase Admin SDK 의 서버 코드와 serviceAccountKey.json을 다운받아
        
        서버에 적용시킨다.
        
    - function 기능 사용을 위해 Flutter 프로젝트에서 터미널에 firebase init를 입력하여 firebase_cli를
        
        설치하고, Function검색에서 node서버를 설치한다.
        
        ```jsx
        💡 간단한 테스트를 위해 firebase에서 제공하는 function을 통해 api를 제작하였다.
        
        - function은 서버리스 api로 사용량이 많지 않을떄 간단하게 제작하여 구현 할 수 있고,
        	테스트용으로는 좋으나 사용량이 많으면 요금이 많이 나올수 있으므로 따로 로컬 서버를 만
        	들거나 aws를 사용하는게 좋아보인다.
        ```
        
        ```jsx
        📕 Firebase Admin SDK
         
        Firebase 자체에서 지원하지 않는 소셜 로그인 기능을 구현하고 싶을때, 규격에 맞는 customToken을
        API로 발급받아 AdminSDK를 통해 firebase auth에 등록한다.
        
        ✨ 노드 서버에 등록할 코드
        
        var admin = require("firebase-admin");
        
        var serviceAccount = require("path/to/serviceAccountKey.json");
        
        admin.initializeApp({
          credential: admin.credential.cert(serviceAccount),
          databaseURL: "https://irept3-default-rtdb.firebaseio.com"
        });
        ```
        
    - 정상적으로 설치가 되었다면 Flutter 프로젝트 안에 functions 폴더가 생성된다.
    - admin-sdk에서 제공된 key를 function 폴더안에 넣고, index.js에 코드를 작성
        
        ```jsx
        const functions = require("firebase-functions");
        const admin = require("firebase-admin");
        // admin-sdk의 서비스키
        const serviceAccount = require("{admin-serviceKey}");
        
        admin.initializeApp({
          credential: admin.credential.cert(serviceAccount),
        });
        
        exports.createCustomToken = functions.https.onRequest(async (request, response) => {
        	// body에 넘어온 user정보 받기
          const user = request.body;
        	// admin에서 업체 구별기능이 없어 id 고유키에 kakao:를 붙여 구별
          const uid = `kakao:${user.uid}`;
          const updateParams = {
            email: user.email,
            displayName: user.displayName,
          };
          try {
        		// 정보가 있으면 update함
            await admin.auth().updateUser(uid, updateParams);
          } catch (e) {
        		// 정보가 없을시 create함
            updateParams["uid"] = uid;
            await admin.auth().createUser(updateParams);
          }
        
        	// customToken 제작
          const token = await admin.auth().createCustomToken(uid);
        	// token을 보냄
          response.send(token);
        });
        ```
        
    - kakaoLogin을 구현하고, user정보를 받아 보내서 admin 로그인
    
    ```jsx
    // api 주소
    final String url = 'https://us-central1-irept3.cloudfunctions.net/createCustomToken';
    
    // user 정보를 uri로 전송하여 customToken으로 가공
    Future<String> createCustomToken(Map<String, dynamic> user) async {
        final customToken = await http.post(Uri.parse(url), body: user);
    
        return customToken.body;
      }
    
    Future kakaoLogin() async {
    		// 카카오톡이 깔려있는지 검사
        bool isInstalled = await kakao.isKakaoTalkInstalled();
        if (isInstalled){
          try{
    				// 카카오톡 로그인
            await kakao.UserApi.instance.loginWithKakaoTalk();
    				// user 정보 받아오기
            user = await kakao.UserApi.instance.me();
    				// 받아온 user정보로 customToken 제작
            final customKakaoToken = await createCustomToken({
              'uid' : user!.id.toString(),
              'displayName' : user!.kakaoAccount!.profile!.nickname,
              'email' : user!.kakaoAccount!.email
            });
    				// customToken으로 admin 로그인
            await FirebaseAuth.instance.signInWithCustomToken(customKakaoToken);
          } catch(e){
            if (e is PlatformException && e.code == 'CANCELED') {
              return;
            }
            // 카카오톡에 연결된 카카오계정이 없는 경우, 카카오계정으로 로그인
            try {
    					// 카카오계정 로그인
              await kakao.UserApi.instance.loginWithKakaoAccount();
    					// user 정보 받아오기
              user = await kakao.UserApi.instance.me();
    					// 받아온 user정보로 customToken 제작
              final customKakaoToken = await createCustomToken({
                'uid' : user!.id.toString(),
                'displayName' : user!.kakaoAccount!.profile!.nickname,
                'email' : user!.kakaoAccount!.email
              });
    					// customToken으로 admin 로그인
              await FirebaseAuth.instance.signInWithCustomToken(customKakaoToken);
    
            } catch (error) {
    
            }
    
          }
        }
        else {
          try{
    				// 카카오 계정으로 로그인
            await kakao.UserApi.instance.loginWithKakaoAccount();
    				// user 정보 받아오기
            user = await kakao.UserApi.instance.me();
    				// user 정보로 customToken 제작
            final customKakaoToken = await createCustomToken({
              'uid' : user!.id.toString(),
              'displayName' : user!.kakaoAccount!.profile!.nickname,
              'email' : user!.kakaoAccount!.email
            });
    				// 발급받은 customToken으로 admin 로그인
            await FirebaseAuth.instance.signInWithCustomToken(customKakaoToken);
          }catch (e) {
            print('카카오 계정으로 로그인 실패 $e');
          }
        }
      }
    ```
    

## 6. 네이버 로그인

- 확인
    - 카카오와 같이 API로 커스텀 토큰을 만들어 주는 형식
    - 네이버 개발자 등록
        - 네이버 개발자 페이지에서 앱 등록 → 앱 이름과 네이버 로그인 API 선택.
        - 앱 API 설정에서 제공 정보 선택
        - 서비스 환경에서 flutter 사용시 꼭 안드로이드, ios 둘다 설정 필요.
        - 안드로이드 앱 패키지 → 프로젝트 패키지 이름
            
            IOS URL Scheme → 번들 ID 
            
        - URL은 자유.
    - Android 설정
    
    ```jsx
    - android/app/src/main/AndroidMenifest.xml
    
    <meta-data
      			android:name="com.naver.sdk.clientId"
                android:value="@string/client_id" />
     <meta-data
                android:name="com.naver.sdk.clientSecret"
                android:value="@string/client_secret" />
     <meta-data
                android:name="com.naver.sdk.clientName"
                android:value="@string/client_name" />
    
    - android/app/src/main/res/values/strings.xml
    
    <resources>
        <string name="client_id">{Client ID}</string>
        <string name="client_secret">{Client Secret}</string>
        <string name="client_name">{your_app_name}</string>
    </resources>
    
    📕 클라이언트 ID나 secret 같은 경우 네이버 개발자 어플리케이션을 선택시 개요에 있음.
    ```
    
    - IOS 설정
    
    ```jsx
    - ios/Runner/info.plist
    
    <key>naverConsumerKey</key>
    <string>{Client ID}</string>
    <key>naverConsumerSecret</key>
    <string>{Client Secret}</string>
    <key>naverServiceAppName</key>
    <string>{your_app_name}</string>
    <key>naverServiceAppUrlScheme</key>
    <string>naver</string>
    
    🖐️ 딥 링크 사용시 추가
    <key>CFBundleURLTypes</key>
    	<array>
        	<dict>
    			<key>CFBundleTypeRole</key>
    			<string>Editor</string>
    			<key>CFBundleURLName</key>
    			<string>naver</string>
    			<key>CFBundleURLSchemes</key>
    			<array>
    				<string>naver</string>
    			</array>
    		</dict>
       </array>
    
    📕 만약 카카오 로그인을 같이 사용시?
    
    - ios/Runner/AppDelegate.swift
    
    import NaverThirdPartyLogin
    
     override func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
             if url.absoluteString.hasPrefix("kakao"){
                super.application(app, open:url, options: options)
                return true
             } else if url.absoluteString.contains("thirdPartyLoginResult") {
                NaverThirdPartyLoginConnection.getSharedInstance().application(app, open: url, options: options)
                return true
             } else {
                return true
             }
         }
    ```
    
    - 네이버 로그인 함수
    
    ```jsx
    Future naverLogin() async {
    		// 로그인 정보 받아오기
        NaverLoginResult result = await FlutterNaverLogin.logIn();
    		// 토큰 제작 후 api로 토큰 제작
        final customNaverToken = await createNaverToken({
          'uid' : result.account.id,
          'displayName' : result.account.name,
          'email' : result.account.email,
        });
    		// customToken으로 firebase admin 로그인
        await FirebaseAuth.instance.signInWithCustomToken(customNaverToken);
      }
    
    📕 firebase admin api의 경우에는 카카오 로그인에서 다루었기 때문에 생략한다.
    ```
    

### 위 방법대로 모두 로그인 한다면 Firebase auth에 다 등록이 된다.

![스크린샷 2023-04-25 오전 11.11.16.png](/assets/png/flutter_login2.png)