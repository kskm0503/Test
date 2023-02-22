### Stratis File System? [인터넷 참조]
Stratis는 새로운 로컬 스토리지 관리 솔루션으로, pool 이라는 개념을 이용하여 스토리지를 구성한다.
기존에 알던 xfs, ext4 같은 경우는 물리 디스크 디바이스를 바로 파일 시스템으로 구성했었다.
하지만 Stratis는 pool 이라는 개념을 사용하여 사용자가 요청하는 파일 시스템을 pool에서 thin provisoning한다.
즉, 파일 시스템을 10GB로 생성한다고 해서 실제로 10GB할당 되는 것이 아니다.
실제로 사용 중인 데이터가 3GB라면 pool 에서는 3GB만 해당 파일 시스템에 할당한다. 실사용량에 따라 가변적인 할당이 가능해서 리소스의 낭비를 최소화 하여 효율적으로 사용할 수 있다. **[아래그림 참조]**

![Stratis 설명](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb0had2%2FbtrCKX06JOk%2FtpyIvppXR9k1k0NqKCwZRK%2Fimg.png)

위의 사진처럼 디스크 디바이스를 pool 단위로 맵핑하고, 파일 시스템은 pool 에서 프로비저닝 된다.


-----------------------------------------------------------------------------------------------------------------------
**실습 진행**

1. 패키지 설치

   패키지 설명 
   
   Stratis-cli: This is the command-line tool that ships with Stratis.
   
   Stratisd daemon: This is a daemon that creates and manages block devices and plays a role in providing a DBUS API.
   
   실습 : dns install stratisd stratis-cli / systemctl enable --now stratisd
   
2. Pool 생성하기(my_pool로 생성)

   실습 : stratis pool create my_pool /dev/nvme0n2
   
   확인 : stratis pool list
   
   참고 : 만약 블록 디바이스를 해당 Pool에 추가하고싶으면 add-data 명령어를 사용한다.
   
          stratis pool create my_pool /dev/nvme0n3 -> Pool에 연결된 블록 디바이스 정보는 아래 방법으로 확인 할 수 있다
          
          stratis blockdev
          
3. Stratis file system 생성하기    

   실습 : stratis filesystem create my_pool data_ST / stratis filesystem list
          
4. 마운트 진행

   실습 : mount /dev/stratis/my_pool/data_ST /data_ST
   
   확인 : df -hT로 확인
   
----------------------------------------------------------------------------------------------------------------------------------------


**참고사항 및 보완할 사항**

파일 시스템이 Thin 하게 저장공간을 할당 받는 것을 확인해 보자!
먼저 처음에 생성한 상태에서 data_ST의 USED 는 546MiB이다.
이 파일시스템이 마운트 된 /data_ST 디렉터리에 1GB 짜리 파일을 생성했다.
그 후에 다시 확인했을 때 filesystem1의 USED size가 약 1.6GIB로 커진 것을 볼 수 있다.
          
최초 546MB  / 그 이후 마운트를 진행하니 1TB로 자동지정됨 / 그리고 7.2G 사용중이다?

이 부분에 관해서 확인필요
   
   
