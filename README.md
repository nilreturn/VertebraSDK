# Vertebra SDK for iOS & macOS

**Vertebra SDK for iOS & macOS**는 (주)카카오 엔터프라이즈의 Logging 시스템에 접근할 수 있는 Swift로 작성되어 있는 SDK 입니다. 이 문서는 Vertebra SDK 사용법을 설명합니다.
---- 

### 1. 개요

기업의 데이터는 데이터 생산자, 처리자, 사용자에 따라 형태가 다르고 데이터 생성 이유나 처리 과정도 다르기 때문에, 서비스별로 각각 다른 데이터 형태를 갖습니다. 따라서 데이터를 저장하고 활용하는 것이 쉽지 않습니다. Vertebra(버테브라) SDK는 기업의 파편화된 데이터 관리 기준을 정립하고 데이터 처리 프로세스를 시스템화한 통합 데이터 수집 도구로, 데이터를 효율적으로 사용할 수 있는 데이터 거버넌스(Data Governance)를 지향합니다.

Vertebra SDK는 서비스 사용성 및 사용자 행동 등의 로그 데이터를 표준화된 규격으로 수집하며, 다양한 이벤트와 사용자 속성을 자동으로 포착합니다. 또한, 커스텀 이벤트를 직접 정의하여 비즈니스에 활용될 수 있는 가치 있는 데이터 항목을 수집하는 것도 가능합니다. Vertebra SDK로 수집된 데이터는 대시보드에서 확인할 수 있으며, 활성 사용자 수 등의 기본 정보부터 가장 많이 클릭된 버튼 등과 같은 상세 데이터에 이르기까지 다양한 세부 통계를 확인할 수 있습니다.

Vertebra(버테브라) SDK는 현재 카카오워크 홈페이지, 카카오워크 화상회의, 카카오워크 캘린더, 카카오워크 워크서비스 등에 적용되어 있으며, 카카오워크 서비스 전반으로 적용 범위가 확장될 예정입니다.

### 2. 요구사양

- iOS 13.0
- macOS 10.15.0
- Swift 5.2 이상
- Xcode 13.2 이상

### 3. 설치방법

Vertebra SDK는 SPM을 통하여 배포됩니다. Vertebra SDK 는 SPM을 이용한 xcframework를 제공합니다.

[^SPM(Swift Packages Manager]: Swift Packages Manager는 Xcode에서 제공하는 First Party Module Manager 입니다.

- Xcode 설정 방법 (**Xcode 13.2 기준**)

  1. Xcode 프로젝트 설정 -> [앱 타겟] 선택 -> General 탭으로 이동합니다.

  2. General 탭의 Frameworks, Libraries, and Embedded Content 에서 + 버튼을 선택합니다.

  ![](https://user-images.githubusercontent.com/71994299/150724737-30c87224-7ed7-488c-9402-218094c7e73d.png)

  3. 프레임워크 및 라이브러리 선택 메뉴 하단의 Add others를 선택한 후, Add Package Dependency... 메뉴를 선택합니다.

  ![](https://user-images.githubusercontent.com/71994299/150724849-62f510e3-c360-4a74-87ca-472090be2b63.png)

  4. SPM 매니저 윈도우 상단의 검색 창에 "https://github.com/nilreturn/VertebraSDK.git" 주소를 입력합니다.

  ![](https://user-images.githubusercontent.com/71994299/150724962-a4d45560-1cc3-4c3e-8500-85ae28635e82.png)

  5. VetebraSDK가 노출되면 Dependency Rule을 Up to Next Major Version과 SDK Version을 선택합니다. 선택이 완료된 후 Add Package 버튼을 눌러 프로젝트에 Vertebra SDK를 설치 완료합니다.

  ![](https://user-images.githubusercontent.com/71994299/150725046-456be6c5-9c3f-4af8-94a1-829d5a954905.png)

- SPM에서 설정하는 방법

  작성하는 Swift Package에 Vertebra SDK 를 설정 하는 방법은 간단합니다. 작성하고 있는 Swift Package의 Package.swift 파일의 dependencies에 Vertebra를 추가하면 됩니다. 

  ```swift
  dependencies: [
      .package(url: "https://github.com/nilreturn/VertebraSDK.git", .upToNextMajor(from: “0.9.0”))
  ]
  ```

### 4. 사용법

Vertebra SDK 는 Chaining 구조의 API를 구현합니다. Vertebra SDK의 사용은 **VertebraTracker**를 통해 이루어집니다. **VertebraTracker**로 시작되는 API들은 대부분은 chaining으로 구성되어 있으며, 최종적으로 **track(), flush()** Operator의 호출로 레코드를 종결하여 Logging 시스템으로 전달됩니다. **set()** operator에는 Verterbra Log Record 생성을 위한 여러 Data Type의 Object를 설정할 수 있습니다. 

- ### 초기화

Vertebra SDK 는 따로 초기화가 필요하지 않도록 설계되었습니다. VertebraTracker의 명령을 최초 실행하는 단계에서 기본 설정으로 초기화 합니다. 초기화 관련한 설정 정보는 VConfiguration을 확인하세요.

- ### 환경 설정

VertebraTracker는 기본 환경 설정에 의하여 작동합니다. track()을 진행하더라도 환경 설정에 맞추어서 로그 전송 동작을 진행합니다.  

```swift
VertebraTracker.setRequired {
  VConfiguration.default()
}
```

VertebraTracker의 기본 설정을 사용할수 있습니다. default()경우 직접 호출하지 않아서도 자동 적용됩니다. 현재 설정 사항의 경우 로그 레코드를 서버로 전송하기위한 레코드의 필요 갯수와 네트워크 타임아웃에 대한 설정이 있습니다. 기본값은 timeOut 5.0, Batch Size 1입니다.

- ### 필수 설정

공용 설정에서 카카오 엔터프라이즈와 협의된 ServiceName과 ServiceID를 반드시 입력해야 합니다. 

```swift
VertebraTracker
	.set {
    VCommon(serviceName: "vbt_sdktest_ios",
            deployment: "dev",
            serviceId: "X1-G1-C3-S2")}}
	}
```

- ### 부가 설정

Vertebra SDK는 부가 설정 함수가 존재합니다. setDevelopment(), setProduction()등과 같이 부가적으로 개발과 배포 환경에 따른 구분을 설정할 수 있습니다.

```swift
VertebraTracker.setDevelopment()
VertebraTracker.setProduction()
VertebraTracker.turnOffConsoleLogs()
VertebraTracker.turnOnConsoleLogs()
VertebraTracker.setBatchSize(5)
VertebraTracker.setTimeout(30.0)
VertebraTracker.useSandboxEnvironment()				// 추가스펙
VertebraTracker.donotUseSandboxEnvironment()	// 추가스펙
```

- ### Log 담기

VertrabraTracker는 레코드 단위로 데이터를 Logging 시스템에 전달하며, 각 레코드는 Action을 필수 값으로 가집니다. track() 함수 호출 실행하기 전에는 chainging 호출의 중복은 새로운 데이터로 치환됩니다. 

```swift
VertebraTracker
  .set { VAction(type:"page_view" ,name: "삭제뷰버튼", kind: "page_view") }
  .track(verbose: true)
```

위 예시의 경우 하나의 완결된 레코드를 생성하여 Vertebra SDK 큐에 전달합니다. 

```swift
VertebraTracker
  .set { VAction(type:"page_view" ,name: "삭제뷰버튼", kind: "page_view") }
  .set { VAction(type: .location ,name: “위치 테스트, kind: “geo”) }
  .track(verbose: true)
```

위의 경우, 하나의 레코드가 완료 되기 전에 새로운 액션을 설정하였기 때문에 Record의 Action에 대한 값이 치환됩니다. track()의 결과는 마지막으로 설정되어 있는 Action값이 최종적으로 전달됩니다.

```swift
VertebraTracker
  .set { VAction(type: .pageView, name: "some", kind: "Some 버튼 노출")}
  .set { VClick(type: "test") }
  .track()
  .set { VAction(type: .event, name: "some", kind: "Some 버튼 클릭")}
  .track()
```

위의 예시의 경우, track() 한번 발생시킨 후에 새로운 Action을 설정하고 track()을 재 호출하였습니다. 위의 경우 두개의 로그 레코드가 생성되어 Vertebra SDK에 전달됩니다. 이 레코드들은 VertebraTracker에 설정된 Configuration기준에 의해 서버에 전달됩니다.

```swift
VertebraTracker
  .set { VAction(type: .pageView, name: "some", kind: "Some 버튼 노출")}
  .set { VClick(type: "test") }
  .track()
  .set { VAction(type: .event, name: "some", kind: "Some 버튼 클릭")}
  .track()
  .flush()
```

flush() 함수를 사용하여 Configuration에 설정된 값과는 무관하게 생성된 두 로그 레코드를 바로 Vertebra서버에 전송을 시도합니다.

Vertebra SDK 는 서버 전송시 실패한 경우, 로그를 시스템내의 공간에 따로 보관을 시도합니다. 네트워크 상황이 변경이 되었을때, 보관된 로그를 자동으로 우선적으로 전송을 시도합니다.

- #### VertebraTracker 상세

  | 메소드               | 설명                                                         |
  | -------------------- | ------------------------------------------------------------ |
  | set()                | 각종 로그 스킴 Data를 설정할수 있습니다. set()으로 기록된 데이터들은 track()이나 getData() 호출 전까지 Overwrite 됩니다. |
  | track()              | 로그를 기록합니다. track은 VertebraTracker에 설정된 기본 동작에 의해서 실제로 서버에 기록한 로그를 전송합니다. |
  | flush()              | 로그를 서버에 바로 전송합니다. track() 함수에 의해 기록된 로그들을 Configuration 동작과는 상관없이 바로 서버로 전송합니다. |
  | getData()            | 로그 데이터를 Json 형태의 스트링으로 표시합니다. getData를 하면 Log Record는 Vertebra Tracker에 기록되지 않습니다. (Debug 용도) |
  | setDevelopment()     | Vertebra Tracker를 개발용으로 설정합니다.                    |
  | setProduction()      | Vertebra Tracker를 배포용으로 설정합니다.(기본값)            |
  | turnOffConsoleLogs() | 개발중, Vertebra Tracker에서 발생하는 콘솔 로그를 끕니다.    |
  | turnOnConsoleLogs()  | 개발중, Vertebra Tracker에서 발생하는 콘솔 로그를 켭니다.    |
  | setBatchSize()       | VertebraTracker가 서버에 전달할 조건이 되는 Batch Size를 설정합니다. Vertebra Tracker는 이 배치 사이즈 만큼의 로그데이터가 쌓일때에 서버로 실제 전송이 일어 납니다. |
  | setTimeout()         | VertebraTracker가 서버에 로그 데이터를 전달할 Timeout 시간을 설정합니다.  전송의 실패시에는 VertebraTracker는 데이터를 따로 저장했다가 추후 재 활성시에 로그 전송을 시도합니다. |
  | setRequired()        | VerterbraTracker의 기본 설정을 지정할수 있습니다. (생략 가능). 호출하지 않을경우, VertebraTracker는 기본 Configuration이 지정됩니다. (timeout 5.0 batchSize 1) |
  
  
  
  ```swift
  VertebraTracker.setDevelopment()
  VertebraTracker.setProduction()
  VertebraTracker.turnOffConsoleLogs()
  VertebraTracker.turnOnConsoleLogs()
  VertebraTracker.setBatchSize(5)
  VertebraTracker.setTimeout(30.0)
  VertebraTracker.track();                // 콘솔에 아무것도 출력하지 않고, 로그 수집 서버에 메시지를 전송
  VertebraTracker.track(verbose: true);            // 콘솔에 원문을 출력하고, 로그 수집 서버에 메시지를 전송
  VertebraTracker.track(verbose: true, disabled: true);      // 콘솔에 원문을 출력하고, 로그 수집 서버에 어떠한 메시지도 전송하지 않음
  
  // Chaining Style
  VertebraTracker
  	.setDevelopment()
  	.trunOffConsoleLogs()
  	.set {
      VCommon(serviceName: "hello vertebra. Abracadabra!!!!",
              deployment: "dev",
              serviceId: "XX-XX-XX-XX")
     }
  	.set { VAction(type: "pageView", name: "페이지", kind: "pageView") }
  	.track()
  ```
  
  

### 5. 스키마 상세

모든 스키마 Type은 VertebraTracker의 set()메소드를 통해 설정됩니다.

| 카테고리    | 분류                     | 메소드 | 설명                                                         |
| ----------- | ------------------------ | ------ | ------------------------------------------------------------ |
| 기본정보    | VCommon                  | set()  | 서비스에 대한 기본 정보                                      |
| 기본정보    | VId                      | set()  | 분석에 사용되는 사용자 구분값  **- 최소 하나의 비식별 아이디 권장** |
| 기본정보    | VUtm                     | set()  | Urchin Tracking Module 광고 트래킹 정보                      |
| 도메인 정보 | VPage                    | set()  | 사용자에게 서비스되고 있는 콘텐츠 페이지 정보                |
| 도메인 정보 | VViewImpressionContent   | set()  | 사용자에게 특정되어 제공된 콘텐츠의 사용/전환 여부를 나타내는 Impression feedback 정의 값. Array 형식으로 여러개를 묶어서 넣을수 있습니다. |
| 도메인 정보 | [VViewImpressionContent] | set()  |                                                              |
| 도메인 정보 | VDomain                  | set()  | 특정 도메인 정의 값(ex. 쇼핑, 검색 등등)                     |
| 도메인 정보 | VPromotion               | set()  | 프로모션 관련 정보                                           |
| 사용자 행동 | VAction                  | set()  | 로그 종류 구분에 사용되는 사용자의 특정 행동을 정의한 값. 모든 각 Vertebra 로그 레코드는 VAction 레코드를 필수 값으로 설정되어야 함. |
| 사용자 행동 | VClick                   | set()  | 사용자가 클릭한 이벤트에 대한 상세 정보.                     |
| 사용자 행동 | VUsage                   | set()  | 사용자의 스크롤과 머문 시간등의 사용성 정보                  |
| 사용자 행동 | VStatus                  | set()  | API call 결과 등의 HTTP 상태 코드 정보                       |
| 기타        | VAgreement               | set()  | 3자 정보 제공 동의 여부(광고, 쿠키, 위치)                    |
| 기타        | VLocation                | set()  | 사용자의 접속 위치 정보                                      |
| 기타        | VBucket                  | set()  | A/B 테스트를 위한 Bucket 정보가 들어갑니다. Array 형식으로 여러개를 묶어서 넣을수 있습니다. |
| 기타        | [VBucket]                | set()  |                                                              |
| 설정        | VConfiguration           | set()  | VertebraTracker의 기본 동작에 관련한 설정 정보. (0.9.0 기준 Batch Size와 Timeout 정보가 설정 가능합니다.) |

- VCommon

  서비스에 대한 기본 정보를 설정합니다.

  ```swift
  VertebraTracker
    .set {
        VCommon(serviceName: "hello vertebra. Abracadabra!!!!",
                deployment: "dev",
                serviceId: "XX-XX-XX-XX")
    }
  ```

  | 생성자 파라미터           | 타입   | 필수 여부 | 설명                                                         |
  | ------------------------- | ------ | --------- | ------------------------------------------------------------ |
  | serviceName               | String | required  | 서비스별 전송된 로그를 구별하기 위한 인덱스명 - 유일값이어야 하며, 데이터 거버넌스와 협의 필요 ex) kakaowork |
  | deployment                | String | required  | 현재 서비스 개발 상태 ex) dev, sandbox, cbt, prod            |
  | svcDomain                 | String | optional  | BundleIdentifier ex) io.kakaoenterprise.android.kakaowork    |
  | serviceId                 | String | optional  | 내부 서비스 아이디(데이터 거버넌스와 협의한 이름) ex) X1-G0-C0-S0 |
  | accessTimeStamp           | Int    | optional  | 사용자 이벤트 발생 시간(단위: micro second) ex) 1527037694000 |
  | serviceType               | String | optional  | 서비스타입                                                   |
  | serviceCode               | String | optional  | 서비스코드                                                   |
  | serviceCodeId             | String | optional  | 서비스코드ID                                                 |
  | serviceCodeGroup          | String | optional  | 서비스코드그룹                                               |
  | serviceCodeGroupId        | String | optional  | 서비스코드그룹ID                                             |
  | serviceOrganizationCode   | String | optional  | 서비스법인코드                                               |
  | serviceOrganizationCodeId | String | optional  | 서비스법인코드ID                                             |
  | displayName               | String | optional  | 외부에서 보여지는 서비스 이름 ex) kakaowork                  |
  | kakaoAppKey               | String | optional  | kakao developers에서 발급 받은 app_key ex) e9c09cfdd918edc72b7727d657b5ae38 |
  | url                       | String | optional  | 현재 페이지 URL ex) https://kakaoenterprise.com/login        |
  | refrerrer                 | String | optional  | 이전 페이지 URL ex) https://kakaoenterprise.com              |
  | clogSeq                   | String | optional  | 클라이언트앱에서 전송하는 로그의 sequence. 세션단위로 증가   |
  | tChannel                  | String | optional  | 유입 파라미터분석(딥링크 유입 시 정보를 추가로 보내야 함) - 다음앱, 톡채널, 플친(로켓)등 플랫폼의 인앱브라우저에서 소비된 경우 해당 매체명을 표시 ex) talk-channel |
  | tSrc                      | String | optional  | 유입 소스 ex) daum                                           |
  | tCh                       | String | optional  | 유입 채널 ex) DisplayAD                                      |
  | tObj                      | String | optional  | 유입 항목 ex) 2019X-mas                                      |
  | tMsgId                    | String | optional  | 유입된 메시지의 id ex) 123a                                  |
  
- VId

  데이터 분석 시 사용되는 사용자 구분값으로, 최소 하나의 비식별 아이디를 사용할 것을 권장합니다.

  ```swift
  VertebraTracker
    .set {
        VId(accountId: "a1sd2d35", accessToken: "xxxxx-xxxx-xxxxx")
    }
  ```

  | 생성자 파라미터 | 타입   | 필수여부 | 설명                                                         |
  | --------------- | ------ | -------- | ------------------------------------------------------------ |
  | accountId       | String | optional | [식별] 카카오계정 ex) a1sd2d35                               |
  | accessToken     | String | optional | [식별] access_token을 통해 account_id를 추출하는데 사용      |
  | appUserId       | String | optional | [식별] 앱마다 고유한 사용자 식별ID                           |
  | talkUserId      | String | optional | [식별] talk_user_id                                          |
  | daumUserId      | String | optional | [식별] daum user_id                                          |
  | melonId         | String | optional | [식별] 멜론 ID                                               |
  | appId           | String | optional | [비식별] 애플리케이션 ID                                     |
  | slid            | String | optional | [비식별] 서버로그 ID로, 서버에서 로그가 발생할때 마다 생성되는 UUID - 해당 값은 클라이언트 로그와 서버 로그를 Join하여 분석할 때 사용됨 ex) 1b9d6bcd-bbfd-4b2d-9b5d-ab8dfbbd4bed |
  | clid            | String | optional | [비식별] 클라로그 ID로, track 호출 시 생성되는 UUID로, 네트워크 문제로 중복되는 값을 처리하기 위한 값 ex) 9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d |
  | uuid            | String | optional | [비식별] 기존 Tiara UUID                                     |
  | webId           | String | optional | [비식별] 웹 비식별자                                         |
  | webIdEnabled    | Bool   | optional | [비식별] 웹 비식별자 허용여부                                |
  | adid            | String | optional | [비식별] 광고 ID                                             |
  | adidEnabled     | Bool   | optional | [비식별] 광고 ID 허용여부                                    |
  | customId        | String | optional | [비식별] 사용자 정의 ID - 비식별화 된 사용자별 지표를 쉽게 확인 가능(입력 권장) |
  | customGroupId   | String | optional | custom_id 를 묶을 수 있는 그룹 키                            |
  
- VUtm

  UTM(Urchin Tracking Module)의 광고 트래킹 오브젝트입니다.

  ```swift
  VertebraTracker
    .set {
        VUtm(type: "Marketing", utmSource: "facebook.com")
    }
  ```

  | 생성자 파라미터 | 타입   | 필수여부 | 설명                                                         |
  | --------------- | ------ | -------- | ------------------------------------------------------------ |
  | type            | String | optional | 앱설치만 있음, 확장을 위해 추가                              |
  | utmSource       | String | optional | 유입소스 - 매 전송 시마다 추가됨 ex) facebook.com, twitter, google, kakao |
  | utmMedium       | String | optional | 유입 매채. 매전송시 추가 ex) social                          |
  | utmCampaign     | String | optional | 유입 캠패인명 ex) buffer                                     |
  | utmContent      | String | optional | 유입 콘텐츠명 ex) buffercf3b2                                |
  | utmTerm         | String | optional | 유입 키워드 값 ex) 과일, 사과                                |
  | timestamp       | Int    | optional | 유입 시점의 Unix Timestamp                                   |
  
- VPage

  사용자에게 서비스 되고 있는 콘텐츠 페이지 정보를 수집합니다.

  ```swift
  VertebraTracker
    .set {
      VPage(id: "20180525102916427", name: "손흥민 골 잔치", title: "손흥민, 12호골 추가")
    }
  ```

  | 생성자 파라미터 | 타입   | 필수여부 | 설명                                                         |
  | --------------- | ------ | -------- | ------------------------------------------------------------ |
  | id              | String | required | 현재 페이지를 담고있는 내용의 ID 값 - 특정 ID에 해당된다면 해당 ID 전송 ex) 20180525102916427 |
  | type            | String | optional | 해당 ID 종류 - 해당 값은 서비스에서 결정되는 값을 그대로 사용 ex) news |
  | name            | String | required | 페이지명 ex) 손흥민 골 잔치                                  |
  | title           | String | required | 제목 ex) 손흥민, 12호골 추가                                 |
  | tab             | String | optional | 서비스 탭 이름 ex) 통합검색                                  |
  | tabId           | String | optional | 서비스 탭 ID ex) 12                                          |
  | category        | String | optional | 카테고리 ex) 스포츠/레저                                     |
  | categoryId      | String | optional | 카테고리 ID ex) 123                                          |
  | subcategory     | String | optional | 서브 카테고리 ex) 축구                                       |
  | subcategoryId   | String | optional | 서브 카테고리 ID ex) 1234                                    |
  | image           | String | optional | 페이지 대표 이미지의 URL ex) http://t1.daumcdn.net/webtoon/op/ |
  | plink           | String | optional | 페이지의 영구 링크(Permanent link url) ex) http://webtoon.daum.net/webtoon/viewer/49858 |
  | ecommerceYn     | Bool   | optional | 전자상거래 여부                                              |
  | provider        | String | optional | 해당 페이지 제공자 ex) 다음 카페                             |
  | providerId      | String | optional | 해당 페이지 제공자 ID ex) 567                                |
  | providerType    | String | optional | 해당 페이지 제공자 타입 ex) 개별 카페                        |
  | tags            | String | optional | 해당 페이지 태그 정보 - 쉼표(,)로 구분 ex) 축구, 손홍민      |
  
- VViewImpressionContent, [VViewImpressionContent]

  콘텐츠의 Impression feedback 정의 값을 수집합니다. Object 형식으로 여러개를 묶어서 넣을 수 있습니다.

  ```swift
  VertebraTracker
    .set {
      VViewImpressionContent(impressionId: "vp_1234567", impressionType: "news")
    }
  VertebraTracker
    .set {
      [
        VViewImpressionContent(impressionId: "vp_1234567", impressionType: "news"),
        VViewImpressionContent(impressionId: "vp_1234568", impressionType: "ad")
      ]
    }
  ```

  | 생성자 파라미터 | 타입   | 필수여부 | 설명                                                         |
  | --------------- | ------ | -------- | ------------------------------------------------------------ |
  | impressionId    | String | optional | 추천 또는 개인화 등에 따라 제공된 콘텐츠의 노출 원인을 식별할 수 있는 고유 구분자 ex) vp_1234567 |
  | impressionType  | String | optional | 제공된 콘텐츠 타입 ex) news                                  |
  | impressionArea  | String | optional | 제공된 콘텐츠 영역 ex) news                                  |
  | id              | String | optional | 콘텐츠ID ex) 20180430215456004                               |
  | type            | String | optional | 해당 ID에 대한 ID 종류 - 해당 값은 서비스에서 결정되는 값을 그대로 사용 ex) episode |
  | name            | String | optional | 해당 ID에 대한 이름 ex) 냉부해' 김성령 '남편, 사업 탓 부산 거주..각자 잘 산다' |
  | category        | String | optional | 해당 ID에 대한 카테고리 ex) 드라마                           |
  | provider        | String | optional | 노출된 콘텐츠 또는 목록 서비스의 제공자 ex) jtbc             |
  | author          | String | optional | 노출된 콘텐츠 또는 목록 서비스의 저자 ex) jtbc               |
  
- VDomain

  특정 도메인에 대한 값을 저장합니다. 

  ```swift
  VertebraTracker
    .set {
      VDomain(search: VSearch(keyword: "바이든", type: "suggest"))
    }
  VertebraTracker
    .set {
      VDomain(purchase: VPurchase(name: "Magic trackpad 2", id: "xx13413adfa13"))
    }
  VertebraTracker
    .set {
      VDomain(chat: VChat(utterance: "안녕하세요, 노래 틀어줘.."))
    }
  VertebraTracker
    .set {
      VDomain(videoMeeting: VVideoMeeting(meetingId: "xx00001", micStatus: "on"))
    }
  VertebraTracker
    .set {
      VDomain(search: VSearch(keyword: "바이든", type: "suggest"),
        purchase: VPurchase(name: "Magic trackpad 2", id: "xx13413adfa13"),
        chat: VChat(utterance: "안녕하세요, 노래 틀어줘..")
        videoMeeting: VVideoMeeting(meetingId: "xx00001", micStatus: "on"))
    }
  ```

  | 생성자 파라미터 | 타입          | 필수여부 | 설명                             |
  | --------------- | ------------- | -------- | -------------------------------- |
  | search          | VSearch       | optional | 검색 도메인 정보                 |
  | purchase        | VPurchase     | optional | 판매 도메인 정보                 |
  | chat            | VChat         | optional | 채팅 서비스 도메인 정보          |
  | videoMeeting    | VVideoMeeting | optional | 화상솔루션(화상채팅) 도메인 정보 |
  
  - VSearch

  | 생성자 파라미터 | 타입     | 필수여부 | 설명                                                        |
  | --------------- | -------- | -------- | ----------------------------------------------------------- |
  | keyword         | String   | optional | 키워드 ex) 바이든                                           |
  | type            | String   | optional | 검색 방법 - suggest : 추천된 키워드 - typing: 입력된 키워드 |
  | tab             | String   | optional | 노출된 결과 갯수 ex) 50                                     |
  | resultNum       | String   | optional | 검색이 제공된 탭 정보                                       |
  | tabOrder        | [String] | optional | 탭의 노출 순서 ex) [chatting, members, ...]                 |
  
  - VPurchase

  | 생성자 파라미터 | 타입   | 필수여부 | 설명                                             |
  | --------------- | ------ | -------- | ------------------------------------------------ |
  | name            | String | optional | 제품명 ex) Magic trackpad 2                      |
  | id              | String | optional | 제품 고유 식별자                                 |
  | orderId         | String | optional | 구매번호 ex) AB-20211116-CHU-120                 |
  | brand           | String | optional | 브랜드명 ex) Apple                               |
  | brandId         | String | optional | 브랜드 ID ex) A01                                |
  | category        | String | optional | 제품 카테고리 ex) 컴퓨터 주변기기                |
  | categoryId      | String | optional | 제품 카테고리 ID ex) C003                        |
  | subcategory     | String | optional | 하위 카테고리 ex) 입력기기                       |
  | subcategoryId   | String | optional | 하위 카테고리 ID ex) H0001                       |
  | seller          | String | optional | 판매자 ex) kakao                                 |
  | sellerId        | String | optional | 판매자 ID ex) 1                                  |
  | payment         | String | optional | 지불방법 ex) credit                              |
  | currency        | String | optional | 통화 ex) krw                                     |
  | shipAmount      | Int    | optional | 배송료 ex) 3000                                  |
  | couponAmount    | Int    | optional | 쿠폰 사용금액 ex) 3000                           |
  | discountAmount  | Int    | optional | 쿠폰 외 할인금액 ex) 2000                        |
  | feeAmount       | Int    | optional | 수수료 ex) 1200                                  |
  | price           | Float  | optional | 제품 가격 ex) 210000                             |
  | quantity        | Int    | optional | 제품 구매수 ex) 1                                |
  | totalAmount     | Int    | optional | 총 구매금액 ex) 210800                           |
  | subscribeYn     | Bool   | optional | 구독 여부 - true : 구독 희망 - false : 구독 해지 |
  
  - VChat

  | 생성자 파라미터 | 타입   | 필수여부 | 설명                                                         |
  | --------------- | ------ | -------- | ------------------------------------------------------------ |
  | utterance       | String | optional | 채팅/발화 내용 ex) 안녕하세요, 노래 틀어줘..                 |
  | utteranceTime   | String | optional | 채팅/발화 시간 ex) 1527037694000                             |
  | type            | String | optional | 채팅/발화 타입 - 채팅 im: 나와의 대화방 dm: 개인 대화방 group : 그룹 대화방 org: 조직 대화방 xdm: 워크스페이스 외부자 개인 대화방 xgroup : 워크스페이스 외부자 포함 그룹 대화방 - 발화 speech : 음성 입력text : 문자 입력 |
  | answered        | String | optional | 채팅/발화 답변 내용 ex) 네 안녕하세요, 팝송 틀어드릴게요..   |
  | id              | String | optional | 현재 채팅/발화 내용을 담고 있는 내용의 ID 값 - 특정 ID에 해당 된다면 해당 ID 전송 ex) 1 |
  
  - VVideoMeeting

  | 생성자 파라미터   | 타입     | 필수여부 | 설명                                                         |
  | ----------------- | -------- | -------- | ------------------------------------------------------------ |
  | meetingId         | String   | optional | 미팅 식별자 ex) xx00001                                      |
  | meetingType       | String   | optional | 미팅 종류 ex) PREMIUM, FREE                                  |
  | agentUUID         | String   | optional | 미팅을 실행하는 Agent 식별자 ex) web                         |
  | micStatus         | String   | optional | 마이크 상태 - on: 마이크를 켬 - off: 마이크 끔               |
  | micEffect         | String   | optional | 마이크 특수효과 - on: 마이크 특수효과 킴 - off: 마이크 특수효과 끔 - effectName: echo, reverb, robot 등 |
  | micDevice         | String   | optional | 선택된 마이크 장비 정보 ex) Macbook Pro 마이크(Built-in)     |
  | cameraStatus      | String   | optional | 카메라 상태 - on: 켜짐 - off: 꺼짐                           |
  | cameraEffect      | String   | optional | 카메라 특수 효과 - on: 카메라 특수효과 킴 - off: 카메라 특수효과 끔 - effectName: Animation_02 |
  | cameraDevice      | String   | optional | 선택된 카메라 장비 정보 ex) FaceTime HD 카메라(내장형)       |
  | pipStatus         | Bool     | optional | PIP(Picture in Picture) 상태                                 |
  | pipInfo           | String   | optional | PIP 정보 ex) PIP position, Size 또는 상태코드                |
  | screenShareStatus | String   | optional | 화면공유 상태                                                |
  | screenShareType   | String   | optional | 화면공유 종류 ex) screen, windows, tab                       |
  | speechRequest     | String   | optional | 발언 요청 상태 ex) requested, approved, ignored, on, off 등  |
  | reaction          | String   | optional | 리액션 ex) clap                                              |
  | reactionType      | String   | optional | 리액션 ex) emoji                                             |
  | invitedCount      | Int      | optional | 초대된 인원 수                                               |
  | participantsCount | Int      | optional | 참여 인원 수                                                 |
  | utterances        | [String] | optional | 발화 내용 - 발화내용 음성인식등에 사용 - 내부 채팅일 경우에는 domain(3) chat을 사용 |
  | utteranceType     | String   | optional | 발화 타입 ex) caption                                        |
  | utteranceDuration | Int      | optional | 발화 시간                                                    |
  
- VPromotion

  프로모션에 대한 정보를 수집합니다. 지원하는 도메인은 [promotion]()입니다.

  ```swift
  VertebraTracker
    .set {
      VPromotion(name: "카톡 이모티콘 30일 무료", id: "kko-emt-001")
    }
  ```

  | 생성자 파라미터 | 타입   | 필수여부 | 설명                                                         |
  | --------------- | ------ | -------- | ------------------------------------------------------------ |
  | name            | String | optional | 이름 ex) 카톡 이모티콘 30일 무료                             |
  | id              | String | optional | 이벤트 ID - 특정 ID에 해당된다면 해당 ID 전송 ex) kko-emt-001 |
  | type            | String | optional | 프로모션 타입 ex) free-trial                                 |
  | category        | String | optional | 카테고리 ex) emoticon                                        |
  | categoryId      | String | optional | 카테고리 ID ex) kko-emt                                      |
  | banner          | String | optional | 프로모션 배너 Title ex) 카카오 메일 만들고 라이언 이모티콘 받자 |
  | bannerId        | String | optional | 프로모션 배너 ID ex) kko-emt-bnn-001                         |
  | bannerUrl       | String | optional | 대표 배너 이미지 URL ex) http://cnd.image.location           |
  | promotionUrl    | String | optional | 프로모션 페이지 URL ex) https://promotion.page.url           |
  | coupon          | String | optional | 쿠폰명 ex) 라이언 이모티콘 30일 무료 사용권                  |
  | couponId        | String | optional | 쿠폰 ID ex) kki-emt-cpn-001abn                               |
  | tags            | String | optional | 프로모션 태그 ex) #카카오톡 #이모티콘                        |
  
- VAction

  Action 도메인은 사용자 행동에 대한 값으로 로그 종류 구분에 사용됩니다. (모든 로그 레코드의 필수값으로 사용되어 집니다.)

  ```swift
  VertebraTracker
    .set {
      VAction(type: "page_view", name: "페이지", kind: "page_view")
    }
  VertebraTracker
    .set {
      VAction(type: .pageView, name: "페이지", kind: "page_view")
    }
  ```

  | 생성자 파라미터 | 타입                  | 필수여부 | 설명                                                         |
  | --------------- | --------------------- | -------- | ------------------------------------------------------------ |
  | type            | String or VActionType | required | 로그 종류별로 구분                                           |
  | name            | String                | required | 액션에 대한 구체적인 설명 - 명사+동사 형태로 구체적인 페이지명과 행동 작성(ex. page_view-xxx화면_보기) - 영문, 숫자, _, 한글 지원 - 서비스 내에서 유일값으로 사용을 권장 |
  | kind            | String                | required | 주요 액션에 대한 사전등록된 값을 입력                        |
  
  ** VActionType 
  
  + VActionType은 VAction생성자에서 사용되는 enum 상수 입니다.
  
    | 종류                 |                                                              |
    | -------------------- | ------------------------------------------------------------ |
    | pageView             | page_view 를 나타냅니다.                                     |
    | event                | event를 나타냅니다.                                          |
    | ecommerce            | ecommerce를 나타냅니다.                                      |
    | usage                | usage를 나타냅니다.                                          |
    | viewImpression       | view_imp를 나타냅니다.                                       |
    | location             | location를 나타냅니다.                                       |
    | app                  | app를 나타냅니다.                                            |
    | custom(type: String) | 정의된 목록 이외의 커스텀한 액션타입을 선언할 경우 사용합니다. ex) .custom("hybirdAction") |
  
- VClick

  사용자가 클릭한 이벤트에 대한 상세 정보를 전달합니다.

  ```swift
  VertebraTracker
    .set {
      VClick(type: "link", layer1: "international_top_sellers")
    }
  ```

  | 생성자 파라미터 | 타입   | 필수여부 | 설명                                                         |
  | --------------- | ------ | -------- | ------------------------------------------------------------ |
  | type            | String | optional | 클릭 시 타입 (button, image, logo, 등) ex) link              |
  | layer1          | String | optional | 분석 구분자 대분류 ex) international_top_sellers             |
  | layer2          | String | optional | 분석 구분자 중분류 ex) home_entertainment                    |
  | layer3          | String | optional | 분석 구분자 소분류 ex) gaming_console                        |
  | clickUrl        | String | optional | 클릭한 LInk 의 URL 주소                                      |
  | setnum          | String | optional | Carousel 등 사용시 노출되는 목록의 판 번호                   |
  | ordnum          | String | optional | 목록 노출시 배열 순서                                        |
  | copy            | String | optional | 클릭한 콘텐츠 title ex) 포토 [실시간 풍계리 취재] 남쪽 기자들의 풍계리... |
  | imageUrl        | String | optional | 클릭한 이미지의 URL 주소                                     |
  | posX            | Int    | optional | 클릭 이벤트의 X 좌표의 값                                    |
  | posY            | Int    | optional | 클릭 이벤트의 Y 좌표의 값                                    |
  | impId           | String | optional | 제공된 콘텐츠의 노출 원인을 식별할 수 있는 고유 구분자 ex) 추천, 개인화 등의 서비스 제공자가 사용하는 식별자 |
  | impProvider     | String | optional | 노출된 콘텐츠 또는 목록 서비스의 제공자 ex) SBS              |
  | campaignId      | String | optional | 노출된 콘텐츠가 속한 캠페인의 구분자 ex) 1                   |
  | bucketId        | String | optional | 노출된 콘텐츠가 속한 버킷의 구분자 ex) AB 테스팅용 버킷 ID   |
  
- VUsage

  사용자의 페이지 사용성 정보를 수집합니다.

  ```swift
  VertebraTracker
    .set {
      VUsage(duration: 12000, scrollInnerHeight: 300)
    }
  ```

  | 생성자 파라미터   | 타입  | 필수여부 | 설명                                                     |
  | ----------------- | ----- | -------- | -------------------------------------------------------- |
  | duration          | Int   | optional | 사용자가 페이지에 머무는 체류시간(ms) ex) 12000          |
  | scrollPercent     | Float | optional | 페이지 스크롤 비율(scroll_bottom / scroll_height) ex) 67 |
  | scrollHeight      | Int   | optional | 스크롤 깊이(전체화면 길이) ex) 600                       |
  | scrollTop         | Int   | optional | 스크롤 상단(측정시 상단 y값) ex) 100                     |
  | scrollInnerHeight | Int   | optional | 스크롤 화면크기(화면에 보이는 스크롤 길이) ex) 300       |
  | scrollBottom      | Int   | optional | 스크롤 하단(측정시 하단 y 값) ex) 400                    |
  
- VStatus

  HTTP 상태에 대한 정보입니다.

  ```swift
  VertebraTracker
    .set {
      VStatus(code: 200, method: "POST", message: "OK")
    }
  ```

  | 생성자 파라미터 | 타입   | 필수여부 | 설명             |
  | --------------- | ------ | -------- | ---------------- |
  | code            | Int    | optional | HTTP Status code |
  | method          | String | optional | HTTP Method      |
  | path            | String | optional | HTTP Url path    |
  | message         | String | optional | HTTP message     |
  
- VAgreement

  사용자의 정보 수집 동의 여부에 대한 값을 수집합니다.

  ```swift
  VertebraTracker
    .set {
      VAgreement(locationInformationAgree: true, cookieAgree: false)
    }
  ```

  | 생성자 파라미터          | 타입 | 필수여부 | 설명                                 |
  | ------------------------ | ---- | -------- | ------------------------------------ |
  | locationInformationAgree | Bool | optional | 위치정보 동의 여부. 계정 동의에 해당 |
  | thirdProviderAgree       | Bool | optional | 제3자 제공동의 여부                  |
  | thirdAdAgree             | Bool | optional | 광고마케팅 제공 동의                 |
  | cookieAgree              | Bool | optional | 쿠키 허용 여부                       |
  | donotTrackAgree          | Bool | optional | Do not track 여부                    |
  
- VLocation

  위치 정보 제공에 동의한 사용자의 위치 정보를 수집합니다.

  ```swift
  VertebraTracker
    .set {
      VLocation(latitude: 37.40176288188301, logitude: 127.10858357786297)
    }
  ```

  | 생성자 파라미터 | 타입   | 필수여부 | 설명                                              |
  | --------------- | ------ | -------- | ------------------------------------------------- |
  | latitude        | Double | optional | 위도 ex) 37.40176288188301                        |
  | logitude        | Double | optional | 경도 ex) 127.10858357786297                       |
  | country         | String | optional | 나라 ex) Republic of Korea                        |
  | city            | String | optional | 도시 ex) Gyeonggi-do                              |
  | address1        | String | optional | 주소 1 ex) Pangyoyeok-ro, Bundang-gu, Seongnam-si |
  | address2        | String | optional | 주소 2 ex) 235                                    |
  | zipCode         | String | optional | 우편번호 ex) 13494                                |
  
- VBucket, [VBucket]

  A/B 테스트를 위한 Bucket 정보가 들어갑니다. Array 형식으로 여러개를 묶어서 넣을수 있습니다.

  ```swift
  VertebraTracker
    .set {
      VBucket(name: "Bucket")
    }
  VertebraTracker
    .set {
      [
        VBucket(name: "Bucket1"),
        VBucket(name: "Bucket2")
      ]
    }
  ```

  | 생성자 파라미터 | 타입   | 필수여부 | 설명             |
  | --------------- | ------ | -------- | ---------------- |
  | name            | String | optional | 버킷테스트명     |
  | group           | String | optional | 버킷테스트그룹명 |
  | experimentKey   | String | optional | 실험키           |
  | variationKey    | String | optional | variation        |
  | idType          | String | optional | 식별자타입       |
  
- VConfiguration

  VertebraTracker의 기본 설정에 관련한 정보입니다.

  ```swift
  VertebraTracker
    .setRequired {
      VConfiguration(timeout: 10.0, batchSize: 5)
    }
  
  // 기본(생략 가능)
  VertebraTracker
    .setRequired {
      VConfiguration.default()
    }
  ```
  
  
  
  | 생성자 파라미터 | 타입         | 필수여부 | 설명                                                   |
  | --------------- | ------------ | -------- | ------------------------------------------------------ |
  | timeout         | TimeInterval | required | Vertebra Tracker의 네트워크 타입아웃 값. default = 5.0 |
  | batchSize       | UInt         | required | Vertebra Tracker의 로그 전송 사이즈. 기본값 1          |
  
  