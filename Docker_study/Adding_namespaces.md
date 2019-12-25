# **Namespaces**
프로그램에 네임스페이스를 추가하려면 한 줄만 추가하면 됩니다. parent() 함수의 두 번째 행에서 다음 행을 추가하여 하위 프로세스를 실행할 때 추가 플래그를 전달하도록 지시해야 합니다.
~~~ go
cmd.SysProcAttr = &syscall.SysProcAttr {
    Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWPID | syscall.CLONE_NEWNS
}
~~~
지금 프로그램을 실행하면, 프로그램이 UTS, PID 및 MNT 네임스페이스 내에서 실행됩니다.