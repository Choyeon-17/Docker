# **Skeleton Code**
Docker_code/DockerSkeleton.go의 코드는 무엇을 할까요?  
main.go에서 시작하여 첫 번째 인수를 읽습니다. 'run'인 경우, parent() 함수를 실행하고, 'child'인 경우, 우리는 child() 함수를 실행합니다. parent() 함수는 '/proc/self/exe'를 실행하는데, 이것은 현재 실행 파일의 메모리 내 이미지를 포함하는 특수 파일입니다. 다시 말해, 스스로를 재실행하나, 첫 번째 인수로서 child를 전달합니다.

놀라운 것은 단지 사용자 요청 프로그램을 실행하는 또 다른 프로그램을 실행할 수 있게 합니다('os.Args[2:]'에 제공). 그러나, 이 간단한 스캐 폴딩으로 컨테이너를 만들 수 있습니다.