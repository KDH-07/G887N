# SM-G887N / CVH2

Samsung Galaxy A9 Pro **SM-G887N**, Android 10 **G887NKSU2CVH2**에서 정상 부팅과 실제 root 명령 실행을 확인한 부트 이미지입니다.

## 다운로드

`AP_SM-G887N_G887NKSU2CVH2_Magisk-v30.7_NSGuard_v0.1.0.tar`와 `SHA256SUMS.txt`를 내려받습니다. tar는 **Odin AP 슬롯용 boot-only 파일**이며, 전체 펌웨어가 아닙니다. 안에는 `boot.img` 하나만 있습니다.

## 지원 범위와 검증 한계

- 모델: **SM-G887N**만 지원 대상으로 합니다. 다른 A9/A8s 모델에 사용하지 마세요.
- 기존 펌웨어: **G887NKSU2CVH2 / Android 10**. 다른 빌드나 다른 버전의 boot와 혼용하는 설치는 검증하지 않았습니다.
- 부트로더가 실제로 해제돼 있어야 합니다. CSC 변경만으로 해제되지는 않습니다.
- 검증 기기의 CSC는 KOO입니다. KTC 상태에서 이 최종 파일의 성공은 따로 검증하지 않았습니다.
- 검증 기기는 앞선 시험 중 데이터를 초기화하고 변경된 boot로 암호화 데이터 환경을 구성한 상태였습니다. **새 순정 기기에서 데이터 보존을 포함한 최초 설치 과정은 검증하지 않았습니다.**
- 확인한 결과는 정상 부팅, Magisk 30.7 / 30700, root의 magiskd 및 ADB Shell의 `su -c id` 실행입니다. 장시간 안정성, 통화·모바일 데이터, 모든 Knox 기능 및 업데이트 호환성은 별도 검증하지 않았습니다.

## Odin 펌웨어 업로드

아래는 시험 기기에서 성공한 boot 교체 방식입니다. 모든 순정 기기의 최초 루팅 절차로 보장하는 안내가 아닙니다.

1.데이터를 백업하고 현재 펌웨어와 같은 빌드의 순정 복구 파일을 확보합니다. Knox 변경과 암호화 키의 무결성 조건 변경으로 데이터 초기화가 필요해질 수 있습니다.
2.모델·빌드 및 실제 부트로더 해제 상태를 확인하고 다운로드 모드로 들어갑니다.
3.Odin을 Reset한 뒤 **AP에 위 tar 하나만** 선택합니다. BL / CP / CSC / USERDATA는 비워 둡니다.
4.시험에서는 **Auto Reboot / F.Reset Time**만 켰습니다. **Re-Partition / Nand Erase / Flash Lock**은 끕니다.
5.플래싱 후 부팅을 확인합니다. 파일을 공식 Magisk로 다시 패치하는 과정은 거치지 않습니다.
6.Magisk 앱이 간이 앱으로만 설치되면 공식 https://github.com/topjohnwu/Magisk/releases/tag/v30.7 앱을 설치합니다.
7.ADB로 확인하려면 USB 디버깅을 허용하고 Magisk 슈퍼유저의 Shell / sharedUID Shell 권한을 허용한 뒤 실행합니다.

```text
adb shell su -c id
uid=0(root) gid=0(root) groups=0(root) context=u:r:magisk:s0
```

## 일반 Magisk 패치본과의 차이

1. **Samsung SignerVer02 구조 보존** 공식 Magisk 패치 후 없어진 순정 boot 끝의 512바이트 구조를 보존했습니다. 수정된 내용에 대한 유효한 삼성 서명을 새로 생성한 것은 아닙니다. 이 기기에서 구조 복원 후 Odin의 modem.bin Auth 실패가 없어지는 것을 관찰했습니다.
2. **첫 단계 init 경로**: Magisk 30.7 ramdisk에 빈 `sdcard` 일반 파일(mode 0600)을 추가해 Magisk에 존재하는 first-stage fallback 경로를 선택합니다. 순정 init 백업은 그대로입니다.
3. **커널의 실행 파일 마운트 검사 수정**: CVH2 raw kernel의 `0x1cc3c8`에서 `e0010034`를 `0f000014`로 변경합니다. `cbz w0, #0x1cc404`를 `b #0x1cc404`로 바꿔 해당 검사 블록을 건너뜁니다. 실제 실패 로그의 `KDP_NS_PROT: Illegal Execution of file #/data/magiskinit#` 및 `flush_old_exec+0x4b8` 오류로 인한 수정입니다.
