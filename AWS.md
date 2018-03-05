- 계정 통합: 회사 또는 지불 조직 별 관리
- IAM: 계정 별 Rule 및 정책 설정
- AWS Directory Service: AD 관리 및 기존 AD 연결 Connector (Direct Connect)
- AWS Config: TAG 부여한 환경 별 인스턴스가 표준준수했는지 확인

VPC는 가상 사설망
- 고성능 컴퓨팅 또는 소규모 단일 Application에 적합
- Private Network는 Outbound는 NAT가 반드시 있어야 하고 Internet G/W를 통해서 밖에있는 AWS 리소스에 접근해야 한다.
- 단, S3는 별도 Private to S3 네트워크 설정이 있음
- EC2 선택시 네트워크 대역폭 결정도 중요
- 배치 그룹: 노드 간 성능 향상(배치 그룹은 가용영역을 확장하지 않음)
- VPC내 두 EC2 사이에서 점보 프레임을 사용하면 패킷 별 페이로드 크기를 늘릴 수 있음
- 고성능 컴퓨팅: 연산이 많고 복잡한 처리
- HPC 애플리케이션 분류:
  > 느슨하게 결합된 그리드 컴퓨팅
  > Amazon EC2 Spot Instance(내가 얼마만큼만 쓰게끔 설정) 또는 Auto Scaling에 이상적
- 정적연결은 모든 경로(IP 접두사)가 지정 필요
- 동적 VPN(BGP)는 최대 100개의 접두사 지원
  > BGP 네크워크: 이웃간 라우팅연결
- VPC에서 나가는 네트워크는 비용 발생

Direct Connect
- AWS Direct Connect Router

Spot Instance : 쓰다가 리소스 넘으면 20분 전에 EC2메타데이터 쪽에 공지

Cloud Formation
- Stack 관리: 환경 설정 관리(Template) 및 자동화 도구
- EC2 설치 시점에 자동화 도구 제공
- sh 파일에 환경초기화 Java 설치 응용 환경 구성 및 실행 까지 가능
- 인프라 변경사항을 소스코드 관리 수준으로 제공

Beanstalk: 응용 서버 자동 구성
- ELB, Auto-Scaling Group, Security Group, Database 등 선택
  : Log, Monitoring 등 통합 설정 --> Route53으로 Beanstalk 테스트 도메인 제공
- 블루/그린 배포를 사용하면 서비스 중단 없이 동적 배포 가능
- 설정: 환경 선택 -> .ebextensions: 초기화 -> 구성
- 내부적으로 Cloud Formation 동작함
- 초기 규모 이후 테스트에 따라 점진적 증가 가능

AWS OpsWorks
- Chef의 AWS 형태의 제공
- 기본 Receipes 및 Cookbook 제공: 스크립트 작성 필요
- AWS 서비스들도 함께 초기화 가능
- Stack - 계층 정의 - 인스턴스 할당

Docker
- 서버 및 OS 가상화 스크립트 --> 재사용
- 하나의 서버에 여러개의 컨테이너를 만들어서 각 컨테이너에 탑재되는 응용 프로그램의 용량을 완전하게 사용하게 함

AWS 아키텍처 센터
- 주요 구성 아키텍트에 대한 가이드
- 아키텍트 그릴때 쓸 수 있는 아이콘 제공

글로벌 서비스
- Route53, CloudFront, IAM

AWS Drive
- 이미지 사이즈 별 제공

Amazon ElastiCache
- 클라우드에서 인 메모리 데이터 스토어 또는 캐시를 손쉽게 배포, 운영 및 확장할 수 있게 해주는 웹 서비스
- 제공 오픈소스 서비스: Memcached, Redis
- RDS --- ElasticCache --- Proxy(Auto Sacling - 최소2개, 최대2개 - HA 구성을 위함) --- Application
- Replication을 위한 대역폭도 인스턴스 규모에 꼭 고려를 해야 한다.

AWS Storage Gateway
- S3기반 블록 스토리지: 스냅샷, 캐싱, 가상테이브 디바이스 형태로 제공
- 실제 디스크 공간 크기과 스토리지 볼륨 크기의 일치 필요
- 각각의 게이트웨이는 최대 32개 저장 볼륨을 붙일 수 있다.

AWS 데이터 고려
- 정적인가?
- 캐싱해야하나?
- 부하가 얼마인가?
- 파일의 실행방식이 어렇게 되나?

Auto Scaling 솔루션: Scryer
- 반응형 확장 외에 "예측 가능한 확장"을 제공
- 가동중단이후 수많은 사용자가 동시에 액세스 시도: 재시도 폭풍
- 노이즈 까지 예측해서 정상적인 상황대로 Scaling 진행

T2 인스턴스
. CPU 크레딧: 기준성능이 있고 안쓸때는 크레딧을 쌓다가 성능이 필요한 상황에 크레딧을 소진 - 다 쓰면 다시 기분 성능으로 다운
  > 초기 크레딧은 부팅 시점에 많이 소진 (서버를 끄면 크레딧은 리셋! 크레딧은 24시간이후 리셋)
- 특징: 즉각적인 확장 기능(버스팅), 100% 까지 vCPU 확장
- EBS SSD(GP2): 즉각적인 확장(대기시간 없음)

마이크로 서비스
- 각각의 서비스가 자체적인 '티어': 따로 처리되고 독립적인 확장
- 언어에 구애받지 않고 각 서비스가 API 표준만 수립하고 연계
- 분산 트랜잭션
  . MSA는 내 작업이 끝나면 작업을 다른 서비스에 넘기는 구조
  . 따라서 Pub/Sub 관계의 데이터 전달 과정일 중계해야 한다.
  . MQ 등 도구를 이해서 데이터 생산-가공 및 소비의 확장 및 분산을 설계할 수 있다.
- 따라서 각 마이크로 서비스의 모든 코드를 동일한 수준의 성숙도를 유지
  . 소스 및 구조 변경이 필요할 경우 기존 서비스를 수정하지 말고 새로 만들어라..
  . 상태 비저장 모드로 처리할 것: 개별적인 상태에 초점을 맞추지 마라
  . 비동기 방식의 처리를 기본으로 하라

Blue-Green 배포
. 다운타임 없이 새 버전 전환
. 모든 사용자를 전환하기 전에 새 버전이 올바른지 확인: 한꺼번에 사용자의 서비스를 옮기지 않고 비중을 배분
. 블루: 구버전, 그린: 신버전
. Route53의 엔드포인트를 조절 (DNS Cache의 TTL을 1분 이내로 조절)
. ASG + Route53 + CloudFormation
. 데이터 변경사항은 어쩔 수 없음...

FS
- 내부적으로만 쓰일 수 있는 File System
- AZ지정 - Subnet 지정 - NFS 포트 오픈

AWS Kinesis : AWS에서 사용하는 Flume 같은 것

AWS WAS(Web Application Firewall)
. DDOS: 슬로우로리스 (클라이언트 Stream Write 자체를 엄청나게 느리게 함으로서 Read Timeout 기회를 뺏는 것)
. Application 앞단 Firewall: 규칙을 정하고 방화벽처럼 동작하게 가능
. ELB를 쓴다면 기본적인 보안 공격을 완화할 수 있는 방법들을 제공
. WAF와 CloudFront를 조합하여 어지간한 공격 방어(API Gateway 도 마찬가지)
Route53 - WAF 또는 Route53 - API Gateway

피크타임에만 몰리는 서버의 경우는 버스팅(크레딧)을 제공하는 T2 EC2를 사용하는 것이 더 나을 수 있다.
- T2는 Auto Scaling Group에 넣지 않는 것이 더 나을 수 있다.

Simian Army
넷플릭스 서비스 자동 장애발생 도구: 상용 에서 HA 테스트 목적 - 운영의 고도화
.Chaos Monkey - 특정 인스턴스 종료
.Chaos Gorilla - 특정 AZ 다운
.Chaos Kong - 특정 Region 다운

암호화
.기본 원칙: 대칭키
.마스터키 -> 데이터키 --> 암복호화
.데이터키로 암호화된 데이터와 마스터키로 암호화된 데이터키를 함께 저장: AWS 내부 스토리지
  - AES256방식으로 암호화, 각 키는 Region 별 저장
.HSM: 암호화작업 및 키 스토리지용 하드웨어 디바이스 제공
.Glacier는 기본적으로 데이터 암호화, S3와 Redshift는 선택사항
.S3 암호화 (데이터 & 암호화 키)
  - 고객 제공 키를 이용해서 암호화
  - S3에서 관리되고 보호되는 마스터키를 이용한 암호화
  - AWS KMS에서 제공 및 중앙 관리되는 키를 이용한 암호화
.Redshift
  - 4단계 암호화: Region Master Key -> Cluster Master Key -> Database Key - Data Key --> Data Encrypt

.HTTPS
앞단 ELB, Cloudfront에서 직접 HTTPS 인증서 처리를 대신 할 수 있다.
. 웹서버 오프로드 감소

Internet G/W -- Public ELB (이친구를 통해서만 외부 연결) ---> Private ELB

시험에는 EC2 인스턴스 별 네트워크 성능 꼭 나온다..

EBS는 Multi-thread에 대해서 단일 I/O로 병합하여 처리
- PIOPS : 용량/성능 중점 선택이 가능한 고가용 디스크
- gp2 타입: 즉각적인 확장 기능(버스팅) 제공 - T타입 EC2처럼 크레딧 방식
- AZ내에서 Mirroring 가능

디스크 성능
. Hadoop같은 분석용 스토리지는 i2타입같은 내부 디스크를 사용하되 고성능 I/O를 제공하는 인스턴스를 휘발성으로 사용
. 스트라이핑
. RAID를 사용(mdadm, RAID 0/1)
  - RAID 5또는 6이나 패리티를 사용하는 RAID는 사용을 피할 것!
. Instance Store를 EBS에 미러링 ... --> 물론 복구과정은 필요

GET 버킷 작업
- 버킷 객체 앞단에 키 값의 해싱 값을 적용: 각 파티션 내에서 잘 분배 되도록

"NAT 자체의 병목 발생 가능성 고려"
"Amazon S3 엔드 포인트"
"EBS Snapshot는 별도 네트워크 구간으로 S3 저장"

셤
AWS 사이트 샘플문항의 답은 일본어 버전에만 있다 + 공식교재