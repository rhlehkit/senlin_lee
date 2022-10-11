========================
Lee Senlin
========================

10월 11일


senlin-api¶
The senlin-api component provides an OpenStack-native REST API that processes API requests by sending them to the senlin-engine over RPC.


oslo-service OpenStack 서비스를 실행하기 위한 라이브러리




Senlin에 대한 분석 시작 

10월 5일 

대략적인 구조 파악 

Senlin의 경우 oslo_* 라는 프레임워크(?) 를 상속 받아 구현되어 있음 

다른 친구들(nova, 뉴트론 등등)도 해당 프레임워크를 상속 받아 구현되어 있는데 그 이유는 2가지로 추측 

1. 각 모듈에 대한 코드 관리(?) 구조에 대해 공통으로 가져가기 위해

2. 각 모듈(?) 간에 통신의 경우 RPC로 구현되어 있음 이에 대한 지원을 위해 ?


ref : 
Oslo : https://wiki.openstack.org/wiki/Oslo
rpc : 


Database 관련

블리자드의 Senlin 적용기에 따르면 특히 많은 클러스터가 운영 중일 때 데이터베이스 효율성이 떨어졌고 액션 리스트, 고객 리스트 작업에서 지연이 나타났다

라고 한다. 그 이유를 찾아보면

def cluster_lock_acquire(context, cluster_id, action_id, engine=None,
                         scope=CLUSTER_SCOPE, forced=False):
                         
                         
def node_lock_acquire(context, node_id, action_id, engine=None,
                      forced=False)

와 같은 lock를 빈번하게 사용하고 있음

원인 1. ?

    self._lock = threading.Lock()
    
    Oslo에서 제공하는 모듈의 경우 threading Lock() 를 사용하고 있음 
    물론 각 execute가 thread로 돌아가긴 하는데 굳이 ? lock()을 잡아야하나 ?
    

원문보기:
https://www.ciokorea.com/tags/662/%EC%98%A4%ED%94%88%EC%8A%A4%ED%83%9D/122204#csidx365d0a2ce03c92b9a43ce1eba536853 

10월 6일 정리 내용 --> 내일 다시 내용정리 

Senlin의 경우 Lock를 무지하게 잡는다. 근데 내가 지식이 부족해서 그런가 굳이 lock()을 잡아야 하나
비동기로 처리 하는 방식을 통해 빠르게 thread를 처리하면 안되나? 
왜냐면 Senlin의 경우 새로운 이벤트(?) 동작이 생길때 tg에 add_~~ 를 통해 새로운 thread를 만들고 프로세스가 진행이 되는데
이렇게 lock을 잡으면 되나 이거 ?
Senlin main 

Senlin 기능 

1번 정책
정책에는 특정 클러스터 작업이 수행되기 전이나 후에 확인/시행되는 규칙 집합이 포함됩니다. 

2번 수신기 
수신기는 senlin 클러스터와 관련된 이벤트 싱크입니다.
rest api를 통해 senlin 클러스터에 대한 in out 등의 설정 가능


type: senlin.policy.scaling
설정에서 알아두어야 할 것 
최선의 노력 확장 ¶
많은 Auto-Scaling 사용 시나리오에서 정책 결정은 클러스터에 설정된 크기 제약 조건을 깨뜨릴 수 있습니다. 예를 들어 클러스터는 min_size 5로 max_size설정되고 10으로 설정되고 현재 용량은 7입니다. 정책 결정이 클러스터에서 3개의 노드를 제거하는 것이라면 우리는 딜레마에 빠져 있습니다. 3개의 노드를 제거하면 클러스터 용량이 4로 변경되며 이는 클러스터에서 허용하지 않습니다. 3개의 노드를 제거하지 않으면 정책 결정을 존중하지 않는 것입니다.

이 adjustment.best_effort속성은 이러한 상황을 완화하도록 설계되었습니다. False로 설정하면 조정 정책이 설정된 규칙을 엄격히 준수합니다. 계산된 클러스터 용량이 크기 제약을 깰 경우 확장 요청을 거부합니다. 그러나 adjustment.best_effort가 True로 설정된 경우 조정 정책은 클러스터의 크기 제약 조건을 위반하지 않는 차선의 숫자를 계산하기 위해 노력합니다. 위의 예에서 이는 정책 결정이 "클러스터에서 2개의 노드를 제거"한다는 것을 의미합니다. 즉, 정책은 크기 제약을 준수하기 위해 최소한 부분적으로 조정 목표를 달성하려고 시도합니다.

다른 정책과의 상호 작용 ¶
확장 정책은 추가하거나 제거할 노드 수를 결정하는 역할만 합니다. 새로 추가된 노드의 경우 다른 정책을 사용하여 예약할 위치를 결정합니다. 삭제할 노드의 경우 다른 정책(예: 삭제 정책)을 사용하여 피해자 노드를 선택합니다.

기본 제공되는 정책은 함께 또는 혼자서 행복하게 일할 수 있도록 세심하게 설계되었습니다.

프로필
Profile 은 Senlin 서비스에서 관리 할 Node 를 생성하는 데 사용 되는 틀 입니다

클러스터
클러스터 는 논리적 개체의 그룹이며, 각 개체는 Senlin 용어 로 노드 라고 합니다. 
클러스터를 생성하려면 클러스터와 연결할 프로필의 ID 또는 이름을 제공
복원 작업은 지정된 클러스터에서 노드를 삭제하고 다시 만듭니다.
클러스터 개체 자체를 삭제하기 전에 클러스터에서 모든 노드를 삭제하고 파괴하는 프로세스를 시작합니다.

노드 는 Senlin 서비스에서 관리하는 논리적 개체입니다
최대 한개의 클러스터에 포함 할 수 있으며 클러스타가 없을 수도 있음
문제 ? Senlin에서는 노드의 상태만 체크 합니다. 문제는 노드=VM으로써 실제 유저가 원하는건 App 연결
LB를 붙이긴 해도 VM 이후 App이 제대로 떴는지에 대한 체크 로직이 필요하고 제대로 뜨지 않는 경우 복구 등의 로직이 필요한데
해당 로직이 보이지 않음




이 문구가 중요
정책 유형의 레지스트리는 Senlin 엔진( senlin-engine )이 시작될 때 메모리에 구축됩니다. 앞으로 Senlin은 사용자가 동적으로 로드할 플러그인으로 추가 정책 유형 구현을 제공할 수 있도록 할 것입니다.


정책 은 정책 유형 에서 인스턴스화 된 개체 입니다.
일단 생성되면 클러스터에 동적으로 연결하거나 클러스터에서 분리할 수 있습니다



Batch Policy
일괄 정책은 많은 수의 작업을 더 작은 일괄 처리로 자동 그룹화하여 DOS(거부- 서비스) 공격으로 인한  서비스 중단을 보다 잘 관리할 수 있도록 함


Healthy 정책
상태 정책은 Senlin이 클러스터 노드 장애를 감지하고 사용자가 사용자 정의할 수 있는 방식으로 복구하도록 설계
고가용성과 관련된 모든 문제를 해결할 수 있는 보편적인 솔루션을 의미하지 않습

문제점 ? : health 체크를 위해 제공 되는 정책 중 폴링 방식, 이벤트를 받는 리스너 방식이 있음 
아마 보편적으로 폴링 방식을 쓸 거 같은데 만약 관리하는 Node가 많으면 과연 senlin이 이를 감당 할 수 있을까?
물론 천천히 한개씩 확인하겠지만 이 경우 대응이 느릴거 같음 ? 
코드 분석이 필요함


설정 과정에서 
type: senlin.policy.health
version: 1.1
properties:
  detection:
    interval: 120
    detection_modes:
      - type: NODE_STATUS_POLL_URL
        options:
            poll_url: "http://{nodename}/healthstatus"
            poll_url_healthy_response: "passing"
            poll_url_conn_error_as_unhealty: true
            poll_url_retry_limit: 3
            poll_url_retry_interval: 2
노드 네임만 지원 ? 
ip도 지원 해야 하지 않나 ?
{nodename}각 노드에서 실행되는 애플리케이션에 의해 구현된 URL을 쿼리하는 데 사용할 수 있습니다. 이를 위해서는 DNS 서비스에 새 서버 인스턴스의 이름을 자동으로 등록하도록 OpenStack 클라우드를 설정해야 합니다. 향후 노드 IP 주소에 대한 새로운 확장 매개변수에 대한 지원이 추가될 수 있습니다.


문제점 ? 
복구 조치 계획은 Senlin 엔진에서 하나씩 시도할 수 있는 조치 목록을 지원하는 것입니다. 현재 구현 제한으로 인해 하나 의 작업 만 지정할 수 있습니다 .

복구 작업의 또 다른 확장은 사용자 제공 워크플로에 트리거를 추가하는 것입니다. 이 또한 개발 중입니다.

복구 작업 ¶
참고 : 현재 목록에서 단일 작업만 지원합니다. Mistral 작업 흐름에 대한 지원도 진행 중인 작업입니다

기본 복구 작업 ¶
Senlin은 서로 다른 유형의 자원을 관리하도록 설계되었으므로 각 자원 유형, 즉 Profile Type 은 장애 복구에 사용할 수 있는 다양한 작업 집합을 지원할 수 있습니다.
실패한 리소스를 복구하는 보다 실용적이고 일반적인 작업은 이전 리소스를 삭제한 다음 새 리소스를 생성하여 RECREATE 작업을 수행하는 것입니다. 이 RECREATE작업은 충분히 일반적이지만 사용자가 원하는 것일 수도 있고 아닐 수도 있습니다. 예를 들어, 재생성된 Nova 서버가 물리적 ID나 IP 주소를 보존한다는 보장은 없습니다. 원래 서버의 임시 상태는 확실히 손실됩니다.

펜싱 지원
비활성 상태인 것처럼 보이는 노드가 여전히 작동 중이고 노드가 아직 살아 있을 가능성을 고려하지 않고 미성숙한 복구 작업만 시도하면 이러한 노드가 전체 클러스터를 예측할 수 없는 상태로 만드는 경우가 많이 있습니다.
이를 고려하여 보건정책에서 울타리에 대한 지원을 모델링하고 구현하는 작업을 하고 있습니

지역 배치 정책
  
지역 배치 정책은 여러 지역에서 배포 및 관리 리소스 풀을 사용하도록 설계되었습니다. 현재 디자인은 여러 지역에 대한 단일 키스톤 엔드포인트에만 관련되어 있으며 키스톤 연합과의 상호 작용은 향후 확장을 위해 계획되어 있습니다.
정책은 모든 프로필 유형의 클러스터에서 작동하도록 설계되었습니다.

영역 배치 정책
영역 배치 정책은 여러 가용성 영역에서 배포 및 관리 리소스 풀을 사용하도록 설계되었습니다. 현재 설계는 Nova 컴퓨팅 서비스에 구성된 가용성 영역에만 관련됩니다. Cinder 가용성 영역 및 Neutron 가용성 영역에 대한 지원은 향후 볼륨 저장소별 또는 네트워크별 프로필 유형이 있을 때 추가될 수 있습니다.

영역 배치 정책의 현재 구현은 Nova 가상 머신의 클러스터에서만 작동합니다.

클러스터에 정책 연결
대부분의 경우 Senlin은 동일한 클러스터에 동일한 유형의 정책을 두 개 이상 연결하는 것을 허용하지 않습니다. 이 제한은 일부 정책 유형에 대해 완화됩니다.
예를 들어, 조정에 대한 정책으로 작업할 때 실제로 둘 이상의 정책 인스턴스를 동일한 클러스터에 연결할 수 있습니다.

원하는 용량 줄이기

Senlin -> celimeter, ador, heat 랑 연결 


def main() : 

    srv = engine.EngineService(CONF.host,
                               consts.ENGINE_TOPIC)
    launcher = service.launch(CONF, srv,
                 
EngineService 실행 -> service.launch 

EngineService.start()
self.server = messaging.get_rpc_server(self.target, self)
rpc server Start()

// """Run the given method in a thread."""
def execute 


"Run action(s) in sub-thread
def start_action



start_action에서 특정 action에 대해 acquire 함수 내에서 select for update 
를 통해 lock을 잡는데 만약 senlin이 한개명 dict 나 내부 변수를 통해 잡아도 되는거 아닐까?
성능 이슈가 있을 거 같음 만약
      if action_id is not None:
            timestamp = wallclock()
            action = ao.Action.acquire(self.db_session, action_id,
                                       self.service_id,
                                       timestamp)
            if action:
                self.execute(action_mod.ActionProc, self.db_session, action.id)
                actions_launched += 1

1번 왜냐면 위에 로직에서 만약 action이 실패하는 경우 
다음 로직에서 아래 처럼 while을 계속 돌게 됌

        while True:
            timestamp = wallclock()
            action = ao.Action.acquire_first_ready(self.db_session,
                                                   self.service_id,
                                                   timestamp)
            if not action:
                break

            if max_batch_size == 0 or 'NODE' not in action.action:
                self.execute(action_mod.ActionProc, self.db_session, action.id)
                continue

            if max_batch_size > actions_launched:
                self.execute(action_mod.ActionProc, self.db_session, action.id)
                actions_launched += 1
                continue

            self.execute(action_mod.ActionProc, self.db_session, action.id)

            LOG.debug(
                'Engine %(id)s has launched %(num)s node actions '
                'consecutively, stop scheduling node action for '
                '%(interval)s second...',
                {
                    'id': self.service_id,
                    'num': max_batch_size,
                    'interval': batch_interval
                })

            sleep(batch_interval)
            actions_launched = 1


개발자 가이드 클러스터

빈번한 데이터베이스 액세스를 피하기 위해 클러스터 객체에는 Python 사전 이라는 런타임 데이터 속성 이 있습니다. 속성은 클러스터에서 참조하는 프로필, 클러스터의 노드 목록 및 클러스터에 연결된 정책을 캐시합니다. 런타임 데이터는 사용자에게 직접 표시되지 않습니다. 클러스터 작업을 위한 편의일 뿐입니다.
  


작업이 실행을 위해 작업자 스레드에 의해 선택되면 Senlin은 많은 NODE_LEAVE관련 NODE_JOIN작업을 분기하고 비동기적으로 실행합니다. 모든 분기된 작업이 완료되면 CLUSTER_REPLACE_NODES 성공으로 반환됩니다. ?? 

이거 참인가 ?




Senlin 구조 그리기


