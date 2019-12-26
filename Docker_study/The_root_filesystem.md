# **The Root Filesystem**
현재 프로세스는 격리된 네임스페이스 집합에 있습니다. 그러나, 파일시스템은 호스트와 동일하게 보입니다. 마운트 네임스페이스에 있으나, 초기 마운트는 생성 네임스페이스에서 상속되기 때문입니다.

루트 파일시스템으로 바꾸려면, 다음과 같은 간단한 4줄이 필요합니다. child() 함수의 시작 부분에 바로 배치하면 됩니다.
~~~ go
must(syscall.Mount("rootfs", "rootfs", "", syscall.MS_BIND, ""))
    must(os.MkdirAll("rootfs/oldrootfs", 0700))
    must(syscall.PivotRoot("rootfs", "rootfs/oldrootfs"))
    must(os.Chdir("/"))
~~~
마지막 두 줄은 중요한 비트이며, OS에 '/'의 현재 디렉터리를 'rootfs / oldrootfs'로 옮기고, 새 rootfs 디렉터리를 '/'로 바꾸라고 지시합니다. 'pivotroot' 호출이 완료되면, 컨테이너의 / 디렉터리는 rootfs를 참조합니다.('pivotroot' 명령의 일부 요구 사항을 충족시키기 위해 바인드 마운트 호출이 필요합니다. OS는 'pivotroot'를 사용하여 동일한 트리의 일부가 아닌 두 개의 파일시스템을 교체해야 합니다.)