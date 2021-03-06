# 논문 읽기
# Enhanced Android App-Repackaging Attack on In-Vehicle Network

## Abstract

connected car가 IoT 기술과 연결되었을 때의 취약점을 분석하고 실제 자동차와 상용앱을 사용한 실험을 통해 cyberattack의 가능성을 증명함.

## 1. Introduction

운전자와 탑승자의 안전과 편안함을 높이기 위해 다양한 전자제어장치(ECU)가 최신 차량에 탑재되고 있으며, 이러한 자동차-ICT(정보통신기술) 융합은 차세대 차량 개발의 새로운 패러다임이 되었음.
자동차 전자 부분이 증가함에 따라 차량 및 외부 데이터 서비스 간의 통신도 증가하고 있음.
대표적인 IoT 활용 사례 중 하나인 커넥티드카는 5세대(5G) 기술 개발로 빠르게 성장하고 있음.
기존 V2X 기술은 LTE나 5G를 이용해 C-V2X로 진화하고 있으며, 차량은 IoT 기술을 통해 외부 기기와 연결됨. 수십 년 동안 전형적인 기계였던 차량은 대형 IT 시스템으로 발전하고 있음. 그 결과, IT 전문가들은 이제 차량 개발에 참여할 수 있게 되었고 일반 대중들도 차량을 보다 쉽게 이해할 수 있게 됨.

그러나 자동차 전자제품의 개발로 차량은 해커들의 새로운 대상이 되었음. Lee 등은 ELM327 기반 CAN-to-Bluetooth 장치와 차량용 앱의 취약성을 분석한 사이버 공격 실험 결과를 발표함. 게다가, Samy Kamkar의 GM OnStar 해킹과 미하일 쿠진과 빅터 체비셰프의 차내 스마트폰의 취약성에 대한 연구에서 보듯이, 취약한 스마트폰 앱들이 차량의 새로운 공격 표면으로 떠오르고 있음.

본 연구에서는 [9]에서 수동으로 수행한 앱 분석 작업을 자동화하고 구글 플레이스토어에 등록된 213개 앱의 취약점을 분석함. 이 분석 결과를 바탕으로, 차량용 상용 앱 중 가장 인기 있는 앱 10가지를 repackaging하여 실제 차량을 통제하기 위한 공격을 수행한다. 마지막으로, 공격으로부터 보호하고 ELM327 기기의 통신 서비스 보안을 위한 화이트리스트에 기반한 접근 제어 장치를 개발하기 위한 보안 기술을 제안한다. 이는 자동차 제조업체가 CAN ID를 설정할 수 있는 경우, 모든 차량에 적용할 수 있는 범용 접근 제어 장치이다.


## 2. Background

#### 2.1 CAN(Controller Area Network)
ECU들간의 효율적인 통신을 지원하기 위해 BOSCH는 1980년대 초에 CAN을 개발했음.
CAN은 버스 네트워크 토폴로지를 지원하는 발신자 ID 기반 브로드캐스트 통신 기술이다. CAN은 point-to-point 통신의 단점을 해결함으로써 차량 내 통신 라인의 복잡성과 길이를 크게 줄였다. CAN을 사용하는 ECU 간의 데이터 전송 프로세스에서 전송ECU는 데이터를 전송하기 전에 CAN 데이터 프레임에 고유한 ID를 포함한다. 수신ECU는 브로드캐스트 데이터 프레임에 포함된 송신 ECU의 ID를 확인한 후 선택적으로 데이터를 수신할 수 있다. CAN 버스 시스템은 데이터 프레임의 ID 길이에 따라 다음과 같은 두 가지 유형으로 분류된다.

1) Standard CAN 2.0A(11-bit ID)
2) Extended CAN 2.0B(29-bit ID)

데이터 프레임은 SOF(start of frame), Arbitraion ID(Arbitraion), Control, Data, CRC(cyclic redundancy check), ACK(acknowledge) 및 EOF(end of frame) 필드로 구성된다. Standard CAN 2.0A 형식은 Arbitraion 비트로 11비트 ID를 사용하고, Extended CAN 2.0B 형식은 Arbitraion 비트로 29비트 ID를 사용한다.
CAN은 원래 closed network 상에서 design되었기 때문에 protection feature들(기밀성, 인증 및 접근제어)을 제공하지 않았음.
그러나 최근에는 커넥티드카 서비스가 상용화되면서 FOTA(Firmware Over-The-Air) 사용이 확산되고 차량 내 네트워크, 외부 네트워크가 상호 연결되고 있다. 그 결과, CAN은 일반 IT 환경에서와 같은 사이버 공격에 노출됨.

#### 2.2 Connected Car
Connected Car라는 말은 차량이 항상 외부와 연결되어 있음을 의미한다.

Connected Car는 다음과 같은 구성요소로 이루어진다.
- (i) ECU가 설치된 차량과 차량 내 네트워크가 구성된 차량
- (ii) 차량에 대한 다양한 서비스를 제공하기 위한 포털
- (iii) 차량을 포털과 연결하기 위한 통신 링크

이러한 환경에서 차량에는 여러개의 ECU가 설치되어 있으며, 이 ECU는 CAN 버스 시스템과 같은 차량 내 네트워크에 연결되어있음. 통신 링크는 텔레매틱스 ECU 또는 ELM327과 같은 무선 통신 장치를 사용하여 구성되고, 포털은 웹 기반 서비스와 스마트폰 애플리케이션 기반 서비스로 구분된다.

실험을 통해, 이 연구는 연결 차량 환경이 차내 네트워크의 취약성을 해결하지 않고 구성될 경우 차량이 심각한 위협에 노출된다는 것을 입증한다. 그런 다음 이 문제를 해결하기 위한 보안 메커니즘을 제안한다. 여기서 제안하는 공격 모델은 그림 1에 나와 있는 Connected Car 환경에 기초하여 설계되었다.

<img width="562" alt="image" src="https://user-images.githubusercontent.com/44834680/103730573-824d1080-5026-11eb-97c0-4062f513084e.png">

<img width="518" alt="image" src="https://user-images.githubusercontent.com/44834680/103731065-a1986d80-5027-11eb-8fda-c6b6711968b9.png">

#### 2.3 OBD Protocol
OBD(on-board diagnostics)는 자동차 상태를 진단하기 위해 사용되는 도구이다.
OBD는 원래 차량 배기 가스를 제어하기 위해 제작되었지만, 자동차 전자장치 개발이 가속화되면서 고장 목록을 점검하기 위해 개발됨. 이후에는 ISO 15765-4으로 제정되어 표준화된 단말기를 통해 자동차 진단장치로 차량 상태를 확인할 수 이게 되었음. 미국에서는 판매되는 모든 차량에 OBD-2 설치를 의무화 함.
OBD-II를 사용한 차량 상태 점검은 SAE J1989 표준에 정의된 점검 프로세스에 사용되는 요청/응답 방법과 OBD PID(OBD Parameter ID)를 통해 작동한다. 이 표준에 정의된 통신을 위한 CAN ID 및 데이터 프레임의 구조는 그림 2에 나와 있고 PID 코드와 파라미터는 표 1에 정의되어 있다.
OBD-II를 통해 얻은 차량의 진단 정보는 자동차 수리점에서 사용하는 전용 진단 기기뿐만 아니라 애프터마켓 공급업체에서 구입한 `스마트폰 애플리케이션` 및 `ELM327` 모듈을 통해 쉽게 얻을 수 있다. 따라서 악성사용자는 차량의 진단정보를 조작하여 차량을 제어할 수 있고 이에 따른 진단장치 및 관련 프로토콜에 대한 보안 정책이 필요하다.

<img width="555" alt="image" src="https://user-images.githubusercontent.com/44834680/103730614-98f36780-5026-11eb-8170-8d7c7bb1b4d0.png">

#### 2.4 ELM327
ELM327은 차량 상태를 진단하는 데 사용되는 차량 수리용 microcontroller이다.
블루투스 같은 무선 통신 모듈이 ELM327에 추가되면 운전자는 스마트폰을 ELM327에 연결하여 차량 상태를 실시간으로 확인할 수 있다. 스마트폰 및 ELM327을 사용하여 차량 상태를 확인하는 프로세스는 그림 3에 나와 있다.
<img width="450" alt="image" src="https://user-images.githubusercontent.com/44834680/103731547-bc1f1680-5028-11eb-96b3-3c9e73ebf757.png">


#### 2.5 Cyber Kill Chain
Lockheed Martin은 일반적으로 모든 사이버 공격에 적용할 수 있는 사이버 공격 프로세스를 제안했다.
그들은 사이버 공격 프로세스를 분석해 공격 단계별로 위협을 파악하고 공격 목적과 의도, 활동을 무력화해 조직의 복원력을 높이는 Cyber Kill Chain 방식도 제시했다. 사이버 킬 체인은 reconnaissance, weaponization, deliv- ery, exploitation, installation, command and control, and actions on objectives의 7단계로 구성된다. 각 단계의 특성은 표 2에 설명되어 있다.

<img width="387" alt="image" src="https://user-images.githubusercontent.com/44834680/103731696-22a43480-5029-11eb-9e1c-37e42a8faed2.png">


## 3. Proposed Attack Model

여기서 제안하는 공격 모델은 기존 연구보다 더 현실적이며 두 가지 위협에 기초하여 설계되었다. 첫 번째 위협은 차량용 스마트폰 앱이 쉽게 위조/repackaged 및 재배포될 수 있다는 것이다. 두 번째 위협은 공격자가 언제든지 repackaging된 악성 스마트폰 앱을 사용하여 무선으로 공격할 수 있다는 것이다. 
제안된 공격 모델은 공격자, 대상 차량 및 공격 대상자로 구성된다. 모든 개체의 특성이 수집되어 하나의 공격 모델을 형성한다. Cyber intrusion kill chain을 이용하여 제안된 공격 모델을 분석하였다.

#### 3.1 Attacker Ability

공격자는 OBD device를 사용하여 차량에 장착된 특정 ECU를 구동할 수 있는 CAN data frame을 얻을 수 있다.
또한 공격자는 앱스토어에서 배포/판매되는 차량 앱을 다운받고 원하는 형식으로 reapackaging할 수 있다.
그런 다음 공격자는 악의적인 앱을 사용하여 차내 CAN에 원하는 CAN data frame을 inject할 수 있다. 공격자가 repackaging한 악의적인 앱은 third-party market을 통해 배포가 가능하다.


#### 3.2 Target Vehicle
Target Vehicle은 그림 1에 표시된 연결 차량 환경을 이용하는 것으로 가정한다. 대상 차량은 CAN을 사용하여 ECU 간의 통신을 용이하게 한다. 그러나, CAN 버스 시스템이 communicated data frame에 대한 인증을 보장하지 않기 때문에, CAN data frame이 재전송 공격에 사용될 수 있다.

#### 3.3 Victim Behavior

target vehicle의 운전자(피해자)가 플레이스토어에서 스마트폰으로 차량 진단 앱을 다운로드하고 차량을 운전하는 동안 다양한 서비스를 받는다. 운전자(피해자)가 다운로드한 차량 진단 앱은 공격자가 플레이스토어를 통해 배포한 악성 앱으로 간주된다. 악의적인 앱을 사용하는 운전자(피해자)는 악의적인 앱(특정 ECU를 제어할 수 있는 data frame injection)의 공격 동작을 인지할 수 없다.

#### 3.4 Attack Model

차량 진단 앱을 다운로드한 target vehicle의 운저자는 차량을 운전하는 동안 다양한 서비스를 제공받을 수 있다. 공격자는 이러한 서비스 상황을 악용하여 지정되지 않은 다수의 차량에 사이버 공격을 행할 수 있다. 그림1의 연결된 차량은 OBD-2 단자에 무선 통신 모듈(ELM327)이 설치된 환경을 나타낸다. 이 단자를 통해 차량은 차내 CAN에 연결할 수 있으며, 스마트폰 앱은 차량 작동 중에 사용자 스마트폰과 ELM327모듈의 패킹을 통해 차내 CAN과 항상 상호 연결됨. 공격 모델 프로세스는 공격 준비 단계와 공격 수행 단계로 구분됨. 공격 준비 단계 및 공격 수행 단계를 포함한 전체 프로세스는 다음과 같다.

<img width="373" alt="image" src="https://user-images.githubusercontent.com/44834680/103740283-4c198c00-503a-11eb-862e-44cbcf4db90c.png">


```
(1) 공격자는 차량용 무선 통신 모듈의 통신 취약성을 분석한다(ELM327 기준).
(2) 공격자는 ELM327 기반의 차량용 무선 통신 모듈을 사용하여 플레이스토어에서 차량용 앱을 다운로드한다.
(3) 공격자는 (1)단계의 분석결과를 이용하여 악성코드를 생성한다.
(4) 공격자는 차량용 다운로드 앱에 악성코드를 삽입하고 repackaging한다.
(5) 공격자는 플레이스토어(타사 앱 마켓 포함)를 통해 repackaging된 악성 앱을 배포한다.
(6) 피해자는 자신의 차량 OBD-II 단말기에 ELM327 기반 무선 통신 모듈을 설치한다.
(7) 피해자가 차량용 앱을 앱 마켓(또는 타사 시장)에서 스마트폰으로 다운로드하여 설치한다. 다운로드한 앱은 공격자가 배포하는 악의적인 앱으로 가정한다.
(8) 피해자가 악성 앱을 실행해 자신의 차량을 운전한다.
(9) 차량이 동작하는 즉시 악성 앱은 악성 행위를 한다.
```

공격 모델은 위에 설명된 총 9단계로 구성된다. 표 3은 사이버 킬 체인을 이용한 제안된 공격 모델의 분석을 보여준다.

## 4. Attack Experiment

이 장에서는 공격자가 재패키지한 앱을 사용하여 차량의 강제 제어 가능성을 입증한다. Attack Experiment는 준비 단계와 실제 공격 단계의 두 단계로 구성된다. 준비 단계와 실제 공격 단계는 각각 두 단계로 나뉜다.

(1) Preparation 단계 1: CAN과 블루투스 모듈의 통신 프로토콜 분석(ELM327 모듈)

(2) Preparation 단계 2 : 차량 적용 취약성 분석 및 악성 차량 적용의 생산

(3) Actual attack 단계 1: LABCAR 기반 공격 실험.

(4) Actual attack 단계 2: 실제 자동차 기반 공격 실험.


마지막으로, 구글스토어에서 판매되는 ELM327과 관련된 모든 앱에 대해 위험 평가를 수행.
리스크 평가를 위해 자동화된 분석 툴을 제작함.

#### 4.1 Preparation Phase(Analysis)

이 단계에서는 ELM327모듈과 차량 진단 앱의 취약점 분석. 안드로이드 앱 마켓에 있는 모든 앱들을 분석하고, 리패키징 공격의 위험을 분석함.

(A) Analysis of Communication Protocol between Vehicle Diagnosis App and ELM327
ELM327 모듈은 차량의 OBD-2 포트에 장착되며 요청-응답 방법에 따라 차량 상태 정보를 차량 진단 앱으로 전달함.
ELM327모듈은 일반적으로 요청 메시지를 보낼 때 고정된 CAN ID를 사용함. 단,특수한 경우에는 ELM Electronics에서 제공하는 AT 명령을 사용하여 요청 메시지의 CAN ID와 데이터를 변경할 수 있다. 차량을 제어하는 데 사용할 수 있는 AT 명령은 표 4에 나와 있음.
단, CAN ID와 데이터가 변경되더라도 AT 명령과 ELM327의 통신 프로토콜을 따를 경우 차량은 모든 데이터를 정상 데이터로 인식함. 즉, 사용자가 원하는 CAN 데이터를 그림 4와 같이 ELM327로 전송하는 경우 차량에 동일한 정보가 수신됨.
ELM327 기반의 모든 차량 진단 앱은 OBD PID를 사용하여 차량 상태 정보를 획득함. 따라서 OBD PID 코드는 차량 진단 앱을 분해 및 분석하여 쉽게 찾을 수 있다. 다음 섹션에서는 상용 진단 앱에서 AT 명령과 OBD PID 정보를 사용하여 취약점을 찾는 방법과 이를 악성 앱으로 리패키징하는 방법에 대해 논의한다.
<img width="603" alt="image" src="https://user-images.githubusercontent.com/44834680/103753016-d5868980-504d-11eb-9bba-1c18e7262ecb.png">


(B) Analyzing and Repackaging Vehicle Diagnosis App

취약한 진단 앱은 ELM327 AT 명령어와 OBD PID이 평문으로 노출되어있음.
공격자가 해당 앱을 디컴파일해서 문자열 검색만으로도 두가지 정보를 얻을 수 있음.
여기서는 searching attack과 inserting attack 방법이 가능.
그리고 이 방법을 사용해서 악성 앱을 만들 수 있음.

차량진단 앱은 차량 상태를 체크할 때 OBD PID를 씀. 만약에 공격자가 앱을 디컴파일해서 수신위치를 알아낼 수 있다면, 그 위치가 공격 포인트가 됨. 

<img width="820" alt="image" src="https://user-images.githubusercontent.com/44834680/103849596-79bd0e80-50e8-11eb-968c-6e73b67c2970.png">


공격 툴 두 가지
- 차량 진단 앱이 시작될 때 공격 명령 전달, AT 명령을 사용하여 ELM327 초기화하는 공격. 공격자는 AT 명령을 원하는 차량 제어 명령으로 바꾸고 리패키징할 수 있음.
- 차량의 특정 조건에서 공격 명령을 전달.

차량 진단 앱은 OBD PID를 사용해서 차량 상태를 확인함. 따라서 공격자는 위에서 언급한 분석 프로세스에서 OBD ID로 리패키징하는 방법을 사용하여 원하는 명령을 삽입 가능.


#### 4.2 Actual Attack Experiment

실제 공격 실험은 사이버 킬 체인의 7단계 중 4~7단계에 해당.


<img width="737" alt="image" src="https://user-images.githubusercontent.com/44834680/103866452-2c04ce00-5109-11eb-8f1c-f6bf0d1b4d3a.png">

[실험 영상]
(https://sites.google.com/team-aegis.com/webpage/publish_data)


## Conclusion

이러한 공격에 대한 대책으로 앱 난독화 및 화이트리스트 기반 방화벽이 제안되었음. 본 연구에서 사용된 공격 방법에는 현재 판매 또는 유통되고 있는 상용 제품이 사용되었다. 누구나 안드로이드 앱을 개발하고 분석하는 능력이 있다면 리패키징할 수 있기 때문에 현실과 위험 측면에서 파급력이 큼. 5G가 확장될수록 차량과 외부 장치 간의 연결성이 증가하므로 사이버 보안의 위험이 커진다. 따라서 현실적인 위협에 대한 보안을 확보하기 위해서는 다단계의 다면적인 보안 조치가 필요.
본 연구에서 제안된 방화벽의 경우 화이트리스트에 따라 작동하는 별도의 장치가 애프터마켓 장치에 연결됨. 그러나 차량 자체에 대한 위협에 대응하기 위해 외부 장치에 현실적으로 의존할 수 없기 때문에 자동차 제조업체는 OBD-II 인터페이스에 보안 기능을 설치해야 함! 

OBD-II 기기와 스마트폰 앱 간 통신에 블루투스 대신 와이파이를 사용하거나 ELM327 대신 다른 동글을 사용하거나 Android 대신 iOS를 사용하는 것은 "reconnaissance", "identification" 또는 "vulnerability" 단계에 있기 때문에 명령 및 제어 단계의 보안 조치는 여전히 유효함. 

"Actions on Objects" 단계에서는 IDS를 사용하여 공격을 탐지하고 방어할 수 있음. 이 주제에 대한 연구는 향후 연구를 위해 남겨두기로..
