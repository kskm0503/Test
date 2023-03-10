#### 1. fdisk 명령어로 파티션을 생성하는 것과 pvcreate의 차이점이 무엇인지?


**1) fdisk/dev/sdb 해서 Primary나 Extended 파티션을 만드는 경우**

      작업을 진행하면서 fdisk 파티션을 생성할 때 참고 사항이 있음
   
      최대 4개의 Primary 파티션을 생성하는 경우 or 3개의 Primary 파티션 / 1개의 Extended 파티션을 생성하는 경우
   
      파티션을 생성 하면 /dev/sdb1 /dev/sdb2 /dev/sdb3 /dev/sdb4 이렇게 4개의 파티션을 생성하고
   
      1) pvcreate -> vgcreate -> lvcreate > mkfs(파일시스템 지정) > mount > fstab 로 등록하면 각각 4개의 파티션을 마운트 가능하다
      2) 다만 이렇게 진행하는 경우 /dev/sdc 같이 다른 디스크를 같이 묶어서 진행 할 수 없다
         이럴때를 위해 lvm이 존재한다


**2) lvm 으로 파티션을 생성하는 경우**
    
     lvm으로 파티션을 생성하는 경우 ->  만약 여러분에게 /dev/sdb1 1G /dev/sdc1 1G /dev/sdd1 1G 가 있다고 치자
     
     각각 개별로 마운트를 해서 진행하는 경우 1G / 1G / 1G 씩 밖에 못쓴다
     
     하지만 3개를 같이 묶어서 마운트를 진행한다면 3G를 한번에 쓸 수 있다. 위에서 1번방식으로 실습 진행시 
     
     각각 파티션을 1G/1G/1G로 쓸 수밖에 없는데 LVM 방식을 쓰면 3G로 쓸수있다
    
-------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **LVM 기초 설명(인터넷 설명 참조)**
  
  LVM이란
  LVM(Logical Volume Manager)는 리눅스의 저장 공간을 효율적이고 유연하게 관리하기 위한 커널의 한 부분이다.

LVM vs. 일반 disk partitioning
LVM이 아닌 기존 방식의 경우, 하드 디스크를 파티셔닝 한 후 OS 영역에 마운트하여 read/wirte를 수행했다.
이 경우 저장 공간의 크기가 고정되어서 증설/축소가 어렵다. 이를 보완하기 위한 방법으로 LVM을 구성할 수 있다.
LVM은 파티션 대신에 volume이라는 단위로 저장 장치를 다룬다.
스토리지의 확장,변경에 유연하며, 크기를 변경할 때 기존 데이터의 이전이 필요 없다.

![LVM 설명](https://github.com/kskm0503/Test/blob/main/%EB%8B%A4%EC%9A%B4%EB%A1%9C%EB%93%9C.png)

물리적 볼륨 / PV (Physical Volume)
- 실제 디스크 장치를 분할한 파티션된 상태를의미한다.
- PV는 일정한 크기의 PE들로 구성된다.

물리적 확장 / PE (Physical Extent)
- PV를 구성하는 일정한 크기의 Block.
- 보통 1PE는 4MB에 해당한다.
- PE와 LE는 1:1로 대응한다.

볼륨 그룹 / VG (Volume Group)
- PV들이 모여서 생성되는 단위이다. (모든걸 합친 거대한 지점토 덩어리의 느낌이다)
- 사용자는 VG를 원하는대로 쪼개서 LV로 만들게 된다.

논리적 볼륨 / LV (Logical Volume)
- 사용자가 최종적으로 사용하는 단위로, VG에서 필요한 크기로 할당받아 LV를 생성한다.
-------------------------------------------------------------------------------------------------------------------------------------------------------------------
**실습 순서(디스크는 /dev/sdb로 실습 진행)** 

1. 물리적 볼륨 생성

   설명: PV 생성
   
   실습 : pvcreate /dev/sdb
   
   확인 : PVS 및 pvdispaly로 확인
   
2. 볼륨 그룹 생성

   설명 : VG 생성
   
         vgcreate [vg이름] [블록스토리지 경로] 
         
   실습 : vgcreate Test_vg /dev/sdb
   
   확인 : vgs 및 vgsdisplay로 확인

3. 로지컬 볼륨 생성(LV)

   설명 : LV 생성
   
         lvcreate -n [LV이름] -L [LV용량] [VG이름]
         
   실습 : lvcreate -n DATA_LVM -L 10G Test_vg
   
   확인 : lvs 및 lvsdisplay로 확인
   
4. 파일시스템 생성   
   설명 : 파일시스템 생성 및 포맷
   
   실습 : mkfs.ext4 /dev/Test_vg/DATA_LVM
   
   
5. 마운트 진행
 
   설명 : LV를 실제로 사용할 수 있도록 파일 시스템과 디렉터리 영역을 연결하도록 한다 **사전에 마운트 할 디렉터리 미리 생성**
   
   실습 : mount /dev/Test_vg/DATA_LVM 
   
6. Fstab 등록   

   설명 : 재부팅시에도 마운트 정보를 유지하려면 fstab에 정보를 등록해야함
   
         [파일_시스템_장치]  [마운트_포인트]  [파일_시스템_종류] [옵션] [덤프] [파일체크_옵션]
         
         [첫번째 필드] 파일 시스템 장치 : 파일 시스템의 장치명을 설정하는 부분이다. 마운트 가능한 장치명를 적는다.
         
         [두번째 필드] 마운트 포인트 : 파일 시스템이 마운트 될 위치를 설정하는 항목이다. 주로 어디 디렉터리에 마운트 될지를 지정한다.
         
         [세번째 필드] 파일 시스템 종류 : 마운트 될 파일 시스템의 파일 시스템 종류를 설정한다.
         
                                        파일 시스템 종류	설명
                                        
                                        ext	초기 리눅스에서 사용되었던 fs-type으로 지금은 사용하고 있지 않다
                                        
                                        ext2	지금도 사용하고 있는 fs-type으로 긴 파일명을 지원한다.
                                        
                                        ext3	저널링 파일 시스템으로 ext2 에 비교해 파일 시스템 복구 기능 및 보안 기능을 향상시켰다.
                                        
                                        xt4	ext3 다음 버전의 리눅스 표준 파일 시스템으로 16TB까지만 지원하던 ext3 보다 훨씬 큰 용량을 지원한다.
                                        
         [네번째 필드] 옵션 : 파일 시스템의 용도에 맞게 파일 시스템 속성을 설정하는 옵션 항목이다.
         
                             옵션	설명
                             
                             defaults	rw, nouser, auto, exec, suid 속성을 모두 가지며, 일반적인 파일 시스템에서 사용되는 속성이다
                             
                             auto	부팅시 자동 마운트 가능하도록 한다
                             
                             noauto	부팅시 자동 마운트가 되지 않도록 한다
                             
                             exec	실행파일이 실행되는 것을 허용한다
                             
                             noexec	실행파일이 실행되지 않도록 한다
                             
                             suid	SetUID와 SetGID의 사용을 허용한다
                             
                             nosuid	SetUID와 SetGID의 사용을 허용하지 않는다
                             
                             ro	read only, 읽기 전용으로 마운트한다
                             
                             rw	read write, 읽기, 쓰기 모두 가능하도록 마운트한다
                             
                             user	일반 계정 사용자들도 모두 마운트할 수 있다
                             
                             nouser	일반 계정 사용자들은 모두 마운트 할 수 없다
                             
                             usrquota	개별 계정 사용자의 디스크 용량을 제한하기 위해 Quota를 설정한다
                             
                             grpquota	그룹 별로 Quota 용량을 설정한다
                             
       [다섯번째 필드] dump : 0 이나 1로 설정하고, 1은 dump가 가능한 백업 가능한 파일 시스템이고 0은 백업 하지 않는다.
       
       [여섯번째 필드] 파일체크 옵션 : 루트 파일 시스템을 점검할때 사용하고 , 0, 1, 2 로 설정한다
       
                                     0 : 부팅시 파일 시스템 점검하지 않음
                                     
                                     1 : 루트 파일 시스템으로 부팅시 파일 시스템을 점검한다
                                     
                                     2 : 루트 파일 시스템 이외의 파일시스템으로서 부팅시 파일 시스템을 점검한다.
  
   실습 : vi /etc/fstab
   
         /dev/Test_vg/DATA_LVM /DATA_LVM ext4 /DATA_LVM defaults 0 0
         
         or UUID 값으로 진행(blkid로 확인)
         
         UUID="--------------------------------" /DATA_LVM ext4 defaults 0 0 
         
