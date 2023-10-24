- 주로 생각한 것

  - 멀티프로세싱 or 쓰레딩 활용하여 객체별로 연산
  - 그런데 멀티쓰레딩의 경우 느림
  - 멀티프로세싱은 좀 빠름

- 사실 원래 코드가 가장 빨랐음

  - 원래 코드는 단일 프로세스 단일 쓰레드
  - 다만 하나의 while문 안에서 모든 cam이 동작함
  - 그러면 while문의 중첩이 성능저하의 원인인가?
    - while문이 두개 중복해서 돌아가는 multi-thread의 경우 1개의 쓰레드일때 16, 2개의 쓰레드일 때 8로 절반씩 나눠지고 있었음
    - 일단 멀티프로세스 or 쓰레드 쓰지 않고 각각의 객체로 생성해봄(멀티쓰레드와 비슷한 결과가 예상됨)
    - 각각의 객체로 생성하니 한번에 나오지 않고 둘이 따로따로 창이 켜짐
      - 객체 내부에 while이 있으니까.
      - 그럼 run에 있는 while문을 밖으로 빼고 구현하려는 객체들을 각각 내부에 넣어야하나?
      - 클래스에서 run부분을 main으로 위와 같이 옮겨서?
        - 그 결과 각각의 창이 동시에 켜지면서 싱글쓰레드였지만 오히려 빨랐음

- 왜 멀티쓰레드를 딕셔너리식으로 하니까 안됐지?

  - 딕셔너리 방식이 문제가 아니라 멀티프로세싱의 작동 방식과 네임스페이스, 메모리 공간에 대한 이해 부족으로부터 발생한 이슈였음

  - 멀티프로세스는 `if __name__ = "__main__":` 이하의 메인 네임스페이스에서 자식 네임스페이스인 `__mp_main__` 으로 분기됨

    - 이때 네임스페이스 개념으로 접근하면 안되고, 독립된 메모리 공간으로 생각해야 함.
    - with 이하의 구문으로 정의되는 excutor는 할당받은 worker 수 만큼 독립된 메모리 공간을 만드는데, 이 때 생성되는 것은 독립된 프로세스임
    - 독립된 프로세스들은 현재 실행중인 파일의 복사본을 각각의 메모리 공간에서 실행하는데, 이 때 설정되는 네임스페이스는 `__mp_main__` 가 된다.
    - 그 내부에서 우리가 여러개를 돌리고 싶은 함수가 실제로 돌아가게 됨.
    - 문제의 원인은 excutor가 실행시킬 함수가 전역 네임스페이스에 정의되어있지 않았기 때문에 발생한 문제였음.
      - 즉 함수 내부에서 찾는게 아니라 전역 공간에서 찾기 때문에, 만약 원래 방식대로 하려면 실행하려는 함수를 정의하고 있는 함수를 실행해야함. 그래야 타겟 함수가 그 함수에 의해 정의되니까.

    - 결과적으로 `__main__` 네임스페이스에서 다시 실행(된다고 봐도)되기 때문에 해당 전역  네임스페이스에 타겟 함수가 정의되어 있어야 한다.

- 왜 싱글쓰레드 방식은 프레임수가 안나오지?

  - 구조의 문제였음.
    - 프레임 처리가 완료된 다음 송출되는데,
    - 프레임 처리가 이미 완료된 다음 개별 프레임에 텍스트를 추가하려고 함.
    - 따라서 프레임 처리 도중에 텍스트를 추가한 다음 송출하는 식으로 바꿔서 해결함.

- 싱글 쓰레드 방식은 빠르고 좋긴한데 일일이 변수들을 복사 붙여넣기 해야되는지 생각해보기.