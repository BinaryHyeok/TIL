# 쓰레드 생성

- Thread

```c#
using System;
using System.Threading;

namespace ServerCorePractice
{
    class Program
    {
        static void mainThread(object state)
        {
            for(int i = 0; i < 5; i++)
                Console.WriteLine("Hello Thread");
        }

        static void Main(string[] args)
        {
            //인력사무소
            //대기하던 직원들을 투입시키고 다시 복귀시키는 개념
            //너무 많은 일을 던지면 안된다.
            ThreadPool.QueueUserWorkItem(mainThread);

            //해당 업무만 하는 직원
            //쓰레드를 생성하여 어떤 함수를 실행할 지 결정
            Thread t = new Thread(mainThread);
            //백그라운드에서 실행 -> 메인함가 종료되면 쓰레드도 종료
            t.Name = "Test Thread";
            t.IsBackground = true;
            //쓰레드시작
            t.Start();

            Console.WriteLine("Waiting Thread");
            //쓰레드가 끝날 때 까지 기다렸다가 시작
            t.Join();
            Console.WriteLine("Hello World!");

            while(true)
            {

            }
        }
    }
}
```

- ThreadPool

```c#
ThreadPool.SetMinThreads(1, 1);
ThreadPool.SetMaxThreads(5, 5);

//5개의 쓰레드 모두 무한루프에 빠짐
for (int i = 0; i < 5; i++)
     ThreadPool.QueueUserWorkItem((obj) => { while (true) { } });

//실행 불가
ThreadPool.QueueUserWorkItem(mainThread);
```

- Task

```c#
ThreadPool.SetMinThreads(1, 1);
ThreadPool.SetMaxThreads(5, 5);


for(int i = 0; i < 5; i++)
    {
    //LongRunning 옵션을 넣어서 ThreadPool에서 thread를 뽑아서 쓰지 않도록 설정
    //LongRunning 옵션이 없다면 설정한 5개의 쓰레드에서 뽑아서 사용
    Task t = new Task(() => { while (true) { } }, TaskCreationOptions.LongRunning);
    t.Start();
    }

ThreadPool.QueueUserWorkItem(mainThread);
```

<br>

# 컴파일러 최적화

릴리즈 모드에서는 코드 최적화로 인하여 일어나지 않을 버그가 생긴다.

```c#
using System;
using System.Threading;

namespace ServerCorePractice
{
    class Program
    {
        //전역은 모든 스레드가 동시에 접근하여 사용가능하다
        //volatile를 붙여서 최적화 무시, 있는 그대로 써라
       volatile static bool _stop = false;

        static void ThreadMain()
        {
            Console.WriteLine("Thread start");

            while(_stop == false)
            {
                // wating stop flag
            }

            Console.WriteLine("Thread end");

        }

        static void Main(string[] args)
        {
            Task t = new Task(ThreadMain);
            t.Start();

            Thread.Sleep(1000);

            _stop = true;

            Console.WriteLine("call stop");
            Console.WriteLine("wating terminate");

            t.Wait();

            Console.WriteLine("terminate success");
        }
    }
}
```

<br>

# 캐시이론

## 캐시

캐시는 데이터나 값을 미리 복사 해놓은 임시 장소를 가르킨다.  
캐시는 캐시의 접근 시간에 비해 원래 데이터를 접근하는 시간이 오래 걸리는 경우나 값을 다시 계산하는 시간을 절약하고 싶은 경우에 사용한다.  
캐시에 데이터를 미리 복사해 놓으면 계산이나 접근 시간 없이 더 빠른 속도로 데이터에 접근할 수 있다.

## 캐시철학

- **Temporal locality**
  - 최근에 접근했던 주소값을 다시 접근하는 경향
  - 한번 사용한 주소의 메모리 영역을 자주 접하게 된다.
- **Spartial Locality**
  - 최근 접근했던 주소값 근처의 주소들을 접근하는 경향
  - 한번 사용했던 주소의 근처 영역은 접근이 이루어질 확률이 관계없는 곳보다 더 높다.

```c#
using System;
using System.Threading;

namespace ServerCorePractice
{
    class Program
    {

        static void Main(string[] args)
        {
            int[,] arr = new int[10000, 10000];

            //하나의 행에대해서 열이 증가하므로 순차적으로 증가
            //Spartial Locality O
            {
                long now = DateTime.Now.Ticks;
                for(int y = 0; y < 10000; y++)
                {
                    for (int x = 0; x < 10000; x++)
                        arr[y, x] = 1;
                }
                long end = DateTime.Now.Ticks;
                Console.WriteLine($"(y, x) 순서 걸린 시간 {end - now}");
            }

            //하나의 열에대해서 행이 증가하므로 띄엄띄엄 증가한다
            //Spartial Locality X
            {
                long now = DateTime.Now.Ticks;
                for (int y = 0; y < 10000; y++)
                {
                    for (int x = 0; x < 10000; x++)
                        arr[x, y] = 1;
                }
                long end = DateTime.Now.Ticks;
                Console.WriteLine($"(y, x) 순서 걸린 시간 {end - now}");

            }
        }
    }
}
```

(y, x) 순 서 걸 린 시 간 2514210  
(x, y) 순 서 걸 린 시 간 4082970

<br>

# 메모리 배리어

메모리 배리어는 CPU나 컴파일러에게 배리어 명령문 전 후의 메모리 연산을 순서에 맞게 실행하도록 강제하는 기능이다.  
CPU가 성능을 좋게 하기 위해 최적화를 거쳐 순서에 맞지 않게 실행시키는 결과를 초래할 수 있기 때문에 메모리 배리어를 사용한다.  
메모리 연산의 재배치는 싱글 스레드로 실행할 때는 알아차리기 어렵지만, 멀티스레드에서는 예상할 수 없는 결과를 초래하기도 한다.

아래 코드는 순서에 맞게 실행된다면 무한루프를 빠져나올 수 없지만, 실제로는 무한루프에서 벗어난다.

```c#
using System;
using System.Threading;

namespace ServerCorePractice
{
    class Program
    {
        static int x = 0;
        static int y = 0;
        static int r1 = 0;
        static int r2 = 0;

        static void Thread_1()
        {
            y = 1; // Store y

            // Full Memory Barrier
            //Thread.MemoryBarrier();

            r1 = x; // Load x
        }
        static void Thread_2()
        {
            x = 1; // Store x

            // Full Memory Barrier
            //Thread.MemoryBarrier();

            r2 = y; // Load y
        }

        static void Main(string[] args)
        {
            int count = 0;
            while (true)
            {
                count++;
                x = y = r1 = r2 = 0;

                Task t1 = new Task(Thread_1);
                Task t2 = new Task(Thread_2);
                t1.Start();
                t2.Start();

                Task.WaitAll(t1, t2);

                if(r1 == 0 && r2 == 0)
                    break;
            }

            Console.WriteLine($"{count} get out!");
        }
    }
}
```

1. 코드 재배치 억제
   > 1. Full Memory Barrier(ASM MFENCE, C# Thread.MemoryBarrier) : Store/Load 둘 다 막는다.
   > 2. Store Memory Barrier (ASM SFENCE) : Store만 막는다.
   > 3. Load Memory Barrier (ASM LFENCE) : Load만 막는다.
2. 가시성
   > 값을 저장한 후 메모리 배리어를 호출하면 메모리에 해당 값을 쓰고, 값을 로드 하기 전에 메모리 배리어를 호출 하였다면, 메모리에서 값을 가져온다.  
   > **동기화 작업**이라고 할 수 있다.

<br>

# Interlocked

number++: 라는 동작은 실질적으로 다음과 같다.

1. int temp = number;
2. temp = temp + 1;
3. number = temp;

여기서 멀티스레드의 경우 순서가 꼬여 정상적으로 동작하지 않을 수 있다.  
이때 Interlocked를 이용하여 문제를 해결할 수 있다.

```c#
using System;
using System.Threading;

namespace ServerCorePractice
{
    class Program
    {
        static int number = 0;

        static void Thread_1()
        {
            for(int i  = 0; i < 100000; i++)
            {
                // 원자적으로 덧셈
                // All or Nothing
                Interlocked.Increment(ref number);
                //number++;
            }
        }

        static void Thread_2()
        {
            for(int i = 0; i < 100000; i++)
            {
                // 원자적 뺼셈
                // All or Nothing
                Interlocked.Decrement(ref number);
                //number--;
            }
        }

        static void Main(string[] args)
        {
            Task t1 = new Task(Thread_1);
            Task t2 = new Task(Thread_2);
            t1.Start();
            t2.Start();

            Task.WaitAll(t1, t2);

            Console.WriteLine(number);
        }
    }
}
```

<br>

# Lock 기초

몇십, 몇백줄을 Interlocked를 사용하기에는 한계가 있다.  
다음과 같은 코드는 Enter과 Exit사이에서 return이나 에러 발생 시 Exit를 실행하지 않게된다.

```c#
Monitor.Enter(_obj); // 문을 잠구는 행위

number++;

Monitor.Exit(_obj); // 잠금을 풀어준다.
```

try-catch-finally를 사용하면 해결 가능하며, lock은 다음을 포함하고 있다.

```c#
using System;
using System.Threading;

namespace ServerCorePractice
{
    class Program
    {
        static int number = 0;
        static object _obj = new object();

        static void Thread_1()
        {
            // lock을 걸어준다. 다음과 같이 사용하면 데드락 방지 가능
            lock (_obj)
            {
                number++;
            }
        }

        static void Thread_2()
        {
           lock (_obj)
            {
                number--;
            }
        }

        static void Main(string[] args)
        {
            Task t1 = new Task(Thread_1);
            Task t2 = new Task(Thread_2);
            t1.Start();
            t2.Start();

            Task.WaitAll(t1, t2);

            Console.WriteLine(number);
        }
    }
}
```

<br>

# Lock 구현

## SpinLock

무작정 기다린다.

```c#
using System;
using System.Threading;

namespace ServerCorePractice
{
    class SpinLock
    {
        volatile int _locked = 0;

        // Lock이 풀릴 때 까지 계속 돈다
        public void Acquire()
        {
            while (true)
            {
                // Exchange를 통해서 locked에 1의 값을 넣는다
                // 이때 이전의 값은 original이라는 변수로 뺴내고 만약 이 값이 0이라면 아무도 스레드를 사용하고 있지 않으므로 사용한다.
                //int original = Interlocked.Exchange(ref _locked, 1);
                //if (original == 0)
                //    break;

                int expected = 0;
                int desired = 1;
                // expected값과 같다면 desired값을 넣어준다.
                if (Interlocked.CompareExchange(ref _locked, desired, expected) == expected)
                    break;

                // 스레드를 누가 사용하고 있네? -> 쉬고올게
                // 무조건 휴식 -> 무조건 1ms 정도 쉬고 싶을 때
                //Thread.Sleep(1);
                // 조건부 양보 -> 나보다 우선순위가 낮은 애들에게는 양보 불가 -> 우선순위가 나보다 같거나 높은 스레드가 없으면 다시 본인이 사용
                //Thread.Sleep(0);
                // 관대한 양보 -> 관대하기 양보 할테니깐 실행 가능한 스레드가 있다면 실행 -> 실행 가능한 스레드가 없다면 본인이 사용
                Thread.Yield();

            }
        }

        public void Release()
        {
            _locked = 0;
        }
    }

    class Program
    {

        static int _num = 0;
        static SpinLock _lock = new SpinLock();

        static void Thread_1()
        {
            for(int i = 0; i < 100000; i++)
            {
                _lock.Acquire();
                _num++;
                _lock.Release();
            }
        }

        static void Thread_2()
        {
            for (int i = 0; i < 100000; i++)
            {
                _lock.Acquire();
                _num--;
                _lock.Release();
            }
        }


        static void Main(string[] args)
        {
            Task t1 = new Task(Thread_1);
            Task t2 = new Task(Thread_2);

            t1.Start();
            t2.Start();

            Task.WaitAll(t1, t2);

            Console.WriteLine(_num);
        }
    }
}
```

## AutuResetEvent

```c#
class Lock
    {
        // 처음 상태를 선택하여 시작 가능
        AutoResetEvent _available = new AutoResetEvent(true);

        public void Acquire()
        {
            // 입장 시도 -> 자동으로 문을 잠군다
            _available.WaitOne();
        }

        public void Release()
        {
            // flag = true
            _available.Set();
        }
    }
```

## ManualResetEvent

```c#
 class Lock
    {
        // 처음 상태를 선택하여 시작 가능
        ManualResetEvent _available = new ManualResetEvent(true);

        public void Acquire()
        {
            // 입장 시도 -> 자동으로 문을 잠구지 않는다
            _available.WaitOne();
        }

        public void Release()
        {
            // flag = true
            _available.Set();
        }
    }
```

Auto & Manual 둘 다 커널 단으로 가서 요청을 해야되므로 시간이 오래걸린다.
