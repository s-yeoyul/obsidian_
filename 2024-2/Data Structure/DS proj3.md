unique_ptr 문제는 해결했다.

원인은 RBNode\<T>::insert함수에서
마지막에 `n.get()` 으로 return 하던거... 이렇게 하면 안되고
`n.release()` 로 소유권을 포기하면서 RBTree의 `root.reset()` 안으로 raw pointer를 전달해야지 정상적으로 root가 업데이트 되는 것을 알 수 있다.

`n.get()` 으로 전달하면 하늘 아래 두 태양이 뜬 것과 같다. 잘은 모르지만 둘 다 죽게 되고, nullptr이 된다.
당연히 nullptr에서 left right를 접근하니까 segfault가 발생.

