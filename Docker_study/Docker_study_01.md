컨테이너를 만드는 것이 create_container 시스템 호출만큼 간단하다면 좋을 것이나, 그렇지 않습니다. 그러나, 솔직히 말해서 비슷합니다.

낮은 수준의 컨테이너에 대해 이야기하려면, 세 가지에 대해 이야기해야 합니다. 컨테이너에 대해 낮은 수준에서 이야기하려면, 세 가지에 대해 이야기해야 합니다. 이것들은 네임스페이스, C그룹 그리고 계층 파일시스템입니다. 다른 것들도 있으나, 이 세 가지는 대부분을 구성합니다.

# **Namespaces**
네임스페이스는 하나의 시스템에서 여러 컨테이너를 실행하는 데 필요한 격리를 제공하면서 각각 고유한 환경처럼 보입니다. 다음 6개의 네임스페이스 각각은 독립적으로 요청될 수 있으며, 프로세스(및 그 하위)에게 컴퓨터 자원의 서브 세트를 제공합니다.

네임스페이스는 다음과 같습니다:

- PID: pid 네임스페이스는 프로세스와 그 자식들에게 시스템 내 프로세스의 하위 집합에 대한 고유한 뷰를 제공합니다. 이를 매핑 테이블로 생각합니다. pid 네임스페이스의 프로세스가 커널에 프로세스 목록을 요청하면 커널은 매핑 테이블을 찾습니다. 프로세스가 테이블에 존재하면 실제 ID 대신 매핑된 ID가 사용됩니다. 매핑 테이블에 존재하지 않으면, 커널은 존재하지 않는 척합니다. pid 네임스페이스는 호스트 ID가 무엇이든 1에 매핑하여 첫 번째 프로세스를 pid 1로 만들어 컨테이너에 격리된 프로세스 트리의 모양을 제공합니다.

- MNT: 어떤 면에서, 이것은 가장 중요합니다. 마운트 네임스페이스는 프로세스에 포함된 자체 마운트 테이블을 제공합니다. 즉, 다른 네임스페이스(호스트 네임스페이스 포함)에 영향을 주지 않고 디렉터리를 마운트 및 마운트 해제할 수 있습니다. 보다 중요한 것은, pivot_root syscall과 결합하여 프로세스가 자체 파일시스템을 가질 수 있도록 하는 것입니다. 컨테이너가 보는 파일시스템을 교체하여 우분투나 비지박스 또는 알파인에서 프로세스가 실행되고 있다고 생각할 수 있는 방법입니다.

- NET: 네트워크 네임스페이스는 자체 네임스페이스를 사용하는 프로세스를 제공합니다. 일반적으로 주 네트워크 네임스페이스(컴퓨터를 시작할 때 시작되는 프로세스 네임스페이스)에만 실제 물리적 네트워크 카드가 연결됩니다. 그러나, 가상 이더넷 쌍을 만들 수 있습니다. 한쪽 끝은 한 네트워크 네임스페이스에 배치하고 다른 쪽 끝은 네트워크 네임스페이스 사이에 가상 링크를 만들 수 있는 연결된 이더넷 카드입니다. 하나의 호스트에서 여러 개의 IP 스택이 서로 통신하는 것과 같습니다. 라우팅을 사용하면 각 컨테이너가 실제 세계와 통신하면서 각 컨테이너를 자체 네트워크 스택에 격리시킬 수 있습니다.

- UTS: UTS 네임스페이스는 프로세스에 시스템의 호스트 이름 및 도메인 이름에 대한 고유한 뷰를 제공합니다. UTS 네임스페이스를 입력한 후, 호스트 이름 또는 도메인 이름을 설정해도 다른 프로세스에는 영향을 미치지 않습니다.

- IPC: IPC 네임스페이스는 메시지 대기열과 같은 다양한 프로세스 간 통신 메커니즘을 분리합니다.

- USER: 사용자 네임스페이스가 가장 최근에 추가되었으며, 보안 측면에서 가장 강력할 것입니다. 사용자 네임스페이스는 프로세스가 보는 uid를 호스트의 다른 uid(및 gid) 세트에 매핑합니다. 이것은 매우 유용합니다. 사용자 네임스페이스를 사용하여 컨테이너의 루트 사용자 ID(예: 0)를 호스트의 임의의(권한이 없는) uid에 매핑할 수 있습니다. 즉, 루트 네임스페이스에서 권한을 부여하지 않고도 컨테이너가 루트 액세스 권한을 가지고 있다고 생각할 수 있습니다. 실제로 컨테이너 특정 리소스에 대한 루트 권한을 부여할 수도 있습니다. 컨테이너는 uid 0으로 자유롭게 프로세스를 실행할 수 있으며, 이는 일반적으로 루트 권한 보유와 동의어일 수 있으나, 커널은 실제로 커버 아래에 있는 uid를 권한 없는 실제 uid에 매핑하고 있습니다. 대부분의 컨테이너 시스템은 컨테이너의 uid를 호출 네임스페이스의 uid 0에 매핑하지 않습니다. 즉, 컨테이너에 실제 루트 권한이 있는 uid가 없습니다.

대부분의 컨테이너 기술은 사용자 프로세스를 위의 모든 네임스페이스에 배치하고 네임스페이스를 초기화하여 표준 환경을 제공합니다. 예를 들어, 실제 네트워크에 연결할 수 있는 컨테이너의 격리된 네트워크 네임스페이스에 초기 인터넷 카드를 만드는 것입니다.

# **CGroups**
기본적으로 C그룹은 일련의 프로세스 또는 작업 ID를 수집하여 제한을 적용합니다. 네임스페이스가 프로세스를 분리하는 경우 C그룹은 프로세스 간에 공정한 (또는 불공평한 방식으로) 리소스 공유를 시행합니다.

C그룹은 커널이 마운트할 수 있는 특수 파일시스템으로 노출됩니다. 단순히 프로세스 ID를 작업 파일에 추가하여 C그룹에 프로세스 또는 스레드를 추가한 다음 해당 디렉터리의 파일을 편집하여 다양한 값을 읽고 구성합니다.

# **Layered Filesystems**
네임스페이스와 C그룹은 컨테이너화의 격리 및 리소스 공유 측면입니다. 계층화 된 파일시스템은 전체 머신 이미지를 효율적으로 이동할 수 있는 방법입니다. 기본 레벨에서 계층화 된 파일시스템은 각 컨테이너에 대한 루트 파일시스템의 사본을 작성하기 위한 호출을 최적화합니다. 이를 수행하는 방법에는 여러 가지가 있습니다. Btrfs는 파일시스템 계층에서 copy on write를 사용합니다. Aufs는 "union mounts"를 사용합니다. 이 단계를 수행하는 방법은 매우 다양하므로 이 문서에서는 아주 간단한 방법을 사용합니다. 실제로 사본을 작성합니다. 느리긴 하나, 작동은 합니다.