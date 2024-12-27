`thread <ID>` 는 특정 thread의 시점으로 옮기는 것
malloc은 thread-safe function이기 때문에 특정 thread에서 `malloc`을 하고 다른 thread를 옮기면 `vis` 커맨드를 쳤을 때 heap이 할당된 적 없다고 뜰 수 있다.
**scheduler-locking**을 적용하면 내가 scheduler가 된 것 처럼 행동할 수 있다.
`on` 을 이용하면 내가 임의로 scheduling을 해볼 수 있다. 즉 특정 순서로 scheduling을 한다고 했을 때 그것의 validity를 확인할 수 있는 것.

Snu Bank 문제는 쉽다.
HTTP Race 문제는 상대적으로 쉽다.

thread schedule을 replay로 두면 OS가 다른 thread도 돌릴 수 있다. 내가 보고자 하는 thread만 continue를 하더라도 말이다.
scheduler-locking on을 켜서 디버깅을 해보자
똑같이 1번으로 이동해서 continue를 하면 thread2의 rip의 주소가 변하지 않는다. 계속 멈춰있는 것.
2를 진행시키려면 직접 `thread 2` 를 통해 넘어가서 continue를 해줘야 한다.
