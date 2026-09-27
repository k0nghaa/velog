<blockquote>
<p>rAF는 &quot;더 정확한 타이머&quot;가 아니다. 화면 갱신 주기에 맞춰 코드를 실행시키는 도구다.</p>
</blockquote>
<p>스톱워치를 setTimeout으로 만들고 setInterval 보다 완성도 높다 생각했는데, rAF 가 있었다.
아래는 당시 setTimeout으로 구현했던 스톱워치 PR이다.
<a href="https://github.com/hangyedan/assignment_test/pull/31">PR 링크</a></p>
<p><em>이 글의 모든 측정은 Chrome 152.0.7977.77 / Windows 11 Version 25H2 (Build 26200.9168) / 60Hz 모니터 환경에서 진행했습니다. 타이머와 프레임 동작은 브라우저·버전에 따라 달라질 수 있습니다.</em></p>
<h3 id="1️⃣-스톱워치-구현-중-만난-raf">1️⃣ 스톱워치 구현 중 만난 <code>rAF</code></h3>
<p>한계단 과제테스트 중 리액트로 스톱워치 만들기를 진행한 적이 있다.
디깅을 해보니 대부분 setInterval로 스톱워치를 구현했는데, 한 블로그에서 setTimeout + Date.now() 시간차를 통해 구현한 걸 봤다. 좀 더 알아보니 setInterval은 <em><strong>오차가 ~~누적돼도 콜백이 계속 쌓이고(이 부분은 setInterval에서 설명)</strong></em>~~ setTimeout은 하나의 콜백이 해결되어야 그 다음 콜백을 호출하기 때문에 시간차를 통해 구현해야한다면 setTimeout으로 구현하는 게 맞다고 생각했다.</p>
<p>그래서 setTimeout을 자기보정식으로 사용해 Stopwatch를 만들었다. 그런데 다른 스터디원 중 requestAnimationFrame을 사용하여 Stopwatch를 만든 분이 있었다. 나는 그때 rAF를 처음 봤다. &quot;setTimeout이랑 뭐가 다르지? 왜 이게 더 낫다는 거지?&quot;</p>
<h3 id="2️⃣-스크롤-애니메이션-병목-해결-중-만난-raf">2️⃣ 스크롤 애니메이션 병목 해결 중 만난 <code>rAF</code></h3>
<p>rAF를 또 만났다. 이번엔 스크롤 애니메이션 병목 상황을 가정해 최적화 연습을 하는 중이었다. &quot;스크롤 인터랙션이 화려한데, 정작 스크롤할 때 버벅인다&quot; 라는 상황 해결 방안 중 rAF가 나왔다.</p>
<p>아 내가 관심있는 애니메이션(화면)에서도 rAF 가 등장하는구나. </p>
<p><strong>이 기회다 바로 딥다이브 해보자!</strong></p>
<p><img alt="" src="https://velog.velcdn.com/images/k0nghaa/post/fef92763-67f7-49b0-a645-d0ed54300752/image.jpg" /></p>
<hr />
<h1 id="setinterval--settimeout--raf-각각의-이벤트-루프-속-위치">setInterval / setTimeout / rAF 각각의 이벤트 루프 속 위치</h1>
<p>우선 setInterval과 setTimeout 부터 알아야 rAF를 비교를 할 수 있을 것 같다.</p>
<p>setInterval/setTimeout 을 통해 스톱워치를 구현한 블로그들에서는 결국 시간차가 생길 수 밖에 없고 그걸 어떻게 해결할지를 고민하고 있었다.</p>
<p>setInterval과 setTimeout은 <strong>브라우저 명세(HTML Standard)의 Timers 절에 정의된 호스트 API</strong>다. 코드가 setTimeout을 만나면 타이머만 등록하고 즉시 리턴한다. 지정한 시간이 지나면 브라우저가 콜백을 태스크로 만들어 태스크 큐에 넣고, 콜 스택이 비고 그 태스크의 차례가 와야 비로소 실행된다.
명세 자체에도 타이머가 정확히 예정대로 실행되는 걸 보장하지 않는다고 적혀있다. CPU 부하나 다른 태스크로 인한 지연은 당연히 생길 수 있다는 것이다.</p>
<blockquote>
<p><a href="https://html.spec.whatwg.org/multipage/timers-and-user-prompts.html#timers">This API does not guarantee that timers will run exactly on schedule. Delays due to CPU load, other tasks, etc, are to be expected.</a></p>
</blockquote>
<h2 id="setinterval">setInterval</h2>
<pre><code>// 공통 로직: 1초마다 count++ 후 로그, 3이 되면 멈춤
let count = 0;

let id = setInterval(function tick() {
    count = count + 1;
    console.log('tick ' + count);

    if (count === 3) {
        clearInterval(id);
        console.log('done');
    }
}, 1000);

console.log('start');</code></pre><p><img alt="" src="https://velog.velcdn.com/images/k0nghaa/post/a996a077-f087-4ba5-acb5-f2b3887bdf50/image.png" /></p>
<p>1초에 한번씩 
setInterval이 콜백을 반복 예약한다</p>
<h4 id="1번째-결과">1번째 결과</h4>
<p><img alt="" src="https://velog.velcdn.com/images/k0nghaa/post/27a35eed-57eb-41d0-843c-bfaa01d684dd/image.png" /></p>
<h4 id="2번째-결과">2번째 결과</h4>
<p><img alt="" src="https://velog.velcdn.com/images/k0nghaa/post/b2233686-ba3e-4e83-9f40-1eba2fd1d3fb/image.png" /></p>
<h4 id="3번째-결과">3번째 결과</h4>
<p><img alt="" src="https://velog.velcdn.com/images/k0nghaa/post/dbc74732-079e-4c78-b8fe-f02b0d8fd793/image.png" /></p>
<blockquote>
<p>용어 정리 (DevTools 공통 정의)
Total = 해당 호출이 자식 호출까지 포함해서 잡은 시간
self = 자식 호출을 제외한 그 호출 자체의 시간
Total - Self = 실제 tick 작업(이 실험 기준)</p>
</blockquote>
<p>Event log를 통해서 시간 차이가 0.x ms 씩 발생하는 걸 알 수 있다. 추가로 알게된 것도 있다(상단에서 언급한 오차 누적 부분에 대한 설명). 극 초반에 테스트했을 때 이 차이가 모두 양수로 나와 <strong>누적</strong>이라고 생각했는데, 테스트를 여러번 진행한 후 확인해보니 <strong>누적(drift)</strong>이 아니라 <strong>흔들림(jitter)</strong> 이었다. 
3번째 결과에서 1,000.1ms ⭢ 2,000.8ms ⭢ 3,000.4ms 로 +0.7ms ⭢ -0.4ms 오차가 누적되지 않는 것을 볼 수 있었다. </p>
<h3 id="setinterval의-함정---coalescing">setInterval의 함정 - coalescing</h3>
<blockquote>
<p>&quot;밀린 콜백이 쌓이는 게 아니라 합쳐진다&quot;</p>
</blockquote>
<pre><code>let t0 = performance.now();                      // ① 기준 시각
const test = setInterval(() =&gt; {                 // ② 200ms마다 콜백 예약, test = 타이머 id
  console.log(`fire @ ${Math.round(performance.now() - t0)}ms`);
}, 200);
const end = performance.now() + 1000;            // ③ 여기서부터
while (performance.now() &lt; end) {}               //    ~1초간 메인 스레드 통째로 블록
console.log(`unblocked @ ...`);                  // ④ 풀린 시각 로그
setTimeout(() =&gt; clearInterval(test), 1400);     // ⑤ 1.4초 뒤 인터벌 자동 정지</code></pre><p>1000ms(1초) 동안 while 반복문을 통해 메인 스레드를 막아보았다. 
이렇게 하면 1초 block 동안 200ms 간격으로 5번 쌓였다가 몰아서 실행될 것이라 예상되지만, 실제로는 1회(fire @ 1000ms)만 실행된다. 밀린 콜백이 쌓이지 않고 1개로 합쳐지며 나머지 tick은 버려지는 것이다. 그 이후에는 조금의 jitter가 있지만 200ms 간격으로 재개된다. 스레드가 한가하니 하나씩 정상 실행되는 것이다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/k0nghaa/post/bd3ff0e7-f3fd-4ba7-9ef9-8bc8155abbf1/image.png" /></p>
<p>타임라인은 다음과 같다
<img alt="" src="https://velog.velcdn.com/images/k0nghaa/post/85cd34a5-519a-4351-99ea-212b3d016f3d/image.png" /></p>
<p><img alt="" src="https://velog.velcdn.com/images/k0nghaa/post/6282d312-2122-4fee-88d4-28073493d39f/image.png" /></p>
<h3 id="여기서-알-수-있는-것">여기서 알 수 있는 것</h3>
<p><strong>이전 콜백이 아직 실행되지 않았다면, 밀린 tick은 쌓이지 않고 버려진다.</strong></p>
<p>명세상 setInterval은 콜백을 실행한 <strong>뒤에</strong> 같은 간격으로 타이머를 다시 거는 구조라, 반복 타이머의 예약은 항상 하나 뿐이다. 그래서 &quot;쌓였다가 몰아서 실행&quot;될 수가 없다. (Chrome은 여기에 더해 다음 실행 시각을 원래 간격의 격자에 맞춰 계산하고, 놓친 박자는 건너뛴다. 앞에서 오차가 누적(drift)되지 않고 흔들림(jitter)만 보였던 이유도 이것이다.)</p>
<p>만약에 <code>count++</code> 처럼 tick 수를 세서 시간을 계산했다면 버려진 tick만큼 시간이 틀어졌을 것이다. 이 실험에서는 1초 동안 5번 와야 할 tick이 1번만 왔으니 800ms를 잃는다. 
그래서 tick을 <strong>세는</strong> 대신 <code>performance.now()</code>로 현재 시각을 <strong>읽고</strong> 시작 시각과의 차이로 경과 시간을 계산해야 하는 이유가 여기에 있다. (setTimeout이든 setInterval()이든 값을 보정해야하는 것은 마찬가지) </p>
<h2 id="settimeout">setTimeout</h2>
<p>일단 가볍게 setTimeout의 구조를 살펴봤다.</p>
<pre><code>// 재귀 setTimeout: 콜백 안에서 다음 setTimeout을 &quot;다시&quot; 예약
let t0 = performance.now();
let count = 0;
function tick() {
  console.log(`fire @ ${Math.round(performance.now() - t0)}ms`);
  count++;
  if (count &lt; 8) setTimeout(tick, 200);   // ← 매번 여기서 새로 등록
}
setTimeout(tick, 200);</code></pre><p><img alt="" src="https://velog.velcdn.com/images/k0nghaa/post/5ecbe14c-b076-4710-8b3b-4e1f4d8ae423/image.png" /></p>
<p>각각의 setTimeout이 호출되며 재귀하는 걸 볼 수 있었다. <code>Repeats = false</code> 를 통해 확실히 개별적인 이벤트임을 확인했다.</p>
<p>그리고 setTimeout도 편차는 있었다. </p>
<p>결국은 setTimeout과 setInterval 모두 오차가 있기 때문에 타이머가 부정확할 수밖에 없다. 그래서 Date.now() 를 사용해 값을 보정하는 방식을 사용한 것이다. </p>
<pre><code>...
const updateTime = () =&gt; {
  if (isRunning) {
    const now = Date.now();
    const diff = now - startTime;
    setTime(diff);                              // ← ① 값 보정: 시계를 &quot;읽음&quot; 
    const nextTick = INTERVAL - (diff % INTERVAL);  // ← ② 스케줄 보정: 30ms 격자에 맞춤
    timer = setTimeout(updateTime, nextTick);
  }
};
...</code></pre><p>cc. 당시 참고한 블로그 <a href="https://develog-yeon.tistory.com/33">https://develog-yeon.tistory.com/33</a></p>
<p>결국 Stopwatch를 만들 때 <strong>setTimeout을 쓴 진짜 이유는 coalesce 회피가 아니라 다음 예약 시점을 매번 보정할 수 있기 때문이었다.</strong>
재귀 setTimeout는 다음 예약이 콜백 안에서 콜백이 끝날 때 걸려서 항상 대기 중인 타이머가 최대 1개가 된다. </p>
<p><strong>그렇다면 rAF는 뭘까?</strong></p>
<h2 id="🔍-raf-딥다이브-본론">🔍 rAF 딥다이브 본론</h2>
<p><code>requestAnimationFrame(cb)</code> : 다음 리페인트 전에 사용자가 지정한 콜백함수(<code>cb</code>)를 호출하도록 요청하는 메서드</p>
<h3 id="특징-1---언제-실행되는가">특징 1 - 언제 실행되는가?</h3>
<p>이벤트 루프는 아래와 같이 돌아간다.</p>
<blockquote>
<p>태스크 1개 실행 → 마이크로태스크 전부 비우기 → (렌더링 기회가 있으면) 렌더 단계 → 다시 태스크 1개 실행 → ... 반복</p>
<p><strong>1.</strong> 렌더 단계는 <strong>매 태스크마다 도는 게 아니라 프레임당 대략 1회</strong> 실행된다.
** 2.** 렌더 단계를 더 쪼개면 <code>resize steps → **scroll steps** → 미디어쿼리 평가 → CSS 애니메이션/이벤트 → **rAF 콜백 **→ style → layout → paint</code> 이다. 여기서 scroll이 rAF보다 앞단계라는 걸 기억해두자</p>
</blockquote>
<p>위 Performance Event log 에서 확인할 수 있듯이 setTimeout/setInterval(타이머 콜백)은 Timer fired, 즉 태스크 큐에서 꺼내져 실행되는 <strong>태스크</strong>다.
반면 rAF는 Animation frame fired로 찍힌다. rAF 콜백은 태스크 큐에 들어가지 않는다. 문서의 animation frame callbacks 목록에 등록되어 있다가, 렌더링 단계(style/layout 직전)에서 한꺼번에 실행된다.</p>
<p>여기서 흔한 오해가 있다. &quot;rAF를 쓰면 리플로우/리페인트가 줄어든다&quot;는 것이다.</p>
<ul>
<li><strong>페인트는 원래 프레임당 최대 1회다.</strong> setTimeout으로 한 프레임 안에 DOM을 다섯 번 바꿔도 화면은 한 번만 그려진다. 렌더링 단계 자체가 프레임당 1회이기 때문이다.</li>
<li><strong>레이아웃도 기본적으로 미뤄진다.</strong> DOM을 바꾸면 &quot;다시 계산해야 함&quot; 표시만 해두고, 실제 계산은 렌더링 단계에서 한 번 한다. 레이아웃이 중복으로 일어난 건 <code>쓰기 ⭢ offsetHeight 같은 읽기 ⭢ 쓰기</code> 를 반복해서 브라우저가 즉시 계산하도록 강제할 때(forced synchronous layout, layout thrashing)다. 이건 rAF 안에서도 똑같이 발생한다.</li>
</ul>
<p>그렇다면 rAF가 실제로 해주는 건 뭘까?</p>
<ol>
<li><strong>화면에 반영되지 않을 낭비를 없앤다.</strong> 한 프레임 안에 타이머나 이벤트가 여러 번 와서 매번 계산하고 DOM을 바꿔도, 사용자가 보는 건 마지막 결과 하나뿐이다. rAF를 하면 프레임당 한 번만 계산한다.</li>
<li><strong>갱신 타이밍을 프레임에 맞춘다.</strong> 어떤 프레임엔 두 번 바뀌고 어떤 프레임엔 안 바뀌는 불균일한 갱신이 사라진다.</li>
</ol>
<p>즉 rAF는 리플로우/리페인트를 줄이는 도구라기보다 <strong>&quot;그리기 직전&quot;이라는 타이밍을 주는 도구</strong>다.</p>
<h3 id="특징-2---얼마나-자주-실행되는가">특징 2 - 얼마나 자주 실행되는가?</h3>
<p>rAF는 1프레임에 1번 실행된다. 정확히는 브라우저의 렌더링 기회에 맞춰 실행되는데, 그게 보통 디스플레이 주사율과 같다. 만약 콜백 실행 시간이 프레임 예산(60Hz면 약 16.7ms)을 넘으면 프레임을 드롭하고 실제 frame rate에 맞춰 실행된다. 
다만 &quot;모니터 주사율 = rAF 주기&quot;는 보장이 아니라 기본값에 가깝다. 가변 주사율(VRR), 배터리 절약 모드, 화면 밖 iframe 등에서는 달라진다.</p>
<p>ex. 60Hz → ~16.67ms마다, 120Hz → ~8.33ms마다 ...</p>
<h4 id="실험-프레임-간격--timestamp-인자-주사율-확인">실험) 프레임 간격 + timestamp 인자 (주사율 확인)</h4>
<pre><code>let f = 0, prev = null;
function frame(t) {
  // t = DOMHighResTimeStamp (이 프레임의 시각)
  const gap = prev === null ? 0 : t - prev;
  prev = t;
  console.log(`frame #${++f} | timestamp ${t.toFixed(1)}ms | 간격 ${gap.toFixed(1)}ms`);
  if (f &lt; 20) requestAnimationFrame(frame);
}
requestAnimationFrame(frame);</code></pre><p><img alt="" src="https://velog.velcdn.com/images/k0nghaa/post/6ab78c84-f8fb-4a87-a87b-629e2b39410b/image.png" /></p>
<p><img alt="" src="https://velog.velcdn.com/images/k0nghaa/post/0ddde895-2c6a-422c-b9b9-38c49f38a4db/image.png" /></p>
<p>내 화면(모니터)은 60Hz 인 것을 확인했다.
<code>1000(ms) ÷ 16.7(간격) ≈ 60 → 60Hz.</code></p>
<h3 id="특징-3---콜백-인자에-무엇을-담나">특징 3 - 콜백 인자에 무엇을 담나?</h3>
<p><code>requestAnimationFrame((timestamp) =&gt; { ... });</code></p>
<p>여기서 timestamp는 <code>performance.now()</code> 와 같은 시계를 쓴다. 기준점(time origin)은 브라우저를 실행한 시점이 아니라 그 document의 time origin(대략 navigation start)이다. 탭마다, 문서마다 0이 다르므로 iframe이나 다른 문서의 값과 그냥 비교하면 깨진다. 
그리고 한 프레임 안의 모든 rAF 콜백은 같은 timestamp를 받는다.</p>
<h4 id="실험-timestamp가-performancenow와-같은-시계인가">실험) timestamp가 performance.now()와 같은 시계인가</h4>
<pre><code>requestAnimationFrame((t) =&gt; {
  console.log('rAF timestamp   :', t.toFixed(1));
  console.log('performance.now :', performance.now().toFixed(1));
});</code></pre><p><img alt="" src="https://velog.velcdn.com/images/k0nghaa/post/06c980fe-b6b8-4452-b131-ca89cffaf5d5/image.png" /></p>
<h4 id="실험-한-프레임-안의-여러-raf는-같은-timestamp">실험) 한 프레임 안의 여러 rAF는 같은 timestamp</h4>
<pre><code>// 같은 프레임에 콜백 3개 예약 → 셋 다 같은 timestamp를 받나?
requestAnimationFrame((t) =&gt; console.log('A', t.toFixed(3)));
requestAnimationFrame((t) =&gt; console.log('B', t.toFixed(3)));
requestAnimationFrame((t) =&gt; console.log('C', t.toFixed(3)));</code></pre><p><img alt="" src="https://velog.velcdn.com/images/k0nghaa/post/6e3eb00d-35d0-4bd9-abc8-3c6a5d40a28f/image.png" /></p>
<p>셋 다 93457.100이 나온 것을 통해 한 프레임에 예약된 모든 rAF 콜백은 콜백이 실행된 순간이 아니라 프레임 시각을 받는다는 걸 알 수 있다.</p>
<h3 id="특징-4---백그라운드에서는-raf-정지">특징 4 - 백그라운드에서는 rAF 정지</h3>
<p>결론부터 말하자면 안 보이는 탭에서는 rAF 루프가 멈춘다. 탭이 안보이면 브라우저가 화면을 그리지 않는다. 렌더링 기회가 없으니 렌더링 단계에서 실행되는 rAF도 실행되지 않는다.</p>
<p>주의할 건 페이지 전체가 멈추는 게 아니라는 점이다. <strong>rAF = 정지 / setTimeout, setInterval = 정지가 아니라 감속</strong> 이다.</p>
<ul>
<li>숨겨진 탭의 타이머는 1초 단위로 묶어서 실행된다. 200ms 타이머여도 1초에 한번꼴로 깨어난다.</li>
<li>메모리 절약 등으로 탭이 frozen/discarded 상태가 되면 타이머, 네트워크까지 전부 멈춘다.</li>
</ul>
<p>그렇기 때문에 <strong>만약 계속 초가 흘러야 하는 스톱워치 구현에서는 rAF든 타이머든 &quot;몇 번 실행됐나&quot;가 아니라 &quot;시작 시각과 지금의 차이&quot;로 시간을 계산해야한다.</strong> 그러면 탭이 멈췄다 돌아와도 첫 프레임에서 바로 올바른 값이 나온다. 
(탭 복귀 시점에 뭔가 처리해야 한다면 <code>visibilitychange</code> 이벤트를 쓰면 된다.)</p>
<p>그렇다면 rAF를 통해 간단하게 스톱워치를 구현해보자.</p>
<h2 id="raf-스톱워치-코드">rAF 스톱워치 코드</h2>
<pre><code class="language-jsx">import { useEffect, useRef, useState } from 'react';
import { formatTime } from './formatTime';

export default function StopwatchRAF() {
  const [time, setTime] = useState(0);
  const [isRunning, setIsRunning] = useState(false);
  const startRef = useRef(0);
  const rafRef = useRef(null);

  useEffect(() =&gt; {
    if (!isRunning) return;

    const update = (t) =&gt; {
    // rAF는 &quot;언제 갱신할지&quot;만 담당하고, 값은 시계를 직접 읽는다
     setTime(t - startRef.current); 
  rafRef.current = requestAnimationFrame(update);
};

    rafRef.current = requestAnimationFrame(update);
    return () =&gt; 
    rafRef.current !== null &amp;&amp; cancelAnimationFrame(rafRef.current);   // 정리 (clearTimeout의 rAF 버전)
  }, [isRunning]);

  const handleToggle = () =&gt; {
    if (!isRunning) {
      startRef.current = performance.now() - time;   // resume 지원 (누적 보존) 
      setIsRunning(true);
    } else {
      setTime(performance.now() - startRef.current); // 멈춘 '순간'의 값으로 확정
      setIsRunning(false);
    }
  };

  const handleReset = () =&gt; { setIsRunning(false); setTime(0); };

  return (
    &lt;div&gt;
      &lt;p&gt;{formatTime(time, false)}&lt;/p&gt;
      &lt;button onClick={handleToggle}&gt;{isRunning ? 'Stop' : 'Start'}&lt;/button&gt;
      &lt;button onClick={handleReset}&gt;Reset&lt;/button&gt;
    &lt;/div&gt;
  );
}</code></pre>
<p>기존 setTimeout 버전과 달라진 것은 </p>
<p><code>const nextTick = INTERVAL - (diff % INTERVAL)</code> 코드가 삭제되었다는 것이다. 값 보정은 그대로 하되, 스케줄 보정(언제 다시 실행할지)은 브라우저의 프레임 주기에 맡겼다. 직접 계산하던 걸 브라우저에게 위임한 셈이다!</p>
<p>[실제 비교 영상]</p>
<p>사실상 setTimeout의 INTERVAL을 30으로 설정해놔 사람 눈으로 봤을 때 차이를 느끼기 어려울텐데도 rAF 버전이 더 부드럽게 느껴졌다. 60Hz(내 모니터) 모니터의 프레임 간격은 16.67ms인데 그 배수가 아닌 30ms 로 설정되어 setTimeout 버전의 갱신은 &quot;2프레임 뒤, 2프레임 뒤, ... 가끔 1프레임 뒤&quot; 처럼 불규칙한 간격으로 화면에 반영된다. rAF 버전은 매 프레임 균일하게 갱신되니 실제로 더 부드러운 것이다.</p>
<p>정리를 해보자면 rAF는 내가 느낀 것 처럼 화면 변화가 이상없이 부드럽게, 화면 주기율에 맞춰 움직이는/변하는 상황에서 사용할 수 있다. </p>
<p>그래서 서론에서 말했던 스크롤 애니메이션에서 많이 쓰이는데,
마지막으로 이에 대한 정리를 해보려 한다. </p>
<h2 id="스크롤-성능에서의-raf--놀라운-사실😲">스크롤 성능에서의 rAF (+ 놀라운 사실😲)</h2>
<p>가정했던 스크롤 애니메이션 병목 상황은 아래와 같았다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/k0nghaa/post/5f87019e-5fb4-4620-a98b-c8290ffc323e/image.png" /></p>
<p><em>문제 상황 중 rAF와 관련있는 scroll 관련해서만 정리</em></p>
<p>사실 이 문제에서 가장 큰 병목을 해결하는 방법이 rAF는 아니다.
top/width를 직접 변경해 reflow/repaint를 유발하는 로직을 transform/opacity로 교체하는 게 가장 큰 효과이다. 단, transform/opacity가 레이아웃과 페인트를 건너뛰고 합성(composite)만 하려면 해당 요소가 <strong>별도 레이어로 승격</strong>되어 있어야 한다(CSS 애니메이션 등). 그렇지 않으면 리페인트가 일어날 수 있다.</p>
<p>이 딥다이브를 하면서 알게된 사실이 있다. <strong>scroll 이벤트는 원래 프레임당 최대 1회만 발생한다.</strong> 최신 브라우저 의 특별한 최적화가 아니라 명세에 정의된 동작이다. 앞에서 본 렌더링 단계를 다시 보면 <code>scroll steps ⭢ ... ⭢ rAF 콜백</code> 순서였다. 브라우저는 스크롤이 일어나도 즉시 이벤트를 쏘지 않고 모아뒀다가, 렌더링 단계의 scroll steps 에서 한 번에 발송한다. 그래서 scroll 핸드러를 rAF로 감싸 &quot;프레임당 1회로 제한&quot;하는 흔한 패턴은 대부부분 이미 브라우저가 해주고 있는 일을 한번 더 하는 셈이다. </p>
<p>하나 더 알게 된 건 <strong>최신 브라우저에서 스크롤 자체는 메인 스레드가 아니라 컴포지터 스레드에서 처리된다</strong>는 점이다. 메인 스레드가 바빠도 페이지는 스크롤된다. 버벅이는 건 스크롤에 연동된 JS 애니메이션이다. 
그래서 스크롤 인터랙션은 다음 방향이 더 근본적인 해법이다.</p>
<ul>
<li>요소가 보이는지 확인할 때는 scroll 이벤트 대신 <code>IntersectionObserver</code></li>
<li>스크롤 위치에 따른 애니메이션은 <strong>CSS scroll-driven animations</strong> (<code>animation-timeline: scroll()</code>)</li>
</ul>
<p>결론적으로 rAF의 유용성은 프레임 동기 트리거를 내가 직접 만들어야 하는지에 따라 결정된다. 스톱워치처럼 프레임 트리거를 직접 만들어야하면(ex. <code>INTERVAL = 30</code>) rAF가 JS에서 프레임에 맞춰 콜백을 받는 대표적인 도구가 된다. 그러나 스크롤처럼 브라우저가 이미 프레임에 맞춘 이벤트를 준다면 rAF는 덜 중요해진다. <em>(그럴 때 진짜 문제는 렌더 파이프라인이다)</em></p>
<p>한마디로 정리하면 </p>
<blockquote>
<p>&quot;화면에 얼마나 자주, 화면 그리는 리듬에 맞춰 갱신하나&quot; </p>
</blockquote>
<p>로 마무리 할 수 있겠다!
<img alt="" src="https://velog.velcdn.com/images/k0nghaa/post/5906fd08-074f-408d-b737-00d49485f77e/image.jpg" /></p>