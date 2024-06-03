
# Signing iOS App
디지털 서명,  Code Signing의 세 가지 주요 단계, `codesign` 명령줄 도구를 사용한 앱 서명 및 확인 프로세스에 대해 살펴보겠습니다. 

![](https://raw.githubusercontent.com/MintSwift/MintImage/main/WeeklyOriginal/20240603-Signing-iOS-App.jpg)

# Code Signing  
최신 버전의 Xcode에서는 모든 코드 서명 활동이 내부적으로 Xcode에서 처리됩니다. 코드 서명 ID를 지정하기만 하면 Xcode가 모든 것을 관리합니다. 이 접근 방식은 개발 팀이 이러한 모든 작업을 Xcode에 의존하여 처리할 수 있고 로컬 Xcode 기기에서 앱을 릴리스할 수 있기 때문에 로컬 기기에서 개발 및 배포하는 데 적합합니다. 하지만 DevOps 및 지속적 배포의 현대 시대에는 로컬 Xcode를 사용하여 애플리케이션을 배포할 수 없습니다. 애플리케이션은 지속적 통합 서버에서 배포해야 합니다. 따라서 빌드를 자동화하고 코드 설계를 자세히 이해해야 할 필요성이 생깁니다.  
  
Apple에는 대부분의 내용을 설명하는 전체 코드 서명 프로세스에 대한 매우 포괄적인 문서가 있습니다. 이 게시물에서는 [애플 문서](https://developer.apple.com/library/archive/documentation/Security/Conceptual/CodeSigningGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40005929-CH1-SW1)를 활용하여 모든 코드 서명 활동에 대해 조금 더 실용적으로 만들려고 노력할 것입니다.

코드 서명은 앱 빌드가 완료된 후에 수행됩니다. bundles, resources, frameworks, tools, scripts, libraries, plugins, Info.plist files, assets 및 기타 모든 종류의 코드를 포함한 모든 애플리케이션은 앱의 개별 구성 요소와 함께 코드 서명을 받아야 합니다.

Xcode는 `codesign` 명령줄 도구를 사용하여 빌드한 앱에 서명합니다. Xcode 없이도 코드 서명 도구를 사용하여 앱에 서명할 수 있습니다. `codesign` 도구는 코드 서명을 생성하고 확인하는 용도로 사용됩니다. `codesign` 도구로 수행할 수 있는 다양한 옵션과 작업에 대한 자세한 정보를 얻으려면 다음을 참조하세요. 
[Apple의 설명서](https://developer.apple.com/documentation/technotes/tn3161-inside-code-signing-certificates/)를 확인하거나 터미널에서 명령어 `man codesign`을 입력하면 자세한 최신 정보를 확인할 수 있습니다.

이제 코드 서명 도구를 사용하여 다음 명령을 사용하여 예제 앱에 서명해 보겠습니다:

``` shell
$ codesign -s "iPhone Developer: Shashikant Jagtap (MY_TEAMID)" ~/Library/Developer/Xcode/DerivedData/iOSCodeSigning-gakpslthucwluoakkipsycrwhnze/Build/Products/Debug-iphonesimulator/iOSCodeSigning.app/
```

이 경우 앱은 이미 Xcode에 의해 서명되었으므로 다시 서명할 필요가 없습니다. 이제 코드 서명 ID를 사용하여 앱이 서명될 때 내부에서 어떤 일이 발생하는지 살펴보겠습니다.  
  
## 코드 서명 단계  
코드 서명은 macOS 보안 기술이며 iOS 앱 서명 시 다음과 같은 주요 단계가 있습니다.  
  
* 봉인 ( Seal )
* 디지털 서명( Digital Signature )
* 코드 요구 사항 ( Code Requirement )  

각 단계를 해당되는 경우 실제 예시와 함께 자세히 살펴보겠습니다.


##  봉인 ( Seal )

> 실생활에서 우리는 물건이 변하지 않도록 잠그기 위해 다양한 물건을 봉인합니다. 봉인된 봉투에 담긴 편지는 보내는 사람이 봉투를 봉인한 이후 편지의 내용이 변경되지 않았음을 보증하는 좋은 예입니다. 
> 받는 사람 역시 봉인이 풀리지 않는 한 보낸사람이 보낸 편지가 수정되거나 변경되지 않았음을 확신할 수 있습니다. 봉인은 내용물의 무결성을 보장한다는 의미입니다. 
> 마찬가지로, 코드 서명의 봉인은 코드의 다양한 부분에 대한 해시( hashes ) 컬렉션을 추가하므로 검증자는 코드가 서명된 이후 코드에 변경 사항이 있는지 감지할 수 있습니다. 
  
코드 서명 소프트웨어인 `codesign`의 경우 앱에서 코드의 다른 부분을 실행하여 봉인( Seal ) 을 생성합니다. 일종의 해싱 알고리즘( hashing algorithm )을 사용하여 코드의 모든 부분에 해시 ( hashes )를 적용합니다. 입력 블록에 적용되는 해시는 고유합니다. 그런 다음 검증자는 동일한 해싱 알고리즘을 사용하여 해시를 비교하고 모두 동일하면 검증 기준을 충족하는 것입니다. 작은 변화로 인해 해시가 손상될 수 있습니다. 해시는 프레임워크와 모든 종류의 코드에 적용됩니다. 속성 목록 형식으로. 일반적인 해시는 다음과 같습니다.


``` xhtml
<key>Frameworks/libswiftos.dylib</key>
    <dict>
        <key>hash</key>
        <data>
        dyKltMCMbq+pYDVJBtY78y7BuP0=
        </data>
        <key>hash2</key>
        <data>
        6DxNIVZgqWfOeWfedGQ1+wOnIuA7vQlU+gVA0WhCiRw=
        </data>
    </dict>
```
그런 다음 검증자는 동일한 해싱 알고리즘을 사용하여 데이터가 변경되지 않았는지 확인합니다. 그러나 이 검증은 저장된 해시의 신뢰성만큼만 신뢰할 수 있습니다. 디지털 서명은 서명 검증을 보장합니다.


## 디지털 서명( Digital Signature ) 
디지털 서명은 메시지의 진위 여부와 무결성을 검증하는 과정입니다. 비대칭 암호화( asymmetric cryptography )라고도 하는 공개 키 암호화를 기반으로 합니다. 코드 서명 과정에서 소프트웨어는 서명자의 개인 키를 사용하여 봉인( Seal )의 해시를 암호화하여 디지털 서명을 생성합니다. 서명자의 인증서와 함께 앱에 저장된 서명된 해시는 디지털 서명을 나타냅니다. 그런 다음 코드 서명 도구를 사용하여 다음 명령을 실행하여 코드 서명을 확인할 수 있습니다.


``` shell
$ codesign -v --verbose=5 ~/Library/Developer/Xcode/DerivedData/iOSCodeSigning-gakpslthucwluoakkipsycrwhnze/Build/Products/Debug-iphonesimulator/iOSCodeSigning.app/
```

다음과 같은 출력이 표시되어야 합니다:
![](http://shashikantjagtap.net/wp-content/uploads/2018/02/verify.png)

코드가 조금만 변경되어도 해시가 무효화되고 서명이 유효하지 않다고 보고합니다. 소프트웨어는 해시를 해독하기 위해 인증서에 포함된 해시와 서명자의 공개 키가 동일한 경우 동일한 세트를 사용합니다. 그런 다음 검증기는 해시를 비교하고 결과를 출력합니다.  
  
범용 코드의 디지털 서명은 앱 바이너리 자체에 저장되며, frameworks, bundles, tools 및 기타 코드의 서명은 번들 내의 `_CodeSignature/CodeResources`에 저장됩니다.


## 코드 요구 사항 ( Code Requirement )  
코드 서명을 평가하기 위한 몇 가지 규칙이 있습니다. 이러한 규칙을 코드 요구 사항이라고 합니다. 모든 앱은 앱에서 사용하는 모든 플러그인이 Apple에 의해 서명되어야 한다는 코드 요구 사항을 적용할 수 있습니다. 이렇게 하면 승인되지 않은 타사 소프트웨어가 기본 앱에 설치되는 것을 방지할 수 있습니다. 서명자는 코드 서명의 일부로 코드 요구 사항을 지정할 수도 있으며, 이러한 요구 사항을 내부 요구 사항이라고 합니다. 지정 요구 사항 또는 DR은 평가 시스템에 특정 코드를 식별하는 방법을 알려줍니다. 코드 요구 사항 언어에 대한 자세한 설명은 [Apple 웹사이트](https://developer.apple.com/library/archive/documentation/Security/Conceptual/CodeSigningGuide/RequirementLang/RequirementLang.html#//apple_ref/doc/uid/TP40005929-CH5-SW1)에서 확인할 수 있습니다.  
  
지금까지 코드 서명의 세 단계에 대해 간략하게 살펴보았습니다. 명령을 사용하여 코드 서명된 데모 앱의 자세한 정보를 인쇄해 보겠습니다.

``` shell
$ codesign -d --verbose iOSCodeSigning.app/
```

데모 앱의 모든 코드 디자인 세부 정보가 인쇄됩니다. 다음과 같이 표시됩니다:
![](http://shashikantjagtap.net/wp-content/uploads/2018/02/code_signing_all.png)


보시다시피 앱에 서명된 코드에 대한 많은 정보가 있습니다. 여기에는 실행 경로, 앱의 식별자 및 앱 서명이 표시됩니다. 개발 인증서와 디버그 빌드를 사용했으므로 Mach-O 씬 바이너리 형식을 보여줍니다. 14+7 해시와 13개의 규칙이 적용되어 있습니다. 현재로서는 코드 서명과 함께 지정된 내부 요구 사항이 없습니다.


# 자동 코드 서명 (Automatic Code Signing)
Apple은 WWDC 2016의 새로운 Xcode 앱 서명 세션에서 새로운 코드 서명 인프라를 발표했습니다. Apple은 Xcode 8 이상에서 iOS 개발자의 코드 서명 수고를 줄이기 위해 Xcode의 자동 코드 서명 기능을 발표했습니다. Xcode에서는 빌드 대상의 일반 탭에 자동 코드 서명을 활성화할 수 있는 새로운 설정이 있습니다. 개발 팀에 이를 알리기만 하면 인증서, 프로비저닝 프로파일 등과 같은 모든 코드 서명 작업을 Xcode가 자동으로 처리합니다.

![](http://shashikantjagtap.net/wp-content/uploads/2018/02/automatic_signing.png)

자동 서명을 사용하면 Xcode는 권한과 같은 앱 설정의 변경 사항, 새 기기를 프로필에 등록해야 하는 경우 등을 계속 주시합니다. 기기 등록 권한을 부여하거나 새로운 기능이 추가되면 Xcode는 새 프로비저닝 프로파일을 생성하고 보고서 탐색기 섹션에 코드 서명 업데이트를 표시합니다.

![](http://shashikantjagtap.net/wp-content/uploads/2018/02/Screen-Shot-2018-02-11-at-18.34.56.png)


Xcode 8에서는 코드 서명 프로세스를 관리하기 위해 두 가지 새로운 빌드 설정이 도입되었습니다. 빌드 설정 `DEVELOPMENT_TEAM`은 서명 ID를 보다 강력하게 제어하기 위해 사용되며, 필수 항목이며 비워 둘 수 없습니다. 두 번째 빌드 설정은 서명 방법의 유형을 나타내는 `PROVISINING_PROFILE_SPECIFIER`입니다. 이 빌드 설정은 자동 서명에만 적용되며, 수동 서명에서는 `PROVISIONING_PROFILE_SPECIFIER`가 사용되지 않습니다.`"iPhone Developer"`와 같은 일반 항목으로 `CODE_SIGN_IDENTITY`를 사용해야 합니다.  
  
자동 코드 서명에 대해 염두에 두어야 할 다른 중요한 사항은 다음과 같습니다.  
  
* `xcodebuild` 명령줄 도구를 통해 Xcode에서 생성된 아카이브는 처음에 개발 인증서로 서명됩니다.  
* `distribution` 방법으로 지정하면 아카이브는 `distribution` 코드 서명으로 다시 서명됩니다.  

즉, 지속적 통합 서버에서 자동 서명을 사용하는 경우 CI 서버에 개발 인증서뿐만 아니라 배포 인증서도 있어야 합니다.


# 수동 코드 서명  ( Manual Code Signing )
`manual Code Signing` 과정에서 빌드 설정에서 코드 서명 ID 및 프로비저닝 프로파일을 명시적으로 지정해야 합니다. 수동 코드 서명은 대상 설정의 일반 탭에서 "자동으로 서명 관리" 확인란을 선택 해제하여 Xcode 8+에서 활성화할 수 있습니다. 이렇게 하면 빌드 설정으로 이동하지 않고 일반 탭에서 코드 서명 ID 및 프로비저닝 프로파일을 설정할 수 있는 옵션이 제공됩니다.

![](http://shashikantjagtap.net/wp-content/uploads/2018/02/manual_Sining-Xcode8.png)


수동 서명 ( Manual Code Signing )을 위한 Xcode 설정에 대한 자세한 내용은 [Apple의 가이드](https://help.apple.com/xcode/mac/current/#/dev1bf96f17e)를 참조하세요. 수동 코드 서명에서 또 다른 변경 사항은 프로비저닝 프로파일이 더 이상 UDID 형태로 설정되지 않고 이름을 사용하여 설정할 수 있다는 것입니다. 다시 말씀드리지만 수동 코드 서명 방법을 사용할 때는 `CODE_SIGN_IDENTITY` 및 `PROVISIONING_PROFILE` 빌드 설정에 대한 값을 지정해야 합니다. 또한 앱을 빌드하는 로컬 머신에 필요한 인증서와 프로비저닝 프로파일이 있어야 합니다.  


# iOS 앱 재 서명 ( Re-Signing iOS Apps )  
매우 드물지만 서명된 앱의 서명을 제거하고 새 프로필 및 인증서로 다시 서명해야 하는 경우가 있습니다. 코드 서명 도구를 사용하면 가능합니다. iOS 앱에 다시 서명하려면 다음 단계를 수행해야 합니다.

* 재서명이 필요한 앱의 서명된 .ipa 파일을 찾습니다.  
* 압축을 풀고 코드 서명을 제거합니다.
```shell
$ unzip -q old.ipa
$ rm -rf Payload/*.app/_CodeSignature
```

* 새로운 프로비저닝 프로파일로 교체합니다.
```shell
cp new_provisining_profile.mobileprovision Payload/*.app/embedded.mobileprovision
```

* 이전 자격 ( entitlements) 을 이용하여 현재 앱에 대한 자격을 생성하빈다.
```
cd Payload/
$ codesign -d --entitlements - *.app > entitlements.plist
$ cd ..
$ mv Payload/entitlements.plist entitlements.plist
```

* 새 인증서와 자격( entitlements) 으로 앱에 강제 서명하기
```
$ /usr/bin/codesign -f -s "Your_certificate_in Keychain" '--entitlements' 'entitlements.plist'  Payload/*.app
```

* 다시 서명된 ipa 파일을 압축
```
$ zip -qr resigned.ipa Payload
```

이 단계에서는 새 인증서 및 프로비저닝 프로파일로 서명된 바이너리를 갖게 됩니다.  

### 작성 및 출처
* DeepL 번역
* 이 글은 2018년 2월 11일 Shashikant Jagtap가 작성한 글의 번역본입니다.
* [출처](http://shashikantjagtap.net/ios-code-signing-5-signing-ios-app/)

### 참고 사이트
* [iOS 코드 서명에 대해서(LINE Engineering)](https://engineering.linecorp.com/ko/blog/ios-code-signing)
* [애플 문서 About Code Signing](https://developer.apple.com/library/archive/documentation/Security/Conceptual/CodeSigningGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40005929-CH1-SW1)
* [TN3161: Inside Code Signing: Certificates](https://developer.apple.com/documentation/technotes/tn3161-inside-code-signing-certificates/)
* [비대칭 암호화 asymmetric cryptography](http://searchsecurity.techtarget.com/definition/asymmetric-cryptography).
* [digital signature](https://www.techtarget.com/searchsecurity/definition/digital-signature)
* [Code Signing Requirement Language](https://developer.apple.com/library/archive/documentation/Security/Conceptual/CodeSigningGuide/RequirementLang/RequirementLang.html#//apple_ref/doc/uid/TP40005929-CH5-SW1)
