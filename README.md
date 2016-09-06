# GCMtest
GCM test App info

* reference site : http://blog.saltfactory.net/android/implement-push-service-via-gcm.html
* 앱 등록 사이트 : https://developers.google.com/cloud-messaging/
* 앱 등록 사이트 : https://developers.google.com/mobile/add?platform=android&cntapi=gcm&cnturl=https:%2F%2Fdevelopers.google.com%2Fcloud-messaging%2Fandroid%2Fclient&cntlbl=Continue%20Adding%20GCM%20Support&%3Fconfigured%3Dtrue
* 앱을 등록하면 최종적으로 Server API Key와 Sender ID값이 발급된다.
* google-services.json 파일을 다운로드 받아서 안드로이드 스튜디오 프로젝트 모드에서 app/ 아래에 넣는다. 만약 이 파일이 없으면 R.string.gcm_defaultSenderId 리소스 에러가 난다.

< google-services.json >

{
  "project_info": {
    "project_id": "sf-gcm-demo-ae3be",
    "project_number": "636926190444",
    "name": "SF-GCM-DEMO"
  },
  "client": [
    {
      "client_info": {
        "client_id": "android:net.saltfactory.demo.gcm",
        "client_type": 1,
        "android_client_info": {
          "package_name": "net.saltfactory.demo.gcm"
        }
      },
      "oauth_client": [],
      "services": {
        "analytics_service": {
          "status": 1
        },
        "cloud_messaging_service": {
          "status": 2,
          "apns_config": []
        },
        "appinvite_service": {
          "status": 1,
          "other_platform_oauth_client": []
        },
        "google_signin_service": {
          "status": 1
        },
        "ads_service": {
          "status": 1
        }
      }
    }
  ]
}

* Android Studio에서 build.gradle 파일을 열어서 아래와 같이 classpath를 추가

// Top-level build file where you can add configuration options common to all sub-projects/modules.
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
//        classpath 'com.android.tools.build:gradle:1.2.3'
        classpath 'com.android.tools.build:gradle:1.3.0-beta1'
        classpath 'com.google.gms:google-services:1.3.0-beta1'
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}


allprojects {
    repositories {
        jcenter()
    }
}

* GCM은 Google Play Services로 통합이 되었다. Google Play Services 라이브러리를 사용하여 GCM 서비스를 이용할 수 있다.

apply plugin: 'com.android.application'
apply plugin: 'com.google.gms.google-services'

android {
    compileSdkVersion 22
    buildToolsVersion "22.0.1"

    defaultConfig {
        applicationId "net.saltfactory.demo.gcm"
        minSdkVersion 14
        targetSdkVersion 22
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.google.android.gms:play-services-gcm:7.5.+'
    compile 'com.android.support:appcompat-v7:22.1.1'

}

* AndroidManifest.xml 파일을 열어서 GCM 서비스를 만들기 위한 메타정보를 정의

< GCM Permission : 디바이스에 GCM 서비스를 사용하기 위한 권한 설정 >

<!-- [START gcm_permission] -->
    <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
<!-- [END gcm_permission] -->

< GCM Receiver : GCM을 받았을 때 동작하기 위한 리시버 >

<!-- [START gcm_receiver] -->
        <receiver
            android:name="com.google.android.gms.gcm.GcmReceiver"
            android:exported="true"
            android:permission="com.google.android.c2dm.permission.SEND">
            <intent-filter>
                <action android:name="com.google.android.c2dm.intent.RECEIVE" />
                <category android:name="net.saltfactory.demo.gcm" />
            </intent-filter>
        </receiver>
<!-- [END gcm_receiver] -->

< GCM Listener Service : GCM을 요청을 대기하고 있는 리스너 서비스 >

<!-- [START gcm_listener_service] -->
        <service
            android:name="net.saltfactory.demo.gcm.MyGcmListenerService"
            android:exported="false">
            <intent-filter>
                <action android:name="com.google.android.c2dm.intent.RECEIVE" />
            </intent-filter>
        </service>
<!-- [END gcm_listener_service] -->

< InstanceID Listener Service : InstanceID 요청을 대기하고 있는 리스너 서비스 >

<!-- [START instanceId_listener_service] -->
        <service
            android:name="net.saltfactory.demo.gcm.MyInstanceIDListenerService"
            android:exported="false">
            <intent-filter>
                <action android:name="com.google.android.gms.iid.InstanceID" />
            </intent-filter>
        </service>
<!-- [END instanceId_listener_service] -->

< GCM Registration Service : GCM을 등록하기 위한 서비스 >

<!-- [START gcm_registration_service] -->
        <service
            android:name="net.saltfactory.demo.gcm.RegistrationIntentService"
            android:exported="false"></service>
<!-- [END gcm_registration_service] -->

* < QuickstartPreferences.java >
이 파일은 GCM Demo에서 사용하는 LocalBoardcast의 액션을 정의한 파일이다. 
* < MainActivity.java >
GCM Demo의 메인 엑티비티 클래스를 정의하자. 
onCreate() : UI를 정의하고 이벤트와 핸들러를 정의한다.
onResume() : 화면이 보여질때 LocalBroadcastManager를 정의한다.
onPause() : 화면이 사라질때 LocalBoradcastManager에 등록된 것을 제거한다.
checkPlayService() : Google Play Service를 사용할 수 있는 환경인지 체크한다.
registBroadcastReciever() : LocalBroadcast 액션에 해당하는 작업을 정의한다.
getInstanceIdToken() : GCM을 등록하고 Instance ID에 해당하는 token을 가져온다.
* < RegistrationIntentService.java >
이 파일은 Instance ID를 가지고 토큰을 가져오는 작업을 한다.
* < MyInstanceIDListenerService.java >
이 파일은 Instance ID를 획득하기 위한 리스너를 상속받아서 토큰을 갱신하는 코드를 추가한다.
* < MyGcmListenerService.java >
이 파일은 GCM으로 메시지가 도착하면 디바이스에 받은 메세지를 어떻게 사용할지에 대한 내용을 정의하는 클래스
onMessageReceived() : GCM으로부터 이 함수를 통해 메세지를 받는다. 이때 전송한 SenderID와 Set 타입의 데이터 컬렉션 형태로 받게된다. 
sendNotification() : GCM으로부터 받은 메세지를 디바이스에 알려주는 함수이다. 

* nodejs GCM설치 : npm install node-gcm
* 서버코드

var gcm = require('node-gcm');
var fs = require('fs');

var message = new gcm.Message();

var message = new gcm.Message({
    collapseKey: 'demo',
    delayWhileIdle: true,
    timeToLive: 3,
    data: {
        title: 'saltfactory GCM demo',
        message: 'Google Cloud Messaging 테스트’,
        custom_key1: 'custom data1',
        custom_key2: 'custom data2'
    }
});

var server_api_key = ‘GCM 앱을 등록할때 획득한 Server API Key’;
var sender = new gcm.Sender(server_api_key);
var registrationIds = [];

var token = ‘Android 디바이스에서 Instance ID의 token’;
registrationIds.push(token);

sender.send(message, registrationIds, 4, function (err, result) {
    console.log(result);
});

* 실행명령 : node gcm_provider.js
