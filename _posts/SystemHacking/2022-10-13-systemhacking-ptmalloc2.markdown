---
layout: post
title: <System Hacking> 20. Heap, 그리고 ptmalloc2
date: 2022-10-13 10:30:23 +0900
category: System_Hacking
comments: true
---

## Memory Allocator : ptmalloc2

운영체제의 핵심 역할 중 하나는 한정된 메모리 자원을 각 프로세스에 효율적으로 배분하는 일이다. 모든 프로세스는 실행 중에 메모리를 동적으로 할당하고, 할당한 메모리를 다 사용한 이후에는 이를 해제하는 작업을 빈번하게 수행한다. 이러한 과정을 수행하는 것이 운영체제의 Memory Allocator이며, 이 동작이 빠르고 낭비없이 이루어지도록 구현되어 있다. 대표적으로는 리눅스의 ptmalloc2, 구글의 tcmalloc, 페이스북과 파이어폭스의 jemalloc이 있으며, 그 중 ptmalloc2에 대해 다루겠다.

<br/>

ptmalloc2는 Glibc에 구현되어 있으며, 어떤 메모리가 해제되면, 해제된 메모리의 특징을 기억하고 있다가 유사한 메모리 할당 요청이 들어오면 이를 빠르게 반환해준다. 이를 통해 메모리 할당 속도를 높이고, 불필요한 메모리 사용을 막는다. ptmalloc의 목표는 아래와 같다.

1. 메모리 낭비 방지 : 메모리 할당 요청 발생시, 먼저 해제된 메모리 공간 중에서 재사용 가능한 공간을 탐색하여, 요청과 크기가 같은 메모리 공간이 있다면 이를 재사용하게 한다. 혹은 작은 크기의 할당 요청이 발생했을 경우, 해제된 큰 메모리 공간이 있다면 그 영역을 나누어 주기도 한다.
2. 빠른 메모리 재사용 : 메모리 공간을 해제할 때, tcache 또는 bin이라는 연결 리스트에 해제된 공간의 정보를 저장해 둠으로써, 해제된 메모리 공간을 빠르게 재사용한다.
3. 메모리 단편화 방지 : ptmalloc은 단편화를 줄이기 위해 정렬(Alignment), 병합(Coalescence), 그리고 분할(Split)을 사용한다. 64비트 환경에서 메모리 공간을 16바이트 단위로 할당하는데, 사용자가 어떤 크기의 메모리를 요청하면, 그보다 조금 크거나 같은 16바이트 단위의 메모리 공간을 제공한다. 4바이트 요청시 16바이트를, 17바이트를 요청하면 32바이트를 할당하는 방식이다. 이렇게 하면, 내부 단편화가 발생할 수 있지만, 역설적으로 외부 단편화를 감소시키는 효과가 있다.

## ptmalloc2의 사용 객체

ptmalloc2는 chunk, bin, tcache, arena를 주요 객체로 사용한다.

### Chunk

청크는 덩어리 라는 의미로, ptmalloc이 할당한 메모리 공간을 의미한다. 청크는 헤더와 데이터로 구성되는데, 헤더에는 청크 관리에 필요한 정보를 담고 있고, 데이터 영역에는 사용자가 입력한 데이터가 저장된다. 헤더는 청크의 상태를 나타내는데, in-use 청크와 freed 청크의 헤더는 구조가 다소 다르다. in-use 청크는 fd와 bk를 사용하지 않고, 그 영역까지 데이터를 저장한다. 아래 그림은 Dreamhack에서 퍼온 좋은 그림이다.

![chunk]({{site.url}}/img/chunk.png)

그림에서 청크 헤더의 각 요소에 대한 설명을 하자면,

1. prev_size : 8바이트. 인접한 직전 청크의 크기. 청크를 병합할 때 직전 청크를 찾는데 사용된다.
2. size : 8바이트. 현재 청크의 크기. 헤더의 크기도 포함한 값이다. 64비트 환경에서 사용중인 청크 헤더의 크기는 16바이트이므로, 사용자가 요청한 크기를 정렬하고, 그 값에 16바이트를 더한 값이 된다.
3. flags : 3비트. 64비트 환경에서 청크는 16바이트 단위로 할당되므로, size의 하위 4비트는 의미를 갖지 않는다. 그래서 size의 하위 3비트를 청크 관리에 필요한 flag 값으로 사용한다. 각 flag는 순서대로 A(allocated arena), M(mmap'd), P(prev-in-use)를 나타낸다. P는 직전 청크가 사용중인지를 나타내므로, 이 플래그를 참조하여 병합이 필요한지 판단할 수 있다.
4. fd, bk : 각각 8바이트. 연결 리스트에서 다음, 이전 청크를 가리킨다. 해제된 청크에만 존재한다.

### Bin

bin은 문자 그대로, 사용이 끝난 청크들이 저장되는 객체이다. 메모리의 낭비를 막고, 해제된 청크를 빠르게 재사용할 수 있게 한다. ptmalloc에는 총 128개의 bin이 정의되어 있는데, 62개는 smallbin, 63개는 largebin, 1개는 unsortedbin, 나머지 2개는 사용되지 않는다.

1. smallbin : 32바이트 이상 1024바이트 미만의 청크 보관. 하나의 smallbin에는 같은 크기의 청크들만 보관되며 index가 증가하면 저장되는 청크들의 크기가 16바이트씩 커진다. circular doubly-linked list. FIFO. (LIFO는 속도가 빠르지만 파편화가 심하고, address-ordered는 정렬을 해야해서 속도는 느리나 파편화가 제일 적고, FIFO는 그 중간.) consolidation (인접한 두 청크가 해제되어 있고 이들이 smallbin에 들어있으면, 이 둘은 병합됨.)
2. fastbin : 크기가 작은 청크들이 큰 청크들보다 빈번히 할당 및 해제되므로, 특정 크기 미만의 청크들 (32바이트 이상 176바이트 이하)은 smallbin 대신 fastbin에 저장함. 메모리 단편화보다 속도를 조금 더 우선순위로 두는 bin. 16바이트 단위로 총 10개의 fastbin 존재 (리눅스는 이 중에서 작은 크기부터 7개의 fastbin만을 사용함. 32바이트 이상, 128바이트 이하의 청크들.) single-linked list (unlink과정 필요없음). LIFO. fastbin에 저장되는 청크들은 서로 병합되지 않음.
3. largebin : 1024바이트 이상의 청크 보관. doubly-linked list. consolidation. smallbin, fastbin과는 다르게, 하나의 largebin에는 일정 범위의 청크들을 보관 (따라서 적은 수의 largebin으로 다양한 크기를 갖는 청크들을 관리 가능). best-fit으로 꺼내 재할당.
4. unsortedbin : 분류되지 않은 청크들 보관. circular doubly-linked list. fastbin에 들어가지 않는 모든 청크들은 해제되었을 때 크기를 구분하지 않고 우선 unsortedbin에 보관됨. 청크 분류에 낭비되는 비용과 시간을 줄일 수 있음.

### arena

fastbin, smallbin, largebin 등의 정보를 모두 담고 있는 객체이다. Multithread 환경에서 race condition을 막기 위해 arena에 접근할 때 lock을 적용하는데, 병목 현상을 일으킬 수 있으므로, ptmalloc에서는 이를 최대한 피하기 위해 최대 64개의 arena를 생성할 수 있게 한다. 즉, lock이 걸려서 대기해야 하는 경우, 새로운 arena를 생성하게 된다. 단, 생성 갯수가 64개로 제한되어 있으므로, 과도한 multithread환경에서는 이를 해결하기 위해 tcache를 추가적으로 도입했다.

### tcache (thread local cache)

이름에서 알 수 있듯, 각 쓰레드에 독립적으로 할당되는 캐시 저장소를 지칭한다. 각 쓰레드는 64개의 tcache를 가지고 있는데, fastbin과 마찬가지로 LIFO방식으로 사용되는 single-linked list이며, 하나의 tcache는 같은 크기의 청크들만 보관한다. 리눅스는 각 tcache에 7개의 청크까지만 보관하게끔 하는데, 무제한으로 연결할 경우 메모리가 낭비될 수 있기 때문이다. 32바이트 이상 1040바이트 이하의 청크들이 보관되며, 이 범위에 속한 청크들은 할당 및 해제시 tcache를 가장 먼저 조회한다. tcache가 가득 찼을 경우, 적절한 bin으로 분류된다. arena의 bin에 접근하기 전에 tcache를 먼저 사용하므로 arena에서 발생할 수 있는 병목 현상을 완화한다.

## 마치며

heap에서 구체적으로 어떻게 메모리가 할당되고 관리되는지, ptmalloc2를 기준으로 알아보았다. 중요한 내용이라 다소 긴 줄글로 포스팅을 했다.